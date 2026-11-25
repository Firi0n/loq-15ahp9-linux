# dGPU D3cold: fixing the `gpe10` wake loop

## Symptom

On battery the NVIDIA dGPU should sit in **D3cold** (fully powered off) whenever nothing uses it.
On the LOQ 15AHP9 it does reach D3cold after boot, but **once it has been woken a single time** (a game, `nvidia-smi`, `glxinfo`, an external monitor...) it never settles again:

- it loops forever: about 20 s in D0, about 1 s in D3cold, repeat;
- no process has `/dev/nvidia*` open, so it is not a userspace problem;
- the ACPI GPE counter `/sys/firmware/acpi/interrupts/gpe10` goes up once per cycle;
- stopping `nvidia-powerd` does not change anything.

The result is several watts wasted on battery for no reason.

## Cause

Right after the GPU enters D3cold, the firmware fires a spurious GPE (`gpe10`) on the dGPU's PCIe port, which wakes the GPU up again.
This is tracked in [NVIDIA/open-gpu-kernel-modules#905](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/905), where the LOQ 15AHP9 (`gpe10`, device `10de:28e0`) is listed.
A kernel quirk for the broken GPE is in the works upstream, but at the time of writing it only matches some Legion models, not the LOQ.

## Fix

Set the NVIDIA module option:

```sh
echo 'options nvidia NVreg_DynamicPowerManagementVideoMemoryThreshold=0' | sudo tee /etc/modprobe.d/nvidia-power.conf
```

If the nvidia modules are in your initramfs (CachyOS's `chwd` puts them there), regenerate it so the option is picked up early.
On CachyOS with Limine that's `sudo limine-mkinitcpio`; elsewhere `sudo mkinitcpio -P` or your distro's equivalent. Then reboot.

That's the only setting needed. On Ampere+ notebooks with recent drivers the rest is already the default:

| Setting you may find in old guides | Status |
|---|---|
| `NVreg_DynamicPowerManagement=0x02` | not needed, default `0x03` already enables fine-grained RTD3 on Ampere+ notebooks |
| `NVreg_PreserveVideoMemoryAllocations=1` | not needed, handled automatically by the open modules (default `2`) |
| `nvidia-drm modeset=1 fbdev=1` | already the default |
| udev rules for runtime PM (`80-nvidia-pm.rules`) | not needed, RTD3 is on by default (CachyOS also ships its own rule) |

Keep `nvidia-powerd` **enabled**: it does not cause the loop, and without it the GPU is capped at 55 W on AC (no Dynamic Boost).

## Verify

After a reboot, wake the GPU once (e.g. run `nvidia-smi`), wait about 10-20 s, then check:

```sh
dev=/sys/bus/pci/devices/0000:01:00.0       # the dGPU, check with: ls /sys/bus/pci/drivers/nvidia
cat $dev/power/runtime_status                # expected: suspended
cat $dev/power_state                         # expected: D3cold
cat /sys/firmware/acpi/interrupts/gpe10      # first number must stop increasing
grep VideoMemoryThreshold /proc/driver/nvidia/params   # expected: ...VideoMemoryThreshold: 0
```

Careful: `nvidia-smi`, `lspci`, `nvtop`, `glxinfo` and `vulkaninfo` **wake the GPU up** themselves, so don't use them to check whether it's asleep.
[`tools/dgpu`](../tools/dgpu) only reads safe sysfs files: `dgpu` prints the current state, `dgpu -w` logs every wake-up and which process caused it.

## Not the cause (things I tried)

- **The compositor holding the GPU open.** niri opens every DRM device at startup, including the NVIDIA card. With the fix above the GPU still reaches D3cold about 10 s after going idle even with niri holding it, so `ignore-drm-device` is not needed (and dropping it keeps the external HDMI/DP outputs, which are wired to the dGPU, working).
- **NVPCF waking the GPU at every 1% of battery** (reported in NVIDIA's repo for other laptops): not happening here on 620.x, checked over several percent of discharge with zero active time.
