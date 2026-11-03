# Power management (Dell Latitude 5420, BIOS 1.55.0)

Sleep and hibernate were tested extensively (forced S3 DSDT patch, HibernationFixup, auto-hibernate). None are safe for daily use on this firmware.

Apply after install:

```bash
sudo pmset -a disablesleep 1 hibernatemode 0
pmset -g | grep -E 'SleepDisabled|hibernatemode'
```

Expected: **`SleepDisabled 1`**, **`hibernatemode 0`**. Apple menu **Sleep** stays greyed out. Use **Shut Down** instead of Sleep.

Display can still dim (`displaysleep`); that is not the same as S3 sleep.
