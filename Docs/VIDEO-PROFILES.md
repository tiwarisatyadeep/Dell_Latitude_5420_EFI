# Video profiles on macOS 15.8.2 (24H232)

WhateverXe-Graphics **0.4.0** enables Metal 2 and loads **`AppleIntelTGLGraphicsVADriver.bundle`**. On build **24H232** (not **24H32**), the VA driver and GVA paths can crash WindowServer and background services. See [WhateverXe #23](https://github.com/b00t0x/WhateverXe/issues/23).

## Profile A — Stable (recommended for daily use)

| Item | Setting |
| --- | --- |
| Boot-arg | **`-wex-nogva`** (WhateverXe build that implements `tglDropGVA`) |
| VA bundle | **Parked** — move `AppleIntelTGLGraphicsVADriver.bundle` to `/Library/WhateverXe-parked/` |
| VideoToolbox HW decode | **False** (CPU decode) |
| Metal | **Metal 2** (unchanged) |
| Typical use | Music, App Store, Safari / YouTube (including full screen), **TV trailers in a window** |
| Avoid | **TV app full screen** (still crashes WindowServer in testing) |

Maintainer scripts (not in this EFI zip): **`wex7-nogva.sh`**, **`wex7-park.sh`** from the private **`WEX-070`** toolkit.

Verify:

```bash
sysctl kern.bootargs | grep wex-nogva
ioreg -lw0 -d1 | grep TGLNoGVA
ls /Library/GPUBundles/
ls /Library/WhateverXe-parked/
```

## Profile B — Stock (hardware decode, higher crash risk)

| Item | Setting |
| --- | --- |
| Boot-arg | **No** `-wex-nogva` |
| VA bundle | **`/Library/GPUBundles/AppleIntelTGLGraphicsVADriver.bundle`** |
| WhateverXe kext | **0.6.1 call-fix** only (not nogva-patched kext required for stock GVA keys) |
| VideoToolbox HW decode | **True** for H.264 / HEVC (typical) |
| Risk | VA **+0x1336f3** crashes, App Store GPU hang, TV / FairPlay paths |

Restore stock from Profile A:

```bash
# Example maintainer commands on the Dell
cd ~/Desktop/wex7
bash wex7-nogva.sh restore
bash wex7-park.sh off
sudo reboot
```

Return to Profile A:

```bash
cd ~/Desktop/wex7
bash wex7-nogva.sh build
bash wex7-nogva.sh install EFI
bash wex7-park.sh on
sudo reboot
```

Replace **`EFI`** with your OpenCore folder name if different.
