# Embedded / DTV Terms Reference

## Core Communication Protocols

### UART — Universal Asynchronous Receiver/Transmitter
A **serial communication standard** — the simplest way two devices talk to each other.

- "Asynchronous" means no shared clock wire; both sides agree on speed (baud rate) beforehand
- **2 data wires:** TX (transmit) and RX (receive) — plus GND
- Speed expressed in **baud rate** (e.g. 115200 baud = ~115,200 bits/sec)
- On a PC: used to be the RS-232 COM port; today you use a USB-to-UART adapter chip (CP2102, CH340, FT232)
- On embedded/DTV boards: the SoC has a built-in UART hardware block; U-Boot and Linux kernel print their logs through it
- **Think of it as:** a plain text pipe between the chip and your PC terminal (like PuTTY or minicom)

---

### IIC / I²C — Inter-Integrated Circuit
A **2-wire serial bus** designed for short-distance chip-to-chip communication on the same board.

- Invented by Philips (now NXP) in the 1980s
- **2 signal wires:** SDA (data) + SCL (clock) — plus GND
- One master (e.g. the CPU), multiple slaves (e.g. EEPROM, sensors, scaler registers) — each slave has a 7-bit address
- Slower than SPI but needs fewer wires; good for register access, config, small data
- "IIC" is just another spelling of "I²C" — same thing, used interchangeably in Chinese/Asian chip industry documentation
- **On MStar:** the chip exposed its internal register map over I²C; the MSTOOL02 dongle is essentially a USB-to-I²C bridge

---

### SPI — Serial Peripheral Interface
A **4-wire high-speed serial bus**, faster than I²C, used for flash memory and displays.

- **4 wires:** MOSI (master out), MISO (master in), SCLK (clock), CS/SS (chip select)
- Full duplex (send and receive simultaneously), no addressing — chip select pin picks the slave
- The NOR flash chip that stores your DTV firmware connects to the SoC via SPI
- SPI programmers (CH341A, Dediprog) talk directly to the flash chip over these 4 wires

---

### JTAG — Joint Test Action Group (IEEE 1149.1)
A **debug and test interface** standard, originally designed for board-level chip testing (boundary scan), later adopted as the universal CPU debug port.

- **4 wires:** TCK (clock), TMS (mode select), TDI (data in), TDO (data out) + optional TRST (reset)
- Lets a debugger: halt the CPU, step through instructions, read/write registers and memory, set breakpoints, program flash
- Implemented in hardware inside the CPU — a separate dedicated state machine, independent of the main core
- **EJTAG** = MIPS-specific extension of JTAG, adds hardware breakpoints, watchpoints, fast data access channel

---

### SWD — Serial Wire Debug
ARM's **2-wire replacement for JTAG**, same capability with fewer pins.

- Only on ARM Cortex cores
- **2 wires:** SWDIO (data, bidirectional) + SWDCLK (clock)
- Same tools (J-Link, etc.) support both JTAG and SWD

---

## Programming / Flashing Terms

### ISP — In-System Programming
A **method** of programming (flashing) a chip's firmware while it is already mounted on the board, without removing it.

- "In-system" = you don't pull the chip out to program it in a separate socket
- The chip enters a special ISP boot mode (usually triggered by a pin state or command at power-on)
- In MStar's case: ISP mode is entered via the IIC interface — the MSTOOL02 sends the firmware image over I²C directly into flash
- ISP is a concept/method, not a specific protocol — the underlying transport can be UART, I²C, SPI, USB, etc.

---

### ICP — In-Circuit Programming
Very similar to ISP; sometimes used interchangeably. Subtle difference:

- **ISP** typically means the chip's own built-in bootloader handles the programming
- **ICP** sometimes implies a dedicated programming interface (like JTAG) that bypasses the chip's firmware entirely
- In practice at MStar/MTK the terms were used loosely

---

### OTA — Over-The-Air Update
Firmware upgrade delivered **wirelessly** (broadcast signal, internet, etc.) rather than via a physical debug cable.

- On DTV: the new firmware image is embedded in the broadcast transport stream (DVB-T/T2/C/S)
- The TV's TSP extracts the firmware package from the stream
- A dedicated OTA manager application verifies, stores, and applies the update
- "Over-the-air" contrasts with "in-factory ISP" — same end result (new firmware in flash), completely different delivery path

---

## Hardware Blocks Inside the SoC

### TSP — Transport Stream Processor
A dedicated **hardware + firmware block** inside the DTV SoC that parses the MPEG-2 Transport Stream coming from the tuner.

- Demultiplexes the stream: separates video, audio, subtitles, EPG data, OTA firmware packages by PID (Packet ID)
- Usually runs its own small firmware on a separate MIPS core inside the SoC
- Low-level, real-time; runs independent of the main application CPU

---

### OSD — On-Screen Display
The **graphics overlay** rendered on top of the video — menus, subtitles, channel info, volume bar.

- Driven by a dedicated graphics engine inside the SoC
- The OSD driver programs the hardware registers to define layers, color formats, alpha blending, source buffers

---

### Scaler
The **video signal processing block** that resizes, deinterlaces, and color-corrects the incoming video to match the panel resolution.

- A major block in MStar's TV chips — central to their product differentiation
- Configured entirely via register writes (which is why live register access via the IIC dongle was so useful)

---

## Summary Table

| Term | Type | One-line meaning |
|------|------|-----------------|
| UART | Protocol | Simple async serial — the debug text console |
| I²C / IIC | Protocol | 2-wire bus for register access and config |
| SPI | Protocol | 4-wire fast bus — used for NOR flash |
| JTAG | Debug interface | Full CPU debug: halt, step, memory R/W |
| EJTAG | Debug interface | JTAG extended for MIPS cores |
| SWD | Debug interface | 2-wire JTAG alternative, ARM only |
| ISP | Method | Flash firmware onto a chip already on the board |
| ICP | Method | Similar to ISP, often via JTAG-level access |
| OTA | Method | Deliver firmware update wirelessly (broadcast/internet) |
| TSP | HW block | Parses broadcast transport stream inside the SoC |
| OSD | HW block | On-screen graphics overlay engine |
| Scaler | HW block | Video resize/deinterlace/color processing |
