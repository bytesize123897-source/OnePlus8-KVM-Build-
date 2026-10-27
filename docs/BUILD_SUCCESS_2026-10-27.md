# OnePlus 8 / IN2010 Android 13 KVM 编译成功基线

日期：2026-10-27

状态：**已完整编译成功（stock baseline + minimal KVM Image）**。当前只证明编译链成功，尚未把本次 KVM boot.img 标记为实机启动验证通过。

## GitHub Actions

- 仓库：`bytesize123897-source/OnePlus8-KVM-Build-`
- 成功 Run ID：`40475096930`
- 成功 workflow commit：`b2a78281e4826d1e386635fa375084ae2131ad70`
- SM8250 最小链接兼容补丁 commit：`afedd48030501571304080c727733a441c97ac53`
- stock 编译错误摘要：空文件（0 bytes）
- KVM 编译错误摘要：空文件（0 bytes）

## 官方源码固定版本

### Kernel

- Repository: `OnePlusOSS/android_kernel_oneplus_sm8250`
- Commit: `1d2678a3548f4825447f17b25a5cc1754c4cd1ae`
- Defconfig: `vendor/kona-perf_defconfig`

### Modules / Device Tree

- Repository: `OnePlusOSS/android_kernel_modules_and_devicetree_oneplus_sm8250`
- Commit: `01fb3921e0f57c338c95f9bcbe1a9938821c7e6d`

### 目标固件

- Device: OnePlus 8 / IN2010
- Firmware: `IN2010_13.1.0.190(CN01)`
- Android: 13 / ColorOS 13.1
- Kernel release: `4.19.157-perf+`

## 已验证构建方式

SM8250 这套 OnePlus/OPlus 源码不能只把 `oplus_native_features.mk` 放在源码根目录后直接普通 `make`。必须把里面的 OPlus feature 变量导出到递归 make 环境，否则会出现调用端被编译、provider 对象没有进入 `built-in.a` 的情况。

已验证关键变量包括：

```text
OPLUS_FEATURE_UFSPLUS=yes
OPLUS_FEATURE_UFS_SHOW_LATENCY=yes
OPLUS_FEATURE_PADL_STATISTICS=yes
OPLUS_FEATURE_PROCESS_RECLAIM=yes
OPLUS_FEATURE_MEMLEAK_DETECT=yes
```

已验证工具链/链接参数：

```text
CC="ccache clang-14"
HOSTCC="ccache clang-14"
HOSTCXX="ccache clang++-14"
LD=aarch64-linux-gnu-ld
AR=llvm-ar-14
NM=llvm-nm-14
OBJCOPY=aarch64-linux-gnu-objcopy
OBJDUMP=llvm-objdump-14
STRIP=llvm-strip-14
CLANG_TRIPLE=aarch64-linux-gnu-
CROSS_COMPILE=aarch64-linux-gnu-
CROSS_COMPILE_ARM32=arm-linux-gnueabi-
BRAND_SHOW_FLAG=oneplus
KCFLAGS="-gdwarf-4 -Wno-error=strict-prototypes -Wno-error=unused-but-set-variable"
```

## SM8250 必需的最小链接兼容补丁

文件：`patches/sm8250-link-compat.patch`

只处理官方源码公开版本在独立内核构建环境中的两个链接兼容点，不包含 KernelSU / ReSukiSU：

1. 将 `RTIC_BSS` 通过 `BSS_FIRST_SECTIONS` 放入正常 BSS 布局，解决 `.bss.rtic` / `selinux_state` 的 AArch64 relocation overflow / `dangerous relocation`。
2. 在 ARM64 linker script 中加入：

```text
PROVIDE(__crc_gsi_write_channel_scratch = 0);
```

解决 stock SM8250 未版本化 CRC 符号的链接问题。

## 成功产物校验

### Stock Image

```text
SHA256: 3d21a6fffd4aae9d71ed759ea250586ceab8e248c450e8acd83cf4fa484342c4
```

### Minimal KVM Image

```text
SHA256: d7ffc23bd93f9dd981dfa359f915a04a551de925db6c3cd83e490f3416caded5
Kernel release: 4.19.157-perf+
```

## Stock -> KVM 实际 Kconfig 差异

成功 run 中 `diffconfig` 的完整结果：

```text
-ARM64_SVE y
 VIRTUALIZATION n -> y
+ARM64_ERRATUM_834220 y
+HAVE_KVM_ARCH_TLB_FLUSH_ALL y
+HAVE_KVM_CPU_RELAX_INTERCEPT y
+HAVE_KVM_EVENTFD y
+HAVE_KVM_IRQCHIP y
+HAVE_KVM_IRQFD y
+HAVE_KVM_IRQ_BYPASS y
+HAVE_KVM_IRQ_ROUTING y
+HAVE_KVM_MSI y
+HAVE_KVM_VCPU_RUN_PID_CHANGE y
+IRQ_BYPASS_MANAGER y
+KVM y
+KVM_ARM_HOST y
+KVM_ARM_PMU y
+KVM_GENERIC_DIRTYLOG_READ_PROTECT y
+KVM_INDIRECT_VECTORS y
+KVM_MMIO y
+KVM_VFIO y
+MMU_NOTIFIER y
+PREEMPT_NOTIFIERS y
+VHOST_CROSS_ENDIAN_LEGACY n
+VHOST_NET n
```

注意：虽然我们主动只启用了 `VIRTUALIZATION` 和 `KVM`，`olddefconfig` 会根据依赖自动调整上面的选项，其中包括关闭 `ARM64_SVE`。以后修改 KVM 配置时必须重新检查 diff，不能只看手工设置的两行。

## 以后编译的默认策略

这次 stock baseline 已经验证成功，因此普通 KVM 配置迭代不需要每次重复完整的 `stock Image -> KVM Image` 双编译。

默认快速路径应该是：

1. 固定上述官方源码 SHA。
2. 固定 `sm8250-link-compat.patch`。
3. 固定经过验证的 stock `.config` 或从同一 defconfig 生成配置。
4. 导出完整 OPlus feature 环境。
5. 应用 KVM 配置后只编译一次 KVM Image。
6. 继续使用 ccache。

以下情况必须重新跑完整 stock baseline：

- 官方 kernel SHA 变化；
- modules/device-tree SHA 变化；
- OPlus feature 导出逻辑变化；
- linker / compiler 主版本变化；
- SM8250 兼容补丁变化；
- defconfig/stock config 基线变化；
- 出现无法解释的 vendor undefined reference / relocation 问题。

## Boot 打包原则

不要重新构造 ramdisk、cmdline、DTB 或 boot header。应使用 **IN2010_13.1.0.190(CN01) 对应的已知 stock boot 布局作为基底，只替换 kernel payload**，并对重打包前后的所有非 kernel payload 做逐项一致性校验。

这样可以最大限度避免再次出现“内核能编译但 boot 卡在开机动画”的问题。
