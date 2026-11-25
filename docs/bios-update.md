# BIOS updates from Linux

> **Warning.** This flashes firmware with a method Lenovo does not support. It worked on my machine, but I can't guarantee it on yours.
> Plug in the charger, read everything first, and don't do it on a machine you can't afford to lose.

## Background

LOQ BIOS updates are **not** on LVFS, so `fwupdmgr update` will never offer them, even though fwupd lists the *System Firmware* device as updatable.
That flag only means the firmware accepts UEFI capsules ([fwupd/firmware-lenovo#461](https://github.com/fwupd/firmware-lenovo/issues/461)).
Lenovo only ships a Windows `.exe`, but the BIOS image inside it is a signed capsule that fwupd can install by hand.

## Update

Tested NZCN28WW → **NZCN42WW** (EC 1.28 → 1.42) on 2026-11-15 with fwupd 2.1.9.

1. **Download** the BIOS `.exe` from Lenovo's support site for your model. For the 15AHP9 the files are `https://download.lenovo.com/consumer/mobiles/nzcnNNww.exe`, and the matching `nzcnNNww.txt` is the changelog. Read it: some versions can't be downgraded.
2. **Extract** it (it's a self-extracting archive):
   ```sh
   7z x nzcn42ww.exe -onzcn42ww
   ```
   The BIOS image is the large `.ROM` file, `JLG50042.ROM` for NZCN42WW.
   It's a Lenovo-signed PE file (~34 MB) that wraps the full 32 MB SPI image, EC firmware included.
   The bundled `platform.ini` has `[UpdateEC] Flag=0` and `[SecureUpdate] viaESP=3`: the BIOS flashes itself from the ESP, nothing Windows-specific is needed.
3. **Find the System Firmware device ID:**
   ```sh
   fwupdmgr get-devices
   ```
   Look for *System Firmware* → *Device ID* (on my 83DX: `220985af3125caf84a6bc579fdb24afa378c3e53`, check yours).
4. **Note your BIOS settings** (Secure Boot, boot order, etc.): they get **reset to factory defaults**.
5. **Install** with the charger plugged in:
   ```sh
   sudo fwupdtool install-blob JLG50042.ROM <device ID>
   ```
   Reboot. The firmware flashes itself during boot: you'll see the Lenovo logo with a progress bar. Don't power off until it's done.
6. **After the update**: Secure Boot was re-enabled on my machine, so disable it again in the BIOS (F2) if your setup needs that, before Linux tries to boot.

Check the result:

```sh
cat /sys/class/dmi/id/bios_version /sys/class/dmi/id/ec_firmware_release
fwupdmgr get-devices    # System Firmware → Update State: Success
```

Checksums of the files I used:

| File | SHA256 |
|---|---|
| `nzcn42ww.exe` | `003c9a9235e921f374fcb1d54aaa442f2a07b841627650af763f71497511e9c7` |
| `JLG50042.ROM` | `57bb5042c6f083f64b4b57a4aa91e097e86011a1d0231b15eb7254581c0d7e40` |

## Crisis recovery (emergency only)

Insyde BIOSes have a recovery mode that reflashes the BIOS from a USB stick when the laptop no longer boots.
**I have not tested this on the 15AHP9**: the procedure was shared (with permission) by [@c8dhjp4tyv-bit](https://github.com/c8dhjp4tyv-bit), who used it to recover their LOQ 15AHP10.
It is an emergency procedure, not a safety net for experiments. A bad flash can also damage the EC or the recovery code itself, and then only an external SPI programmer can help.

1. Format a USB stick as **FAT32**.
2. Copy the `.ROM` extracted from Lenovo's official `.exe` (unmodified) to the root of the stick and **rename it to your machine type + `.bin`**, e.g. `83DX.bin` for the 15AHP9 or `83JG.bin` for the 15AHP10. The machine type is in `/sys/class/dmi/id/product_name`.
3. Make sure the battery is well charged first: the flash runs on battery. Laptop fully off, **charger unplugged**. Hold the power button for 30 s, then release it.
4. Insert the stick. Hold **Fn + R** for 5 s, and while still holding it press the power button. Keep Fn + R held for about 30 s, then release.
5. It starts on its own: **there is no confirmation prompt**. The laptop beeps continuously, possibly for several minutes, then restarts and shows the Lenovo logo while flashing. Don't interrupt it and don't remove the stick or the power.

Keep the stick unplugged when you don't mean to use it, since Fn + R at power-on is the only "confirmation".
