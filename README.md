# Dell Latitude 5420 — OpenCore EFI (Tiger Lake / Iris Xe)

OpenCore **1.0.8** configuration for **Dell Latitude 5420** with **Intel Core i5-1145G7**, **Iris Xe** (`8086:9A49`), and **macOS Sequoia 15.8.2 (24H232)**. This repo is the daily-driver EFI used on real hardware (Metal 2, Wi‑Fi, Ethernet, Bluetooth, audio, I2C touchpad and touchscreen). Sleep and hibernate are intentionally disabled on BIOS **1.55.0**; video on **24H232** still has open issues with **WhateverXe-Graphics** (see [Known issues](#known-issues)).

> **Disclaimer:** Hackintosh builds may violate the Apple macOS license agreement. This repository is shared for education and troubleshooting. You assume all risk. Do not redistribute Apple proprietary files or WhateverXe/Graphics payloads—only OpenCore, ACPI, and third-party kexts that their licenses allow.

---

## Table of contents

- [Hardware tested](#hardware-tested)
- [Software versions](#software-versions)
- [What works](#what-works)
- [Known issues](#known-issues)
- [Repository layout](#repository-layout)
- [BIOS settings](#bios-settings)
- [Installation](#installation)
- [Post-install (required for a usable desktop)](#post-install-required-for-a-usable-desktop)
- [Video: two profiles on 24H232](#video-two-profiles-on-24232)
- [Verification commands](#verification-commands)
- [ACPI and DSDT patches](#acpi-and-dsdt-patches)
- [Kexts (OpenCore `config.plist`)](#kexts-opencore-configplist)
- [Boot arguments](#boot-arguments)
- [Sleep and power management](#sleep-and-power-management)
- [Updating macOS or WhateverXe](#updating-macos-or-whateverxe)
- [Credits](#credits)
- [Related links](#related-links)

---

## Hardware tested

| Component | Details |
| --- | --- |
| Model | Dell Latitude **5420**, SKU **0A20**, BIOS **1.55.0** (2026-05-26) |
| CPU | Intel Core **i5-1145G7** (Tiger Lake-U, 4C/8T) |
| iGPU | Intel **Iris Xe**, device ID **0x9A49**, 80 EU, DVMT **64 MB** in firmware |
| RAM | **16 GB** DDR4-3200 (single SK hynix SO-DIMM, single-channel) |
| Storage | KIOXIA NVMe (OpenCore on internal EFI partition **`/EFI`**) |
| Panel | 14" **1920×1080** @ 60 Hz (AUO eDP) |
| Wi‑Fi / BT | Intel **AX201** CNVi (`8086:A0F0`) |
| Ethernet | Intel **I219-LM** (`8086:15FB`) |
| Audio | Realtek **ALC236** (layout **69**) |
| Touchpad | Dell **TPD0** I2C precision touchpad (`DELL0A20`) |
| Touchscreen | ELAN **04F3:2846** on **TPL0** (I2C) |
| Keyboard | PS/2 (VoodooPS2) |
| Other | Realtek RTS525A SD reader, Thunderbolt 4 / USB4 (not fully validated) |

SMBIOS: **MacBookPro16,2** with CPUID spoof to **Ice Lake-U** (required for stable boot on this firmware).

---

## Software versions

| Item | Version |
| --- | --- |
| macOS | **Sequoia 15.8.2**, build **24H232** |
| OpenCore | **1.0.8** RELEASE |
| Lilu | **1.7.2** |
| VirtualSMC | **1.3.8** (+ battery / processor / light sensor plugins) |
| WhateverXe | **0.6.1** with **24H232 call-target fix** (see [Post-install](#post-install-required-for-a-usable-desktop)) |
| WhateverXe-Graphics | **0.4.0** (`.pkg` installed in macOS, not in this EFI folder) |
| AMFIPass | **1.4.1** |
| AirportItlwm | **2.4.0** |
| IntelMausi | **1.0.8** |
| IntelBluetoothFirmware | **2.5.0** + **IntelBTPatcher 2.5.2** (lshbluesky fork) + **BlueToolFixup 2.6.9** |
| AppleALC | **1.9.8** |
| VoodooPS2Controller | **2.3.7** |
| VoodooI2C / VoodooI2CHID | **master CI `6be653a2`** (Tiger Lake; **2.9.1 release does not load** on this machine) |
| ECEnabler | (bundled with ACPI set) |

Reference `config.plist` in the maintainer’s Docs tree: MD5 **`93eb00cda8a98bbdce1ce5a1d0d4aa4c`** (sleep/hibernation experiments disabled, no `-wex-nogva` in boot-args).

---

## What works

| Feature | Notes |
| --- | --- |
| **Metal 2** / Iris Xe acceleration | WhateverXe + Graphics **0.4.0** after call-fix kext and post-install steps |
| **Internal display** | Native **1920×1080** |
| **Wi‑Fi** | AirportItlwm, 802.11ac; Heliport not required on this setup |
| **Ethernet** | IntelMausi on **en0** |
| **Bluetooth** | Audio and **BLE mouse** (2.5.2 patcher); pair keyboard after mouse fix |
| **Speakers / mic / volume keys** | AppleALC **layout 69** (`alcid=69` + `layout-id` 69) |
| **Touchpad** | Full gestures (VoodooI2C CI + **TPD0** DSDT patches) |
| **Touchscreen** | ELAN I2C (**TPL0** DSDT patches) |
| **Battery** | Percentage, AC state, Battery pane (with Metal LaunchAgents below) |
| **Menu bar / Control Center** | Requires **`MTL_ERROR_MODE=nslog`** LaunchAgents (see Post-install) |

---

## Known issues

| Issue | Status |
| --- | --- |
| **Sleep / lid close / hibernate** | **Not supported** on BIOS 1.55.0 (Modern Standby, no S3 resume). Use **`pmset disablesleep 1`** and **Shut Down**. |
| **Hardware video decode on 24H232** | Graphics **0.4.0** VA driver can **SIGSEGV** in WindowServer / `mediaanalysisd` / WebKit GPU; App Store can **hang the GPU**. Tracked upstream: [WhateverXe #23](https://github.com/b00t0x/WhateverXe/issues/23). |
| **Apple TV full screen** | Crashes WindowServer even with software-decode workaround; windowed trailers OK with workaround. |
| **AirDrop / AWDL** | Not available with Intel Wi‑Fi |
| **Fingerprint reader** | No driver |
| **WhateverXe 0.7.0** | GT2 gate rejects this **6 L3 bank** fuse shape; stay on **0.6.1 call-fix** until upstream admits the part |

---

## Repository layout

```
.
├── EFI/
│   └── OC/
│       ├── ACPI/          # SSDTs for 5420 (PC00 scope)
│       ├── Kexts/         # OpenCore kexts (not the Graphics .pkg)
│       ├── config.plist   # Main config (verify MD5 before publishing)
│       ├── config-install.plist   # Optional: installer stage (optional)
│       └── Drivers/       # OpenRuntime, OpenHfsPlus, etc.
├── PostInstall/
│   ├── LaunchAgents/      # Metal menu-bar fix (copy to ~/Library/LaunchAgents)
│   └── pmset-notes.md     # Sleep disabled commands
├── Docs/
│   └── VIDEO-PROFILES.md  # Stock vs -wex-nogva + park VA bundle
└── README.md              # This file
```

Before publishing, **remove** personal serials from `PlatformInfo`, scrub OpenCore logs, and confirm **`WhateverXe.kext`** is the **call-fix** binary (SHA-256 **`e0b1ceea1d77c138834867e50177bc6eb297b0c3eda4d344cb47be07f764be06`** for the maintainer’s build—not stock 0.6.1 on 24H232).

---

## BIOS settings

| Setting | Value |
| --- | --- |
| **TPM** | Disabled (or configure per your OpenCore SIP plan) |
| **VT-d** | Disabled (or enabled with `dart=0` if you change config) |
| **Secure Boot** | Disabled |
| **SATA mode** | AHCI |
| **Block Sleep** | Optional; macOS also uses `disablesleep` |

DVMT Pre-Allocated: **64 MB** (checked via firmware / OpenCore setup).

---

## Installation

### 1. Create the macOS installer

Follow [Dortania](https://dortania.github.io/OpenCore-Install-Guide/) for **Tiger Lake laptop**. Use **`config-install.plist`** (or a minimal EFI without GPU acceleration) if the installer needs the firmware framebuffer; this project’s installer path removed early GPU kext load until Sequoia is on disk.

### 2. Install macOS Sequoia **15.8.2 (24H232)**

Install to the internal NVMe. First boot after install may use a reduced kext set until WhateverXe-Graphics is installed.

### 3. Copy this EFI to the system ESP

Mount the internal EFI partition (example):

```bash
diskutil list
sudo diskutil mount disk0s1
sudo cp -R /path/to/repo/EFI /Volumes/EFI/
```

Use **`/Volumes/EFI/EFI/`** (standard layout). The maintainer boots **`/EFI/OC/config.plist`** on **`disk0s1`** without a USB stick.

### 4. Approve kexts and reboot

First boot after copying EFI: approve kexts in System Settings if prompted, then reboot once.

---

## Post-install (required for a usable desktop)

These steps are **not** stored only in the EFI folder.

### A. WhateverXe **call-fix** kext (24H232)

Stock **WhateverXe 0.6.1** fails **`TGLAcceleratorAdmissionReady`** on **24H232** (kernel `call` displacement at `_IOProtectCacheMode+0xCA`). Rebuild **0.6.1** with the call-target check described in [b00t0x/WhateverXe](https://github.com/b00t0x/WhateverXe) issues/discussions, or use the maintainer’s **`WEX-kit`** build script, then replace **`EFI/OC/Kexts/WhateverXe.kext`**.

Verify after reboot:

```bash
ioreg -lw0 -d1 | grep TGLAcceleratorAdmissionReady
system_profiler SPDisplaysDataType | grep Metal
```

Expect **`TGLAcceleratorAdmissionReady = Yes`** and **Metal 2**.

### B. WhateverXe-Graphics **0.4.0** package

Download **`WhateverXe-Graphics-0.4.0.pkg`** from [b00t0x/WhateverXe-Graphics releases](https://github.com/b00t0x/WhateverXe-Graphics/releases). Install in macOS (installs `/Library/GPUBundles/` drivers). Reboot.

### C. Metal UI LaunchAgents (Control Center / Battery pane)

Copy from **`PostInstall/LaunchAgents/`** to **`~/Library/LaunchAgents/`**:

| File | Purpose |
| --- | --- |
| **`local.wex.metal-env.plist`** | Sets **`MTL_ERROR_MODE=nslog`** for GUI apps |
| **`local.wex.metal-env-xpc.plist`** | Sets **`__XPC_MTL_ERROR_MODE=nslog`** and quits System Settings if reopened before agents run |

Log out and back in (or reboot). Verify:

```bash
launchctl getenv MTL_ERROR_MODE
launchctl getenv __XPC_MTL_ERROR_MODE
pgrep -lx ControlCenter
```

### D. Bluetooth NVRAM (if pairing fails)

`config.plist` may include NVRAM entries for **`bluetoothInternalControllerInfo`** / **`bluetoothExternalDongleFailed`** (Intel BT on Sequoia). After changing NVRAM, reset NVRAM once if Bluetooth is unstable.

### E. Disable sleep

```bash
sudo pmset -a disablesleep 1 hibernatemode 0
pmset -g | grep -E 'SleepDisabled|hibernatemode'
```

---

## Video: two profiles on 24H232

See **`Docs/VIDEO-PROFILES.md`**. Summary:

| Profile | When to use | Trade-off |
| --- | --- | --- |
| **Stable (recommended)** | Daily use, Music / App Store / Safari / windowed TV | **`boot-arg -wex-nogva`** + park **`AppleIntelTGLGraphicsVADriver.bundle`**; **software** video decode; less crash risk |
| **Stock** | Testing upstream fixes, hardware decode | VA driver in **`/Library/GPUBundles/`**, no **`-wex-nogva`**; **issue #23** crashes may return |

Upstream work: [WhateverXe issue #23](https://github.com/b00t0x/WhateverXe/issues/23).

---

## Verification commands

```bash
# OpenCore / kexts
kextstat | grep -E 'Lilu|WhateverXe|AppleALC|VoodooI2C|IntelMausi|AirportItlwm'

# Graphics
system_profiler SPDisplaysDataType | grep -E 'Chipset|Metal|Resolution'
ioreg -lw0 -d1 | grep -E 'TGLAcceleratorAdmissionReady|TGLNoGVA'

# Network
ifconfig en0
networksetup -listallhardwareports | grep -A2 Ethernet

# Input
ioreg -r -n TPD0 -n TPL0 | grep -E 'IOClass|name'
hidutil list | head -20

# Audio
ioreg -lw0 -r -c AppleHDAEngineOutput | head -5

# Power
pmset -g
```

---

## ACPI and DSDT patches

**SSDTs** (scope **`\_SB.PC00`**, generated for this DSDT—not generic **`PCI0`** prebuilts):

| SSDT | Role |
| --- | --- |
| **SSDT-EC-USBX-5420.aml** | Fake EC + USB power |
| **SSDT-RHUB-5420.aml** | RHUB for USB |
| **SSDT-PNLF-5420.aml** | Backlight (brightness keys may still need testing) |
| **SSDT-AWAC.aml** | AWAC clock |
| **SSDT-PLUG-DRTNIA.aml** | CPU power management |

**OpenCore ACPI patches** (8 entries in reference config): lower **`OSYS`** gate on **`TPD0`** and **`TPL0`** so macOS receives I2C + GPIO resources (Dell DSDT gates them on “Windows version”). Sources and **`tp-fix.sh` / `ts-fix.sh`** live in the maintainer’s private **`WEX-070`** / touchpad notes; document patch hex in repo **`Docs/ACPI-PATCHES.md`** if you export them.

---

## Kexts (OpenCore `config.plist`)

Enabled in reference **`93eb00cd…`** config:

- Lilu, VirtualSMC (+ SMC plugins), **WhateverXe**, AMFIPass  
- AirportItlwm, IntelMausi  
- IntelBluetoothFirmware, IntelBTPatcher, BlueToolFixup  
- AppleALC, ECEnabler  
- VoodooPS2Controller (+ plugins), VoodooI2C (+ GPIO, services), VoodooI2CHID  

**HibernationFixup** may be present in the folder but **disabled** in the daily config after sleep/hibernate tests failed.

---

## Boot arguments

Reference daily **`boot-args`** (no `-v` in production config):

```
keepsyms=1 debug=0x100 alcid=69 itlwm_he=1 -igfxdvmt -allow3d -disablegfxfirmware msgbuf=1048576
```

| Flag | Purpose |
| --- | --- |
| **`alcid=69`** | ALC236 layout (with AppleALC **`layout-id` 69** in config) |
| **`itlwm_he=1`** | AirportItlwm high-efficiency / 802.11ax path |
| **`-igfxdvmt`**, **`-allow3d`**, **`-disablegfxfirmware`** | WhateverXe Tiger Lake framebuffer |
| **`msgbuf=1048576`** | Larger kernel log buffer for debugging |

Optional (profiles):

- **`-wex-nogva`** — removes GVA keys from IntelAccelerator after start (stable video profile)  
- **`-v`** — verbose boot while debugging  

---

## Sleep and power management

| Test | Result on BIOS 1.55.0 |
| --- | --- |
| S3 sleep | DSDT **`SS3 = Zero`**; forced S3 → firmware **restarts** (~21 s, **`pmset 0x1F`**) |
| Hibernate | Image write sometimes works; **resume** → frozen **blue screen** or no **`boot-image`** in NVRAM |

**Daily rule:** **`sudo pmset -a disablesleep 1`**, **`hibernatemode 0`**, use **Shut Down** instead of Sleep.

---

## Updating macOS or WhateverXe

1. Read [WhateverXe](https://github.com/b00t0x/WhateverXe) / [WhateverXe-Graphics](https://github.com/b00t0x/WhateverXe-Graphics) release notes.  
2. If the servicing build changes (e.g. new **24H*** kernel), re-run **call-fix** validation or **`wex-callcheck`** on the new kernel.  
3. Retest **Metal**, **Control Center**, and **video** (App Store, Safari, TV windowed/full screen).  
4. Keep **`PostInstall/LaunchAgents`** until **`MTLDevice.architecture`** is fixed upstream.  
5. Do **not** enable sleep until Dell firmware exposes working S3 (unlikely on 1.55.0).

---

## Credits

- [Acidanthera](https://github.com/acidanthera) — OpenCore, Lilu, VirtualSMC, AppleALC, VoodooPS2  
- [OpenIntelWireless](https://github.com/OpenIntelWireless) — AirportItlwm, IntelBluetoothFirmware  
- [VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) — I2C touchpad/touchscreen  
- [b00t0x](https://github.com/b00t0x) — WhateverXe, WhateverXe-Graphics  
- [Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)  
- Community Tiger Lake laptop reports (including b00t0x Reddit threads)

Maintainer: **Satya** — Dell Latitude 5420 daily driver (2025–2026).

---

## Related links

- [WhateverXe issue #23 — Graphics 0.4.0 / 24H232 video crashes](https://github.com/b00t0x/WhateverXe/issues/23)  
- [OpenCore 1.0.8 release](https://github.com/acidanthera/OpenCorePkg/releases/tag/1.0.8)  
- [WhateverXe-Graphics 0.4.0 release](https://github.com/b00t0x/WhateverXe-Graphics/releases)

---

**License:** Config and documentation in this repo are provided as-is. Third-party kexts and OpenCore carry their own licenses. Do not commit Apple macOS installers, serial numbers, or decrypted drivers.
