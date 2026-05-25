# Zuzhi (Frank) Kang

Mississauga, Ontario, Canada | zzhikang@gmail.com | +1 647 923 9089

---

## Professional Summary

Senior systems software engineer with over a decade of hands-on experience in embedded firmware, hardware driver development, and low-level Linux systems programming. Core expertise built at VIA Technologies (Windows CE display drivers, hardware-accelerated MPEG decoding) and MediaTek/MStar Semiconductor (DTV SoC firmware, GStreamer multimedia framework, embedded Linux WebKit browser) — including a granted patent for a WebKit content-loading optimization.

Since immigrating to Canada, has continued to work at the systems level: extended a Linux kernel module with a custom ACL subsystem in C, designed a Linux Namespace-based encryption architecture implemented in Rust (adopted for all banking client deployments), and built a fanotify-based file monitoring tool through independent kernel API research. Holds an AWS Certified Security – Specialty certification. M.Sc. in Computational Mathematics, Fudan University.

---

## Technical Skills

| Category | Skills |
|---|---|
| **Languages** | C, C++, Rust, Assembly (ARM/MIPS), Python, Java, Bash, JavaScript |
| **Embedded & Firmware** | Firmware development, hardware driver development (Linux, Windows CE) |
| **Linux Kernel & Systems** | Kernel module development, eCryptfs, fanotify, Linux Namespaces, VFS, address_space |
| **Toolchains** | ARM/MIPS cross-compilation, GCC, GNU Debugger, Makefile, CMake, AutoTools |
| **Multimedia** | MPEG-2, MPEG-4, H.264/AVC, GStreamer, WebKit, DirectDraw |
| **Protocols & Standards** | ISO 7816 (smartcard), GSM, EMV, SMB, iSCSI, MPEG-TS |
| **Testing & Automation** | pytest, unittest, Jenkins, protocol simulation, stress testing |
| **Cloud** | AWS (Security Specialty, CodePipeline, CodeDeploy), Datadog |
| **OS** | Linux, Windows CE/Embedded/Server |

---

## Professional Experience

### Software Development Engineer in Test (SDET)
**BicDroid Inc.** | Waterloo, Ontario, Canada | Oct 2024 – Present

BicDroid develops enterprise-grade data security products, including transparent encryption drivers for Windows and Linux, and an identity authentication platform. Engineering projects are directed by a professor of systems security who leads the company.

**Linux Kernel Development**

- Extended the open-source **eCryptfs Linux kernel module** with a custom, self-contained ACL subsystem (C, co-developed with AI tooling): controls who (user / group / process) can access which encrypted files and in what form (plaintext or ciphertext), enforced at the kernel level with persistent rules, directory inheritance, and no dependency on external LSM frameworks (SELinux / AppArmor)

- Built a **file access monitoring tool** using the Linux **fanotify** API (C, independent research): monitors file access events on specified directories and tracks file move operations across designated mount points

**Systems Architecture & Innovation**

- Conceived a **Linux Namespace-based encryption architecture** to simultaneously expose plaintext and ciphertext views of encrypted data without modifying client applications (Rust, Linux; architecture proposed independently, implemented with AI tooling) — addressing the requirement for dual-access capability in banking client deployments

- Designed an **iSCSI-based remote encrypted storage architecture** delivering transparently encrypted block storage over iSCSI, enabling secure remote file storage for client environments (independent initiative) — adopted as the baseline storage architecture for banking client deployments

**Automation & Monitoring**

- Built **AccessTracker**: automated file and process access monitor for Windows, leveraging ProcMon as the capture engine
- Implemented **Jenkins CI/CD pipelines** on Linux for product build and test automation
- Built a **pytest automation framework** from scratch for Windows encryption driver NAS share protection validation:
  - Covers 6 Windows Server versions (2008R2 → Server 2022) × 3 SMB protocol versions = 18+ test matrix combinations; designed to scale without structural changes
- Designed a **protocol simulation client** for the identity authentication platform:
  - Simulates all client types (Android, Windows, Linux); launches concurrent multi-user authentication flows
  - Enables both functional flow validation and full system stress testing; all production validation was performed using this tool

---

### Cloud Engineer
**Kama.AI** | Mississauga, Ontario, Canada | Dec 2023 – Sep 2024

Kama.AI develops AI-powered conversational assistant products for enterprise clients.

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
- **Hardware Drivers:** Maintained the Mstar Scaler driver (video signal processing: scaling, NR, color space conversion), Graphics Engine (GE) driver (UI/OSD/Subtitle), and TSP (Transport Stream Processor) firmware

---

### Advanced Embedded Software Engineer
**VIA Technologies Inc.** | Shanghai, China | Sep 2005 – Aug 2008

Responsible for embedded display driver development for Windows CE platforms.

- **H/W Acceleration Interface:** Designed and implemented a hardware acceleration interface for the video decoder subsystem, decoupling H/W overlay management from MPEG decoding routines; reused across multiple chipset projects
- **Driver Migration CE 5.0 → CE 6.0:** Led complete migration of the display driver, covering DirectDraw implementation, hardware MPEG decoding, and Video Port support
- **Multi-Chipset Porting:** Ported display drivers to multiple new chipsets; implemented MPEG-2/4 decoder validation at motion compensation level
- **AGP Driver Optimization:** Maintained and optimized AGP driver including hardware-accelerated MPEG-2/4 decoding routines and AGP GART/command manager redesign

---

### Smartcard Software Engineer
**Shanghai COS Software Co., Ltd.** | Shanghai, China | Jul 2002 – May 2005

- Designed and implemented COS low-level driver per **ISO 7816 Protocol (T=0)**
- Developed and certified multiple Smartcard product lines: Social Security Card, GlobalPlatform Java Card, CMCC SIM Card, China Mobile Wuyou Assistant, STK Downloading
- Built a Python `unittest`-based automated validation framework for Smartcard OS development

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
