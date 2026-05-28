# MStar Debug Hardware & Protocols

## MStar IIC ISP Dongle — MSTOOL02

**Product name:** MSTOOL02 "Original MStar Burner Programmer Debug Tool"
(Still sold on Alibaba as of 2025 — used/new units available from resellers.)

The primary debug/programming tool used daily at MStar Semiconductor for DTV SoC work.

**Physical form:**
- Small square PCB (~3–5 cm per side)
- USB (Type-A or mini-USB) to PC
- Short cable to target EVB board — 3 or 4 wires:

| Pin | Signal | Notes |
|-----|--------|-------|
| SDA | I²C data | bidirectional |
| SCL | I²C clock | |
| GND | Ground | shared reference |
| VCC | 3.3V | optional — target often self-powers |

Target-side connector: a dedicated `IIC_DEBUG` or `ISP` header on the EVB board next to the main SoC.

**Why so few wires:** MStar's debug protocol rides on top of standard I²C, which only needs 2 signal lines. The chip's I²C bus was already present for DDC (display data channel) — the debug interface reused the same bus.

---

## What you could do with it

| Function | Detail |
|----------|--------|
| **Firmware download** | IIC boot mode — chip enters ISP mode at power-on; tool streams binary over I²C to internal SRAM or SPI flash |
| **Debug message output** | Reads chip's internal log buffer over I²C; PC tool displays it like a serial console |
| **Register / memory R/W** | Direct read/write to internal register map or DRAM via I²C commands — primary way to tune scaler, video path, OSD engine settings live without reflashing |
| **E-fuse / OTP access** | Some chips: read/burn one-time-programmable security fuses |

---

## PC-side Tool

MStar provided a proprietary Windows GUI tool — called **MStar ISP Tool** or **MSD ISP** — wrapping all the above functions. It talked to the dongle via USB HID or CDC driver; the dongle translated to I²C on the chip side. Command-line variants were used in production flashing scripts.

---

## Why IIC, not JTAG

MStar TV SoC family (MSD series, T-series) used I²C for ISP/debug because:
- Fewer package pins needed
- I²C already on-chip for DDC — repurposed for debug
- A cheap USB-I²C bridge dongle was enough — no expensive JTAG license hardware needed on production lines

JTAG/EJTAG was present on the internal MIPS cores but locked down or not exposed on most EVB boards outside core platform teams. Day-to-day firmware and driver work used the IIC dongle.

---

## Other Debug Interfaces (broader embedded context)

### JTAG (IEEE 1149.1)
- 4 wires: TDI, TDO, TMS, TCK + optional TRST
- CPU halt/step, register/memory R/W, flash programming, boundary scan
- Adapters: Segger J-Link, Lauterbach TRACE32, FTDI-based OpenOCD dongles

### MIPS EJTAG (JTAG extension for MIPS cores)
- Same 4 JTAG wires, adds hardware breakpoints, watchpoints, fastdata channel
- Used on MStar's MIPS cores (e.g. TSP firmware core)
- Lauterbach TRACE32 was the premium tool for this; OpenOCD also supports it

### SWD (ARM Serial Wire Debug)
- 2 wires: SWDIO + SWDCLK — ARM Cortex alternative to JTAG
- Same capability as JTAG for ARM cores; used on ARM subsystems in later MediaTek chips

### ETM / ETB (Embedded Trace)
- Non-intrusive instruction-level execution trace
- ETB: on-chip trace buffer (no external hardware needed)
- ETM + TPIU: streams trace out via high-speed parallel port to Lauterbach or DS-5 probe
- Useful for race conditions or boot-path bugs where halting breaks timing

### UART / Serial Console
- 3.3V TTL UART via USB-to-Serial adapter (CP2102, CH340, FTDI FT232)
- Dedicated UART header on EVB board
- Used for: U-Boot interaction, kernel console, `printk`/`dmesg`, TSP log ring buffer

### SPI Flash Programmer
- Connects directly to NOR flash chip via 4 wires (MOSI, MISO, CLK, CS)
- Tools: CH341A, Dediprog SF100
- Used for firmware recovery when the chip itself is unresponsive

---

## How these mapped to MStar DTV work

| Area | Primary debug method |
|------|---------------------|
| Graphics / OSD / Scaler driver | IIC dongle (live register tuning) + UART log |
| TSP firmware | MIPS EJTAG or IIC dongle + UART log ring buffer |
| OTA firmware upgrade | UART console during flash, SPI programmer for recovery |
| WebKit / GStreamer on Linux | `printk` / `dmesg` over UART console |
