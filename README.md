# flame-fart12-build

Builds a `userdebug` AOSP 12 image for **Pixel 4 (flame)** with **fart12-lite** patches
integrated, targets `com.sacombankpay` for DexProtector dump extraction.

## What you get

GitHub Release artifact per run:

```
flame-fart-<sha>.zip
├── boot.img                  (AOSP build with FART)
├── system.img                (AOSP build with FART)
├── vbmeta.img                (AVB chain)
├── vendor-flame-*.img        (factory blob)
├── bootloader-flame-*.img    (factory blob)
├── radio-flame-*.img         (factory blob)
├── f1rt.config               (Sacombank config)
├── flash-all.sh              (one-shot flash wrapper)
└── build.log                 (full m output)
```

## Flash procedure

```bash
unzip flame-fart-<sha>.zip -d flame-fart
cd flame-fart
# device in fastboot mode
./flash-all.sh
```

After flash, push the FART config and launch the target app:

```bash
adb push f1rt.config /data/local/tmp/f1rt.config
adb shell am start -n com.sacombankpay/.MainVer2Activity
# wait 60s for FART trigger
sleep 60
adb pull /data/data/com.sacombankpay/zskkk/
```

Dumped files include `*_dexfile.dex`, `*_classlist.txt`, `*_deep_dexfile.dex`,
`*_ins_*.bin`, `*_dexfile_repair.dex`.

## Build pipeline

Three-job GitHub Actions workflow:

| Job | Purpose | Wall-clock |
|---|---|---|
| `sync` | `repo init` against `android-12.0.0_r32`, sync AOSP tree, cache `.repo` archive | 30-60 min |
| `build` | Apply FART patches, `lunch aosp_flame-userdebug`, `m system.img boot.img vbmeta.img` | 3-5 hr |
| `assemble` | Download flame factory blob, build `flash-all.sh`, upload to GitHub Release | 5-10 min |

## Trigger

`workflow_dispatch` only. Manual run from Actions tab. Inputs:

- `aosp_tag` — default `android-12.0.0_r32`

## Local submodule

Pinned to `Zskkk/fart12-lite` commit `7671fe3b95d28162e1b2024262ddf4fd8fe4077b`.

```bash
git clone --recurse-submodules https://github.com/ZuoqTr/flame-fart12-build.git
```

To bump the pin:

```bash
cd fart12-lite && git fetch && git checkout <new-sha> && cd ..
git add fart12-lite && git commit -m "bump fart12-lite pin"
```

## Files

- `.github/workflows/build.yml` — 3-job pipeline
- `.github/workflows/pr-lint.yml` — `git apply --check` on PRs
- `f1rt.config` — Sacombank dump config (`isDeep: true`)
- `flash-all.sh.template` — substituted at build time
- `fart12-lite/` — submodule (pinned upstream)

## License

Research / offensive-security use only. Not for production deployment.