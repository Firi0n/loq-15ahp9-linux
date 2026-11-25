# Speaker EQ (EasyEffects)

Out of the box on Linux the LOQ 15AHP9 speakers sound thin and quiet. On Windows the sound is shaped twice: by Lenovo's own speaker tuning in the Windows audio stack, then by Nahimic.
None of that exists on Linux, so the signal reaches the speakers flat.

[`easyeffects/output/LOQ 15AHP9 Speakers.json`](../easyeffects/output/LOQ%2015AHP9%20Speakers.json) is an EasyEffects 8 preset that recreates that tuning.

## How it was made

1. **Measurement on Windows** (before removing it): white noise was played and recorded with a loopback capture, with Nahimic turned off, to get the curve applied by Lenovo's tuning alone.
   It's roughly a high-pass below ~80 Hz, +4-5 dB at 125-400 Hz, a dip around 1 kHz and up to +8.5 dB of treble, with about -9 dB of headroom and a limiter at 0 dBFS.
2. **Tuning by ear** on top of that curve.
3. **Fitting** the result to an EasyEffects equalizer: a 60 Hz high-pass plus 27 bell filters (40 Hz-16 kHz, Q 2.5), with less than 1 dB of error between 40 Hz and 10 kHz.
4. **Loudness**: a 4-band compressor and a fast limiter, because the speakers are very quiet without the Windows processing.

## The chain

| # | Plugin | Settings |
|---|---|---|
| 1 | Equalizer | input -5 dB, HPF 60 Hz + 27 bells (the measured curve) |
| 2 | Multiband compressor | splits at 200 Hz / 2 kHz / 6 kHz, thresholds -30 dB, ratios 4 / 3 / 3 / 2.5, makeup +8 / +12 / +12 / +10 dB |
| 3 | Limiter | input +9 dB, threshold -1 dB, attack 2 ms, release 30 ms, ALR off |

The preset needs `lsp-plugins-lv2` (equalizer, compressor and limiter all come from LSP).

## Install

```sh
mkdir -p ~/.local/share/easyeffects/output
cp "easyeffects/output/LOQ 15AHP9 Speakers.json" ~/.local/share/easyeffects/output/
```

Tested with EasyEffects 8.3 (native package). The Flatpak stores presets under `~/.var/app/com.github.wwmm.easyeffects/` instead.
Then load it in EasyEffects. To apply it automatically only on the speakers, add a rule in EasyEffects' preset autoload settings for the internal audio output with the speaker route, using this preset.
Speakers and wired headphones are the same sink and only differ by route, so the headphones can get a different (or empty) preset as the fallback.

## Notes

- **Keep the system volume at or below 100%.** The ALC257 hardware controls are already at 0 dB, so anything above 100% is just digital clipping. The loudness comes from the preset instead.
- **Laptop position matters.** The speakers fire downwards: the preset was tuned with the laptop on my lap. On a desk you get noticeably more bass and low-mids.
- **Corrective EQ goes after the compressor.** Cuts placed before the multiband compressor get mostly undone by it. If you want to change the tone, add a second equalizer between the compressor and the limiter.
- **Not perfect yet.** Compared with Windows + Nahimic, voices still lack a bit of body. A broad +2-3 dB around 800 Hz-1 kHz after the compressor is the next thing to try.
- Other LOQ models probably have different speakers, so there this is just a starting point.
