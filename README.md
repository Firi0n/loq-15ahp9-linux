# Lenovo LOQ 15AHP9 on Linux

Notes, fixes and configs for running Linux on the **Lenovo LOQ 15AHP9** (machine type **83DX**: Ryzen 7 8845HS + RTX 4060 Laptop GPU).
Everything here was worked out and tested on my own machine. Other LOQ models may need changes.

| Topic | What's in it |
|---|---|
| [dGPU D3cold](docs/d3cold.md) | Fix for the NVIDIA dGPU waking up every ~20 s (`gpe10` wake loop), so it really powers off on battery |
| [Speaker EQ](docs/speaker-eq.md) | EasyEffects preset for the internal speakers, fitted from measurements of the Windows tuning |
| [BIOS updates](docs/bios-update.md) | Updating the BIOS from Linux with fwupd and Lenovo's official file, plus Insyde crisis recovery |
| [`tools/dgpu`](tools/dgpu) | Shows the dGPU power state **without waking it up** (`-w` needs `fuser`, from `psmisc`) |

## Test system

| | |
|---|---|
| Laptop | LOQ 15AHP9, MT 83DX, BIOS NZCN42WW / EC 1.42 |
| OS | CachyOS (Arch), kernel 7.3, niri (Wayland) |
| NVIDIA | open kernel modules 620.x |
| Audio | EasyEffects 8.3, lsp-plugins-lv2 |

## Disclaimer

Unofficial, not affiliated with Lenovo or NVIDIA. The BIOS section in particular involves flashing firmware: read it fully and use it at your own risk.

## Related

- [fwupd/firmware-lenovo#461](https://github.com/fwupd/firmware-lenovo/issues/461): request to publish LOQ BIOS updates on LVFS
- [NVIDIA/open-gpu-kernel-modules#905](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/905): the dGPU wake loop

Contributions and reports from other LOQ models are welcome: open an issue.
