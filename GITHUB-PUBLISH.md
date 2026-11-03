# Publishing this repo on GitHub

Checklist before the first push:

1. **Copy EFI** from the working tree (maintainer: `~/Documents/Hackintosh/EFI/`) into **`EFI/`** in this repo. Eject USB sticks cleanly; do not commit `.DS_Store` or `._*`.
2. **Scrub secrets:** Open **`EFI/OC/config.plist`** → **`PlatformInfo`** → remove or genericize **`SystemSerialNumber`**, **`MLB`**, **`SystemUUID`**. Use [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) to generate your own if you fork.
3. **WhateverXe.kext:** Confirm binary SHA-256 is the **24H232 call-fix** build, not stock 0.6.1. Expected maintainer hash: **`e0b1ceea1d77c138834867e50177bc6eb297b0c3eda4d344cb47be07f764be06`**.
4. **Optional configs:** Include **`config-install.plist`** only if you want installer instructions; do not ship multiple active **`config.plist`** files without README notes.
5. **PostInstall:** Copy LaunchAgent plists from the Dell (`~/Library/LaunchAgents/local.wex.metal-env*.plist`) into **`PostInstall/LaunchAgents/`**.
6. **Do not commit:** macOS installers, Apple `.pkg` extracts, personal crash logs, or **`config.plist`** backups with your Apple ID tokens.
7. **`.gitignore`** suggestion:

```
.DS_Store
._*
*.log
SysReport/
**/config-before-*.plist
**/backup-*/
```

8. **Create repo** (GitHub UI or CLI):

```bash
cd /path/to/Dell-Latitude-5420-OpenCore
git init
git add README.md EFI Docs PostInstall GITHUB-PUBLISH.md .gitignore
git commit -m "Initial OpenCore EFI for Dell Latitude 5420 (Sequoia 24H232, Iris Xe)"
gh repo create Dell-Latitude-5420-OpenCore --public --source=. --remote=origin --push
```

9. **Repo description** (GitHub short text):

```
OpenCore 1.0.8 EFI for Dell Latitude 5420 (i5-1145G7, Iris Xe) — macOS Sequoia 15.8.2 with WhateverXe Metal 2. Sleep disabled; video caveats on 24H232.
```

10. **Topics:** `hackintosh` `opencore` `dell-latitude-5420` `tiger-lake` `iris-xe` `whateverxe` `macos-sequoia`
