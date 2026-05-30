# Zuzhi (Frank) Kang

Mississauga, Ontario, Canada | zzhikang@gmail.com | +1 647 923 9089

---

## Professional Summary

Senior systems software engineer with over 12 years of hands-on experience in embedded firmware, hardware driver development, and low-level Linux systems programming. Core expertise developed at MediaTek/MStar Semiconductor (DTV SoC firmware, GStreamer multimedia framework, embedded Linux WebKit browser) and VIA Technologies (Windows CE display drivers, hardware-accelerated MPEG decoding) — including a granted patent for WebKit content-loading optimization.

Since immigrating to Canada, has continued to work at the systems level: extended the open-source eCryptfs Linux kernel module with a custom ACL subsystem in C, conceived a Linux Namespace-based encryption architecture (adopted for all banking client deployments), and independently designed a protocol simulation client replicating the full identity authentication protocol across Windows, and Linux client types. Holds an AWS Certified Security – Specialty certification. M.Sc. in Computational Mathematics, Fudan University.

---

## Technical Skills

| Category | Skills |
|---|---|
| **Languages** | C, C++, Assembly, Python, Bash, JavaScript, Rust |
| **Embedded & Firmware** | Firmware development, hardware driver development (Linux, Windows CE) |
| **Linux Kernel & Systems** | Kernel module development, eCryptfs, Linux Namespaces, VFS |
| **Toolchains** | ARM/MIPS cross-compilation, GCC, GNU Debugger, Makefile, CMake, AutoTools |
| **Multimedia** | GStreamer, DirectShow, MPEG-2, MPEG-4, H.264/AVC|
| **Protocols & Standards** | EMV, ISO 7816 (smartcard), SMB, MPEG-TS |
| **Testing & Automation** | pytest, unittest, Jenkins, protocol simulation |
| **Infrastructure & DevOps** | AWS (Security Specialty, CodePipeline, CodeDeploy), Datadog |
| **OS** | Linux, Windows CE/Embedded/Server |

---

## Professional Experience

### Software Development Engineer in Test (SDET)
**BicDroid Inc.** | Waterloo, Ontario, Canada | Oct 2024 – Present

Developed security and encryption products under direction of a systems security professor.

- Extended the open-source **eCryptfs Linux kernel module** with a custom ACL subsystem in C: per-file access control enforced at kernel level with persistent rules, directory inheritance, and no LSM dependency
- Conceived and implemented a **Linux Namespace-based encryption architecture** in Rust, enabling simultaneous plaintext/ciphertext access — adopted for all banking client deployments
- Built a **pytest automation framework** for encryption driver validation covering 6 OS versions × 3 SMB protocol versions (18+ combinations)
- Designed and implemented a **protocol simulation client** for the identity authentication platform — independently replicating the core authentication protocol across all client types (Android, Windows, Linux) with concurrent multi-user flows; became the core infrastructure for exercising the full authentication system under realistic load conditions

---

### Software Engineer Intern
**Kama.AI** | Mississauga, Ontario, Canada | Dec 2023 – Sep 2024

- Investigated and fully resolved a **production security breach**: identified root cause, contained the incident, and delivered a complete incident report to the client; achieved **AWS Certified Security – Specialty** certification through this engagement
- Designed and implemented the company's CI/CD release pipeline; integrated real-time product monitoring with Datadog

---

### Parental Leave & Immigration to Canada
**Sep 2013 – Nov 2022**

Served as primary caregiver for my first child from birth. Concurrently planned and executed family immigration to Canada — a multi-year process further extended by COVID-19 travel restrictions. We arrived in Canada in November 2022.

---

### Senior Software Engineer
**MediaTek Inc.** *(formerly MStar Semiconductor, Inc.; acquired 2012)* | Shanghai, China | Aug 2008 – Sep 2013

MStar Semiconductor was a global leader in DTV SoC design. Responsible for firmware, middleware, and driver development across multiple chipset generations.

- **OTA Firmware Upgrade (LGE HK51):** Designed and implemented Over-the-Air DTV firmware upgrade for the LG Electronics T2 HK51 project; provided on-site technical support through mass production
- **WebKit Browser — Patent Holder:** Developed a WebKit-based browser for DTV on embedded Linux; invented a novel webpage content-loading optimization — granted patent: *"WebKit browser webpage content loading method and device"*
- **GStreamer Multimedia Framework:** Developed and maintained GStreamer-based multimedia framework on DTV platforms, enabling broad media format support across video player applications
- **Hardware Drivers:** Maintained Mstar Scaler driver (video signal processing: scaling, NR, color space conversion to LCD panel), Graphics Engine driver (UI/OSD/Subtitle rendering), and TSP (Transport Stream Processor) firmware; ported TSP to new chipset generations

---

### Advanced Embedded Software Engineer
**VIA Technologies Inc.** | Shanghai, China | Sep 2005 – Aug 2008

Developed embedded display drivers for Windows CE on VIA SoC platforms, covering the full stack from AGP hardware to DirectDraw/DirectShow multimedia pipeline.

- **H/W Acceleration Interface:** Designed and implemented a hardware acceleration interface for the video decoder subsystem, decoupling H/W overlay management from MPEG decoding routines and enabling modular reuse across chipsets
- **Driver Migration CE 5.0 → CE 6.0:** Led complete migration of the display driver, covering DirectDraw implementation, hardware MPEG decoding, and Video Port support
- **Multi-Chipset Porting:** Ported display drivers to multiple new chipsets; implemented MPEG-2/4 decoder validation at motion compensation level
- **AGP Driver Optimization:** Maintained and optimized AGP driver including hardware-accelerated MPEG-2/4 decoding routines and AGP GART/command manager redesign; resolved overlay rendering bugs in the new driver version

---

### Smartcard Software Engineer
**Shanghai COS Software Co., Ltd.** | Shanghai, China | Jul 2002 – May 2005

Developed bare-metal firmware for smartcard Chip Operating Systems (COS) on 8051-class microcontrollers in Assembly and C, targeting severely resource-constrained environments (KB-range RAM/ROM).

- **ISO 7816 Communication Driver:** Implemented smartcard communication protocol driver in Assembly on 8051 architecture, handling APDU command dispatch and low-level UART framing
- **Multi-Product Firmware:** Developed and formally certified COS firmware for five smartcard product lines — Social Security Card, GlobalPlatform Java Card, CMCC SIM, China Mobile Wuyou Assistant, and STK Downloading

---

## Education

**Master of Science — Computational Mathematics**
Institute of Mathematics, Fudan University, Shanghai | 1999 – 2002

**Bachelor of Science — Computational Mathematics & Application Software**
Department of Mathematics, Fudan University, Shanghai | 1995 – 1999

---

## Certifications & Patents

- **AWS Certified Security – Specialty** — Amazon Web Services
- **Patent:** *"WebKit browser webpage content loading method and device"* — MStar Semiconductor / MediaTek
