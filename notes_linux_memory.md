# Linux Memory: Complete Reference for Troubleshooting & Leak Detection

---

## Part 1 — Three-Dimensional Framework

### Dimension 1: Spatial State (Virtual vs Physical)

```
  malloc(1GB)          Only RSS matters
  VIRT = 1GB    ──→    RSS = 0  (never touched)
  RSS   = 0            RSS grows only on page fault
```

```
Process Virtual Address Space          Physical RAM (actual hardware)
┌──────────────────────────┐          ┌─────────────────┐
│  0x7fff0000  [stack]     │ ───────→ │ page: stack     │
│      ↓                   │          ├─────────────────┤
│  0x7f800000  [libc.so]   │ ──┐      │ page: libc code │ ←─┐ shared
│                          │   │      ├─────────────────┤   │
│  0x00a00000  [heap]      │ ──┼──→   │ page: heap      │   │
│      ↑                   │   │      ├─────────────────┤   │
│  0x00400000  [.text]     │   │      │ page: .text     │   │
│                          │   │      └─────────────────┘   │
│  [reserved - never       │   │                            │
│   touched, no RSS]       │   │  Process B also maps ──────┘
└──────────────────────────┘   │  the same libc page
                               └→ (RSS counted in both, but
                                   physically ONE page in RAM)
```

**Key rule:** VIRT = address space reserved. RSS = pages actually in RAM.

---

### Dimension 2: Physical Nature (File-backed vs Anonymous)

```
All RSS pages
│
├── File-backed (has a file on disk)
│   ├── Clean  → directly discard under pressure (safe)
│   └── Dirty  → write back to disk, then discard
│
└── Anonymous (no backing file — runtime-generated data)
    ├── Private → heap, stack, BSS
    │   └── under pressure → swap out (or OOM kill if no swap)
    └── Shared  → fork() unmodified pages, shm
        └── under pressure → depends on sharing state
```

```
Memory pressure response timeline:

[System RAM low]
      ↓
Reclaim file-backed clean pages first  (free, no cost)
      ↓
Reclaim file-backed dirty pages        (write to disk, slower)
      ↓
Swap anonymous pages to swap space     (slow, thrashing risk)
      ↓
No swap left → OOM Killer selects and kills a process
```

---

### Dimension 3: Audit Accounting (SHR / USS / PSS)

```
Process A  RSS = 100 MB
┌──────────────────────────────────────────┐
│                                          │
│   Private pages (USS)       │  60 MB    │
│   heap, stack, COW dirty    │           │
│                             │           │
├─────────────────────────────┤           │
│   Shared pages (SHR)        │  40 MB    │
│   libc.so, libstdc++.so     │           │
│   shared with 4 other procs │           │
│                             │           │
└──────────────────────────────────────────┘

PSS = USS + (SHR / number of sharing processes)
    = 60  + (40  / 5)
    = 60  + 8
    = 68 MB   ← true cost attributed to this process
```

```
Summing across all processes:

Process A: PSS = 68 MB
Process B: PSS = 45 MB
Process C: PSS = 32 MB
...
─────────────────────
Total PSS = actual physical RAM in use  (no double-counting)

vs.

Total RSS = 280 MB  (inflated — shared pages counted multiple times)
```

---

## Part 2 — Process Memory Layout

```
Virtual Address Space of a Process (64-bit Linux)

0xFFFFFFFFFFFFFFFF ┌──────────────────────────────────┐
                   │         Kernel Space              │ (inaccessible)
0xFFFF800000000000 ├──────────────────────────────────┤
                   │                                  │
                   │    Stack  (main thread)          │ grows ↓
                   │         ↓                        │ default limit: 8MB
                   │    [guard page - SIGSEGV]        │
                   │    Stack  (thread 2)             │
                   │         ↓                        │
                   │                                  │
                   │    mmap region                   │ grows ↑
                   │    - shared libraries (.so)      │ loaded by dynamic linker
                   │    - file mappings               │ mmap(file)
                   │    - large malloc (>128KB)       │ via mmap(anon)
                   │         ↑                        │
                   │                                  │
                   │    Heap                          │ grows ↑
                   │         ↑                        │ via brk() / sbrk()
                   ├──────────────────────────────────┤
                   │    BSS   uninitialized globals   │ zeroed at startup
                   ├──────────────────────────────────┤
                   │    Data  initialized globals     │ int x = 5;
                   ├──────────────────────────────────┤
                   │    Text  executable code         │ read-only
0x0000000000400000 └──────────────────────────────────┘
```

```c
// Where each variable lives:

int  g_init   = 42;        // Data segment
int  g_uninit;             // BSS segment
static int s  = 10;        // Data segment

void foo() {
    int  local   = 1;      // Stack
    int *p = malloc(64);   // p → Stack,  *p → Heap
    static int c = 0;      // Data segment  (NOT stack)
    mmap(...);             // mmap region
}
```

---

## Part 3 — Page Fault Mechanism

```
Process accesses virtual address 0x00a01234
         │
         ▼
   CPU checks TLB
   (Translation Lookaside Buffer — fast cache of page table)
         │
    ┌────┴────┐
    │ TLB hit?│
    └────┬────┘
         │ NO (TLB miss)
         ▼
   CPU walks Page Table
         │
    ┌────┴──────────────┐
    │ mapping valid?     │ NO → SIGSEGV (segfault)
    └────┬──────────────┘
         │ YES
    ┌────┴──────────────┐
    │ physical page      │ YES → update TLB, resume process
    │ present in RAM?    │
    └────┬──────────────┘
         │ NO → PAGE FAULT (trap into kernel)
         ▼
   ┌─────────────────────────────────────────┐
   │ Kernel Page Fault Handler               │
   │                                         │
   │  File-backed?                           │
   │  ├── YES → find page in page cache      │
   │  │         or load from disk            │
   │  └── NO (anonymous) →                  │
   │        allocate new zeroed page         │
   │        (demand zero page)               │
   └─────────────────────────────────────────┘
         │
         ▼
   update page table + TLB
         │
         ▼
   resume process (transparent to application)
```

---

## Part 4 — Copy-on-Write (COW)

```
Before fork():                   After fork():

Parent process                   Parent            Child
┌──────────────┐                 ┌──────────┐      ┌──────────┐
│ heap page A  │                 │ heap pg A│─┐ ┌─│ heap pg A│
│ (writable)   │                 │ (R/O now)│ │ │ │ (R/O now)│
└──────────────┘                 └──────────┘ │ │ └──────────┘
                                              │ │
                                  shared physical page
                                        ┌────┴─┴────┐
                                        │  Page A   │
                                        │ (1 copy)  │
                                        └───────────┘

Child writes to heap page A:
         ↓
   Page fault → kernel allocates new page for child
         ↓
Parent ──→ original Page A    (unchanged)
Child  ──→ new Page A copy    (modified)
         ↓
   Private_Dirty in child's smaps increases
   This page now counted in child's USS
```

---

## Part 5 — Commands Reference

### System-wide overview

```bash
# Available memory (the number that matters most)
free -h

# Detailed memory breakdown
cat /proc/meminfo

# Memory stats summary
vmstat -s

# Memory by process, sorted by RSS
ps aux --sort=-%mem | head -20

# Watch system memory every 2 seconds
watch -n 2 free -h
```

### Per-process inspection

```bash
# Quick summary: VIRT, RSS, shared
cat /proc/<pid>/status | grep -E "VmPeak|VmSize|VmRSS|VmData|VmStk"

# All memory mappings with sizes
pmap -x <pid>

# Detailed per-mapping breakdown (the raw source of truth)
cat /proc/<pid>/smaps

# Extract USS (private dirty pages) from smaps
grep Private_Dirty /proc/<pid>/smaps | awk '{sum += $2} END {print sum " kB"}'

# Extract PSS from smaps
grep ^Pss /proc/<pid>/smaps | awk '{sum += $2} END {print sum " kB"}'

# Watch USS grow over time (leak detection)
watch -n 3 "grep Private_Dirty /proc/<pid>/smaps | awk '{sum+=\$2} END {print sum\" kB\"}'"
```

### PSS / USS with smem

```bash
# Install
sudo apt install smem

# All processes ranked by PSS (most expensive first)
smem -r -k -p

# Summary by process name
smem -r -k -t -P <name>

# Pie chart of memory by process (requires matplotlib)
smem --pie name -s pss
```

### Valgrind — userspace leak detection

```bash
# Full leak check (finds definite + possible leaks)
valgrind --leak-check=full \
         --show-leak-kinds=all \
         --track-origins=yes \
         --verbose \
         ./your_program

# Heap profiler — memory usage over time
valgrind --tool=massif ./your_program
ms_print massif.out.<pid>        # text report
massif-visualizer massif.out.*   # GUI (if available)
```

Valgrind leak categories:
```
definitely lost  → pointer to block is gone — real leak
indirectly lost  → reachable only via a lost block
possibly lost    → interior pointer (may or may not be a leak)
still reachable  → pointer exists at exit but not freed (often OK)
```

### AddressSanitizer — compile-time instrumentation

```bash
# Compile with ASan
gcc -fsanitize=address -fsanitize=leak -g -O1 -o program program.c

# Run normally — ASan output appears on error or exit
./program

# Suppress false positives with suppression file
ASAN_OPTIONS=suppressions=asan.supp ./program
```

ASan catches: heap overflow, stack overflow, use-after-free, double-free, use-after-return.

### Kernel memory — kmemleak

```bash
# Enable in kernel config: CONFIG_DEBUG_KMEMLEAK=y
# (rebuild kernel or use a debug kernel)

# Trigger a scan
echo scan > /sys/kernel/debug/kmemleak

# Read results
cat /sys/kernel/debug/kmemleak

# Clear reported leaks
echo clear > /sys/kernel/debug/kmemleak
```

### Kernel slab allocator

```bash
# Live slab usage (like top, for kernel objects)
slabtop

# Raw slab stats
cat /proc/slabinfo

# Watch for growing slab entries
watch -n 2 slabtop
```

### OOM analysis

```bash
# Check if OOM killer fired
dmesg | grep -iE "oom|killed process|out of memory"

# OOM score of a process (higher = more likely to be killed)
cat /proc/<pid>/oom_score

# Adjust OOM priority (-1000 = never kill, +1000 = kill first)
echo -500 > /proc/<pid>/oom_score_adj
```

---

## Part 6 — Full Reference Matrix

| Physical type | Runtime identity | smaps field | Metric | Fate under pressure |
|---|---|---|---|---|
| File-backed clean | Shared lib code, unmodified page cache | `Shared_Clean` | SHR | Discarded immediately |
| File-backed dirty | `MAP_SHARED` modified file | `Shared_Dirty` | SHR | Written back, released |
| File-backed COW | Library globals, private modified file | `Private_Dirty` | USS | Swapped out |
| Anonymous private | `malloc` heap, thread stack, BSS | `Private_Dirty` | USS | Swapped out or OOM |
| Anonymous shared | post-`fork` unmodified, explicit shm | `Shared_Dirty` | SHR | Depends on sharing |

---

## Part 7 — Memory Leak Troubleshooting Workflow

```
Step 1: Confirm a leak exists
─────────────────────────────
Watch RSS or USS grow over time under idle/stable load.

  watch -n 5 "grep VmRSS /proc/<pid>/status"

  If RSS grows monotonically → suspect leak
  If RSS grows then plateaus → likely not a leak (steady state)


Step 2: Isolate private vs shared growth
─────────────────────────────────────────
  watch -n 5 "grep Private_Dirty /proc/<pid>/smaps | \
              awk '{sum+=\$2} END {print sum\" kB\"}'"

  USS growing → leak is in your process's own heap/stack
  RSS growing but USS stable → shared mapping issue (rare)


Step 3: Identify which mapping is growing
──────────────────────────────────────────
  Take two smaps snapshots, diff them:

  cat /proc/<pid>/smaps > smaps_before.txt
  # ... wait / run workload ...
  cat /proc/<pid>/smaps > smaps_after.txt
  diff smaps_before.txt smaps_after.txt

  Look for heap region [anon] size increases.


Step 4: Find the allocation site
──────────────────────────────────
  Userspace:
    valgrind --leak-check=full ./program
    gcc -fsanitize=address,leak → run program

  Kernel module:
    echo scan > /sys/kernel/debug/kmemleak
    cat /sys/kernel/debug/kmemleak


Step 5: Confirm fix
─────────────────────
  Re-run the same workload after fix.
  USS should remain stable over time.
  Valgrind should report 0 bytes "definitely lost".
```

---

## Part 8 — Common Leak Patterns

```c
// Pattern 1: Allocation without free
void process() {
    char *buf = malloc(1024);
    if (error) return;          // BUG: buf leaked on error path
    // ...
    free(buf);
}

// Pattern 2: Lost pointer (realloc)
char *p = malloc(100);
p = realloc(p, 200);            // BUG: if realloc fails, returns NULL
                                // original p is lost

// Pattern 3: Container not cleared
struct node *head = NULL;
while (condition) {
    struct node *n = malloc(sizeof(*n));
    n->next = head;
    head = n;
}
// BUG: list never freed — entire chain leaks at exit

// Pattern 4: Kernel — missing kfree
void *buf = kmalloc(size, GFP_KERNEL);
if (some_error)
    return -EINVAL;             // BUG: buf leaked — must kfree before return
kfree(buf);
```

---

## Part 9 — Quick Diagnostic Cheatsheet

```
Symptom                       First command to run
──────────────────────────────────────────────────────────────
System RAM low                free -h  →  check "available"
Which process eating RAM      ps aux --sort=-%mem | head -10
Process RSS growing           watch grep VmRSS /proc/<pid>/status
Is it a real leak?            watch grep Private_Dirty /proc/<pid>/smaps
Which mapping growing         diff two smaps snapshots
Find allocation site          valgrind --leak-check=full
Kernel leak                   cat /sys/kernel/debug/kmemleak
OOM kill happened             dmesg | grep -i oom
Slab growing                  slabtop
True per-process cost         smem -r -k -p  (use PSS not RSS)
```
