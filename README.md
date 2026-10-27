# OnePlus 8 KVM Minimal Kernel Build

Clean GitHub Actions build for the OnePlus 8 (IN2010 / SM8250) using official OnePlus kernel sources.

## Pinned official sources

- Kernel: `OnePlusOSS/android_kernel_oneplus_sm8250`
  - branch: `oneplus/sm8250_t_13.1_op8`
  - commit: `1d2678a3548f4825447f17b25a5cc1754c4cd1ae`
- Modules / device tree: `OnePlusOSS/android_kernel_modules_and_devicetree_oneplus_sm8250`
  - branch: `oneplus/sm8250_t_13.1_op8`
  - commit: `01fb3921e0f57c338c95f9bcbe1a9938821c7e6d`

## KVM delta

The public build starts from the official `vendor/kona-perf_defconfig` and adds only:

```text
CONFIG_VIRTUALIZATION=y
CONFIG_KVM=y
```

`make olddefconfig` is then used to resolve Kconfig dependencies. The generated config diff is uploaded with the build artifact for review.

## Privacy / scope

This repository intentionally does **not** contain any device-extracted stock config, `boot.img`, APatch files, private logs, keys, or other device-specific private artifacts.

The workflow builds a kernel `Image` only. It does not create or flash a boot image.

## Device test safety

When repacking later, preserve the stock boot image ramdisk/header/cmdline and test with `fastboot boot` first. Do not permanently flash an unverified build.

Target firmware used for this build track: OnePlus 8 IN2010, ColorOS 13.1 / Android 13, `IN2010_13.1.0.190(CN01)`.
