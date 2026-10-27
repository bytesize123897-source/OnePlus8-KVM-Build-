# OnePlus 8 IN2010 KVM boot.img 打包成功记录

日期：2026-10-27

状态：**boot.img 已成功打包并完成结构一致性验证；尚待实机启动验证。**

## 来源

- 成功 KVM kernel Run ID：`40475096930`
- 成功 kernel artifact ID：`12723510313`
- KVM kernel release：`4.19.157-perf+`
- KVM Image SHA256：`d7ffc23bd93f9dd981dfa359f915a04a551de925db6c3cd83e490f3416caded5`

## Boot 打包 Run

- Package Run ID：`40482672936`
- Package artifact ID：`12723995894`
- Workflow：`.github/workflows/package-boot.yml`
- 修复后 workflow commit：`a67c5254cae5cc872267c82ab5b4202353d5ce4a`

## 目标固件

- Device：OnePlus 8 / IN2010
- Firmware：`IN2010_13.1.0.190(CN01)`
- Android / ColorOS：Android 13 / ColorOS 13.1
- Boot header version：2
- Boot image size：`100663296` bytes
- OS patch level（boot header）：`2024-04`

## 打包基底校验

使用与 IN2010_13.1.0.190(CN01) 对应、已验证只替换 kernel 的 boot 容器作为布局基底。

- Reference package SHA256：`76ff74b28a1eb5954d2c275a772642bd7e37891b765a6ade1d595e34f04be947`
- Reference carrier boot SHA256：`ce16155f0f1dfaa90da4c2508c8d219a8fdeeec35101669ea50d3979001a04d4`
- Reference boot size：`100663296` bytes

打包工具固定为：

- Magisk：`v30.7`
- Magisk APK SHA256：`e0d32d2123532860f97123d927b1bb86c4e08e6fd8a48bfc6b5bee0afae9ebd5`
- 使用 APK 内 `lib/x86_64/libmagiskboot.so`

## 最终 boot.img

文件：

```text
boot-IN2010-13.1.0.190-KVM.img
```

SHA256：

```text
02af472526f8398b28bb9702b0aacda6a263cabb6d0bd0a5d3364de5073f9624
```

大小：

```text
100663296 bytes
```

## 验证结果

最终 boot.img 被重新解包并进行了以下验证，全部通过：

1. 解包后的 `kernel` SHA256 与成功 KVM `Image` 完全一致。
2. 非 kernel 的解包文件列表与参考 boot 一致。
3. 所有非 kernel payload（包括 ramdisk、DTB）逐文件 SHA256 与参考 boot 一致。
4. 最终 boot.img 保持原 boot image 的 `100663296` 字节大小。
5. boot header v2、cmdline、ramdisk/DTB 布局来自匹配的 IN2010 固件容器，而不是重新手工构造。

## 当前验证等级

- Kernel compile：PASS
- Stock baseline compile：PASS
- KVM kernel compile：PASS
- Boot repack：PASS
- Non-kernel payload equivalence：PASS
- On-device boot：**PENDING**

在实机验证完成以前，不要把此记录改成 device verified。

优先使用临时启动方式验证（设备/bootloader 支持时使用 `fastboot boot`），确认能进入系统、ADB 正常、关键硬件正常后，再考虑写入 boot 分区。
