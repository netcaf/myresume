# Hardware Protocols & Firmware Methods — Quick Reference

---

## 1 · Physical Communication Protocols

| Protocol | Min Lines | Pins | Duplex | Talks To | Multi-device | Main Use |
|---|---|---|---|---|---|---|
| **UART** | 2 | TX, RX | Full / async | Point-to-point, no software | ✗ not supported | Debug console, GPS/4G modules, RS-485 |
| **I2C** | 2 | SCL, SDA | Half / sync | Peripheral chip registers (needs SW) | 7-bit address, no extra wire | Slow sensors, config EEPROM, RTC |
| **SPI** | 4 | CLK, MOSI, MISO, CS | Full / sync | Independent peripheral chip directly, no SW | One CS wire per device | SPI Flash, LCD/OLED, SD cards |
| **JTAG** | 4 | TCK, TMS, TDI, TDO | Full / sync | CPU internal TAP (16-state state machine) | Daisy-chain across chips | Full debug + flash; boundary scan |
| **SWD** | 2 | SWCLK, SWDIO | Half / sync | CPU internal TAP (ARM only) | Single chip only | ARM Cortex-M: step, breakpoint, registers |

**Key insight — SPI:** "SPI Flash" means a Flash chip that *uses SPI to talk to the CPU*. There is no extra SPI chip. The Flash is an independent device; SPI is just the protocol. Same logic applies to I2C sensors or UART modems — the protocol name describes the interface, not an extra chip.

**JTAG vs SPI for flashing:** SPI talks directly to the Flash chip — CPU not needed, you could even desolder the chip and flash it alone. JTAG goes through the CPU's debug core, which then writes Flash on your behalf. SPI is faster; JTAG gives full debug capability on top of flashing.

---

## 2 · Non-Volatile Storage Media
> All three use floating-gate transistors. They differ only in erase granularity.

| Type | Erase granularity | CPU exec (XIP) | Capacity / Cost | Typical use |
|---|---|---|---|---|
| **EEPROM** | Per-byte independent gate — precise overwrite | ✅ yes | KB / very expensive | Config params, meter readings, calibration |
| **NOR Flash** | Block erase; byte-level random read | ✅ yes — address bus wired directly | MB / expensive | Bootloader, code CPU runs at power-on |
| **NAND Flash** | Page read/write; block erase only | ❌ no — must copy to RAM first | GB–TB / very cheap | SSD, eMMC/UFS, full OS + filesystem |

**Shadow RAM trick:** NAND can't be executed directly. Fix: tiny ROM/NOR boots CPU → initialises RAM → bulk-copies OS from NAND into RAM → CPU jumps into RAM and runs at full speed. Net result: NAND's huge capacity and low cost, with RAM's perfect random-access speed.

---

## 3 · Firmware Flashing Methods

| Method | Channel | Needs software? | Use case |
|---|---|---|---|
| **ICP** — In-Circuit Programming | JTAG / SWD | ❌ none — pure hardware | Dev loop (flash + debug together); brick recovery even when clock is misconfigured |
| **ISP** — In-System Programming | UART / USB | ⚠️ Vendor Bootloader #1 (factory ROM, read-only) | Budget dev ($1 cable); production line batch flash; last-resort recovery |
| **IAP** — In-Application Programming | Any — CAN, Ethernet, BLE… | ⚠️ Developer Bootloader #2 (your own code) | Sealed devices (buried sensors, enclosed machines) updated over existing bus |
| **OTA** — Over-The-Air | WiFi / 5G / BLE | ⚠️ Bootloader #2 + A/B dual partition | Consumer electronics — silent download, one-reboot swap, auto-rollback on failure |

---

## 4 · A/B Dual-Partition Anti-Brick

```
[ Bootloader 2 ][ Partition A — running ][ Partition B — OTA downloads here ]
```

1. App runs in **A** → OTA silently downloads new firmware into **B**
2. Reboot → Bootloader 2 switches execution to **B**
3. **B crashes** → auto-rollback to **A**. Never bricks.

---

## 5 · Pull-Up and Pull-Down Resistors

**Problem:** A floating (unconnected) digital input reads random noise — undefined 0 or 1.
**Fix:** Tie the pin to a known voltage through a resistor so it has a default state.

| | Pull-Up | Pull-Down |
|---|---|---|
| Tied to | VCC | GND |
| Default state | HIGH | LOW |
| Activated by | connecting pin to GND | connecting pin to VCC |
| Common value | 1 kΩ – 100 kΩ (10 kΩ typical) | same |

**Why a resistor, not a wire?** When the switch closes it would short VCC→GND directly. The resistor limits current and lets the active signal "win" cleanly.

```
Pull-Up               Pull-Down
  VCC                  Switch→VCC
   |                       |
  [R]                  Input pin
   |                       |
  Input pin               [R]
   |                       |
 Switch→GND               GND
```

- **I²C buses** always use pull-ups (SDA and SCL must rest HIGH).
- Most MCUs (Arduino, STM32, RPi) have **internal pull-ups/pull-downs** configurable in software — no external resistor needed for simple GPIO buttons.

---

## 6 · Transport Stream Processor (TSP)

**What it is:** A dedicated hardware+firmware subsystem inside a DTV SoC that ingests and sorts MPEG-2 Transport Streams in real time — offloading the main ARM CPU entirely.

**Why it exists:** A TS arrives as thousands of interleaved 188-byte packets per second. Sorting them in software would saturate the OS scheduler and kill UI responsiveness.

### Pipeline (simplified)

```
Incoming TS (interleaved 188-byte packets)
        │
        ▼
┌─────────────────────────┐
│  Hardware PID Filters   │──► drop irrelevant PIDs at wire-speed (nanoseconds)
└──────────┬──────────────┘
           ▼
┌─────────────────────────┐
│  TSP Micro-Core         │──► parse PSI/SI tables, manage crypto/CAS keys
│  (MIPS / RISC-V / 8051) │
└──────────┬──────────────┘
           ▼ DMA
  [ System DDR RAM Buffers ]   ← video frames, audio, OTA images land here
```

### TSP Micro-Core — Which CPU and Why?

| Core | When used | Why |
|---|---|---|
| **8051 variant** | Legacy / cost-sensitive designs | Register config + basic routing only; no royalties, tiny silicon area; state machines do the heavy work |
| **MIPS** | Mid-range / older modern chips | 32-bit address space handles large DDR buffers; enough headroom for real-time decryption |
| **RISC-V** | New designs | Same capability as MIPS, royalty-free, open ISA |

### Key Concepts

- **PID filter** — the 13-bit Packet Identifier in each packet header is matched in hardware; unmatched packets are discarded before touching RAM.
- **PSI/SI tables** — network metadata packets (channel list, EPG, scrambling info) parsed by the micro-core, not the main CPU.
- **CAS / descrambling** — Conditional Access System keys loaded by the micro-core; decryption runs on the TSP, never exposing raw keys to the application layer.
- **DMA** — filtered payloads (HD video, OTA firmware blobs) are written directly to DDR; main CPU only gets a "buffer ready" interrupt.

**Design principle:** Hardware handles the high-frequency, predictable work (PID match, DMA). Firmware handles control logic (table parsing, crypto). Main CPU handles none of it — this is hardware-software co-design.

---

## 7 · TSP Firmware — State Machine Architecture

Two layers, two paradigms. Packet pipeline = hardwired silicon FSM. Protocol orchestration = firmware FSM on the micro-core.

```
┌──────────────────────────────────────────────┐
│  Micro-Core Firmware  (software FSMs)         │
│  — channel change, key renewal, OTA           │
└───────────────────┬──────────────────────────┘
                    │ interrupt / register read
┌───────────────────▼──────────────────────────┐
│  Hardware State Machine  (RTL / gates)        │
│  — PID match, continuity counter, DMA push   │
│  — wire-speed, zero firmware cycles           │
└──────────────────────────────────────────────┘
```

### Hardware FSM (silicon)

Runs at line rate — no firmware latency tolerated. Classic Moore/Mealy baked into RTL:

```
WAIT_SYNC → READ_HEADER → FILTER_PID → DMA_PAYLOAD → WAIT_SYNC
```

Handles: `0x47` sync lock, PID extraction + filter lookup, continuity counter, DMA dispatch to correct RAM ring buffer.

### Firmware FSMs (micro-core software)

Higher-level, event-driven sequences with branching and timing:

| FSM | Example states |
|---|---|
| Channel change | `IDLE → REQUEST_PIDS → WAIT_PAT → WAIT_PMT → ARMED` |
| CAS / key renewal | `ACTIVE → ECM_RECEIVED → KEY_PENDING → KEY_LOADED` |
| OTA download | `IDLE → DETECT_DSI → COLLECT_DDB → VERIFY_CRC → SIGNAL_DONE` |

**Why FSMs?** TS processing is inherently sequential and event-driven — can't act on PMT before PAT; can't decrypt before the key arrives. FSMs map directly to "wait for X → do Y → handle error Z." A plain interrupt handler or RTOS task loop becomes unmanageable at this protocol depth.

---

## 8 · VIA Technologies — Work Experience (Sep 2005 – Aug 2008)

**Role:** Advanced Embedded Software Engineer | Shanghai | Windows CE display driver development for VIA/S3 chipsets (UniChrome / Chrome9 series).

### 4 Key Contributions

| # | What | The Point |
|---|---|---|
| 1 | **H/W Acceleration Interface** | Designed interface between overlay and MPEG decoder — decoupled them so each is independent; reused across multiple chipset generations |
| 2 | **Driver Migration CE 5.0 → CE 6.0** | Full port of display driver: DirectDraw, H/W MPEG decoding, Video Port. CE 6.0 had a new kernel process-isolation model — non-trivial migration |
| 3 | **Multi-Chipset Porting** | Ported drivers to new chipsets; validated MPEG-2/4 decoder at **motion compensation level** — compared decoded block output against reference data to verify block prediction math |
| 4 | **AGP Driver Optimization** | Maintained AGP driver; optimized H/W-accelerated MPEG-2/4 decoding; redesigned **AGP GART** and command manager |

### Key Technical Terms

| Term | What it means |
|---|---|
| **DirectDraw** | Windows CE graphics API for H/W-accelerated 2D rendering; sits between driver and display hardware |
| **Video Port** | Dedicated hardware path streaming video data directly to display chip, bypassing CPU |
| **Motion compensation** | Decode step reconstructing frames from predicted block differences (P/B-frames); validated by comparing decoded block output against reference data — verifying the block prediction math, not just functional playback |
| **AGP GART** | Page table mapping scattered physical RAM into contiguous AGP address space; redesigning it = owning the GPU memory management layer |
| **Overlay** | Separate hardware layer composited on top of framebuffer; used for video to bypass the main framebuffer entirely |

### The Key Design Decision (interview story) — H/W Acceleration Interface

**Root cause:** CE 6.0 introduced per-process virtual address spaces (process isolation). The old CE 5.0 model had the decoder writing decoded frames directly into a flip buffer in shared physical memory, which the overlay then flipped to screen. CE 6.0 broke this — decoder and display driver live in different address spaces; the same physical address means different things to each process. Direct buffer sharing stops working.

**The fix:** Redesigned the flip buffer so it no longer holds decoded pixel data. Instead it holds the **address (pointer/handle) of where the decoded data lives**. The decoder decodes into its own buffer and passes the address. The overlay receives the address and reads directly from it — zero copy, no shared buffer ownership.

**The result:** Decoder and overlay are fully decoupled — neither knows about the other's memory layout. The interface was reused across multiple chipset generations.

**Interview framing:** The decoupling wasn't an abstract design preference — it was forced by the CE 6.0 memory model. The address-passing approach was the solution. Shows understanding of *why* the architecture had to change, not just *what* changed.

---

## 9 · VIA Chipset — GPU Hardware Access Methods (x86 Windows CE)

Three completely separate paths. I/O ports and AGP command buffer are **never interchangeable**.

| Method | Target | Who executes | Batchable? |
|---|---|---|---|
| **I/O Ports** (`OUT` instruction) | Display controller — CRTC, DAC, mode-set | CPU only | ❌ Never |
| **MMIO** (via BAR mapping) | GPU engine registers | CPU only | ❌ One-by-one |
| **AGP Command Buffer** | GPU engine operations | GPU command processor (via AGP DMA) | ✅ Whole batch at once |

### I/O Ports — VGA legacy registers
```c
__outbyte(0x3D4, 0x12);   // write index to CRTC index port
__outbyte(0x3D5, value);  // write data to CRTC data port
```
Index/data pair pattern. VIA extended standard VGA indices with proprietary CR/SR registers. Used during mode-set, resolution change, power state. CPU can never delegate these — `OUT` is a CPU instruction only.

### MMIO — direct engine register write
```c
*(volatile uint32_t*)(bar_base + offset) = command;
```
GPU registers mapped into CPU address space via AGP BAR. One write = one immediate engine action.

### AGP Command Buffer — batch mode
```
CPU fills command buffer in RAM (GART-mapped)
    └── GPU engine opcodes: draw, blit, set color, etc.
CPU writes base address + length to MMIO register  ← single kick
    └── GPU issues AGP DMA reads, fetches entire buffer, executes in sequence
```
- **GART** maps scattered physical RAM pages into a contiguous AGP address space the GPU can read
- Buffer contains GPU-specific opcodes — NOT raw MMIO addresses, NOT x86 instructions
- Conceptually: same operations as MMIO writes, but encoded as GPU commands and executed in one burst
- **AGP GART + command manager** = the two components that own this path end-to-end

### Hard boundary
I/O port writes are x86 `OUT` instructions — only the CPU can execute them. No DMA engine, no GPU command processor, no AGP buffer can issue them. I/O ports always stay CPU-driven, always one at a time.

---

## 9a · AGP GART — Deep Dive (VIA Northbridge)

**Core concept:** A hardware MMIO routing unit in the Northbridge (early IOMMU) that tricks the GPU by mapping fragmented, non-contiguous 4 KB system RAM pages as one contiguous, linear block of high-speed VRAM (the AGP Aperture).

### Three-phase operation

**Phase 1 — Aperture setup (once, at init)**
Driver writes a global base address (e.g. `0xE000_0000`) and aperture size (e.g. 16 MB) into Northbridge registers, establishing the fake VRAM window on the bus.

**Phase 2 — Page table fill (load time)**
OS allocates physically fragmented 4 KB pages; driver writes each page's Physical Frame Number sequentially into the GART array. Entry size is hardwired to 4 KB — only the high bits are stored, no size field needed. This is the "Loading…" bar cost.

**Phase 3 — Runtime translation (hardware, nanosecond)**
GPU addresses the aperture linearly. For every bus request, Northbridge hardware splits the address instantly:
- **High bits** → index into GART array → locate the physical page
- **Low 12 bits** → in-page byte offset (passed through unchanged)

Zero CPU involvement at runtime. GPU reads textures directly from system RAM without copying to local VRAM (Direct Texture Execution).

### Why it matters
Expensive pointer-mapping work is front-loaded to load time; runtime cost is pure hardware lookup with nanosecond-level throughput. Architectural predecessor to Intel VT-d / AMD-Vi (IOMMU) and modern GPU Virtual Memory.

---

## 9 · MStar Scaler Driver — Multi-Panel / Multi-Product Support

**Role:** The Scaler is a dedicated hardware block between the video decoder and the display panel. The driver programs its registers to process video correctly for each customer's panel.

### Pipeline position
```
Tuner → Demodulator → TSP → Video Decoder → [Scaler] → Display Panel
```

### Core Scaler functions

| Function | What it does |
|---|---|
| **Scaling** | Resize frame to panel native resolution — e.g. SD 480p source → 1080p panel |
| **NR (Noise Reduction)** | Filter signal noise — critical for weak antenna/cable DTV signals |
| **Color space conversion** | YUV → RGB, BT.601 → BT.709 (SD vs HD color standard) |
| **De-interlacing** | Convert 1080i / 480i → progressive for the panel |
| **Aspect ratio correction** | 4:3 vs 16:9, pillarbox, letterbox, stretch |
| **Picture enhancement** | Sharpness, contrast, gamma via hardware filters |

### Multi-Panel Support — the real job

Different customers (LG, Samsung, Hisense…) use the same MStar SoC with **different panels**. Each panel has its own spec — the driver must be configured per panel:

```c
typedef struct {
    char     *name;
    uint16_t  h_res, v_res;       // native resolution
    uint16_t  hsync_width, vsync_width;
    uint16_t  h_blanking, v_blanking;
    uint32_t  pixel_clock_khz;
    uint8_t   color_matrix[9];    // 3×3 YUV→RGB coefficients
    uint8_t   gamma_table[256];
    // NR params, sharpness, interface type (LVDS/TTL)...
} PanelConfig;
```

- Core driver logic stays **generic**
- Panel differences isolated in a **config table** — one entry per supported panel
- Adding new customer product = add new panel entry + tune parameters

### Interview point
> Every customer product shipped with a different panel. My job included porting the Scaler driver to new panel configs, tuning NR / color matrix / scaling parameters to match each panel's spec, while keeping the core driver architecture clean and reusable.

---

## 10 · Self-Introduction — "Tell me about yourself"

### Structure
```
1. Identity         — one sentence
2. Core experience  — VIA + MediaTek, 2–3 sentences
3. Patent           — don't mumble past it
4. Gap              — one sentence, no apology, pivot immediately
5. Recent work      — proves skills are current
6. Why I'm here     — one sentence, confident close
```

### Script
> I'm a systems software engineer with about ten years of hands-on experience in embedded firmware and low-level driver development.
>
> I started my career building COS — the Card Operating System firmware that runs on the smartcard chip itself — then moved to VIA Technologies where I developed Windows CE display drivers and owned the AGP command manager and hardware acceleration interface for their chipsets. After that I spent five years at MStar Semiconductor — later acquired by MediaTek — working on DTV SoC firmware: OTA upgrade, Transport Stream Processor firmware, GStreamer, and an embedded WebKit browser, which led to a granted patent for a content-loading optimization I invented.
>
> I took time off to raise my first child and immigrate my family to Canada — we arrived in late 2022. Since then I've been working at BicDroid, a security software company, where I've been doing genuine kernel-level work: I extended the eCryptfs Linux kernel module with a custom ACL subsystem in C, and independently designed a Linux Namespace-based encryption architecture implemented in Rust that was adopted for all our banking client deployments.
>
> I'm now looking to return to embedded or systems engineering full-time, where I can apply this depth directly.

### Rules when speaking

| Rule | Why |
|---|---|
| Say **"granted patent"** clearly | It's rare — make the interviewer hear it |
| Gap = one sentence, pivot immediately to BicDroid | Don't linger, don't apologize |
| Say **"kernel-level work"** and **"Rust"** clearly | Signals you're current, not rusty |
| End with what you want — don't trail off | Shows confidence and direction |

---

## 11 · Decision Tree — Which Method to Use?

```
Device boots normally?
├── YES → OTA / software update  (no hardware needed)
└── NO  → JTAG/SWD accessible?
          ├── YES → ICP  (flash + debug, fastest recovery)
          └── NO  → BOOT pin available?
                    ├── YES → ISP  (hold BOOT, serial cable, vendor tool)
                    └── NO  → SPI direct  (clip onto Flash chip, bypass everything)
```

**Speed:** SPI direct > ICP/JTAG. SPI writes Flash without going through the CPU. JTAG is slower because every byte travels: debugger → JTAG scan chain → CPU → bus → Flash. That's why production lines prefer SPI direct for batch flashing.

**Capability:** JTAG/SWD wins everywhere else — single-step, breakpoints, read/write any register or memory address, even with the CPU halted.
