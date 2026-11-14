# kernel-8056 — SC8810 8056 板级内核（GitHub Actions 云编译）

把酷派 8056（SC8810, 2.6.35.7 vlx guest）升 Android 4.0.4 的内核编译线。
上游 = fxsheep/sprd-kernel-kyletd（SC8810 3.0.8 BSP，kyletd-vlx 与 8056 同代 vlx guest）。

## 结构

- 仓库本体只有 workflow + 板级补丁；内核源码以 git submodule 引用
  `fxsheep/sprd-kernel-kyletd`（分支 sprdlinux3.0-kyletd）。
- CI：GitHub Actions `kernel.yml`，push 到 `sc8810-8056` 分支自动编译。
  工具链 = AOKP arm-eabi-4.4.3 prebuilt（与 8056 官方内核同代 gcc4.4.3，避免 ABI 坑），
  缓存 arm-eabi-4.4.3-tc-v1-ccache-v1。
- 产物：`zImage + System.map + .config`，artifact 名 `zImage-<sha>`，30 天保留。

## 板级补丁序列（commit = 可回滚实验点）

1. 基线：kyletd-vlx_defconfig 裸编译（验证 vlx 层在 8056 上出 kernel log）
2. MACH_8056：mach-sc8810 板级文件（GPIO 表/中断/串口——从 8056 2.6 zImage 反解）
3. LCD：8056 屏驱动（对比 2.6 zImage init 序列）
4. 传感器/触摸：adxl34x/lis3dh/ft5x0x/cy8c242
5. WiFi：BCMDHD + 8056 的 b2.bin/nvram（固件走 /system/etc/wifi，不进内核）

## boot.img 重打包

CI 只产 zImage；boot.img 重打包在本地用 port2.py 的 repack-boot 子命令
（8056 官方 boot.img 头字段：kernel 5663616B / ramdisk 268468224B 记录段、
gzip payload @3899788、Linux 2.6.35.7 gcc 4.4.3、构建路径 8056_MP）。

## 已知风险（按致命度）

1. vlx ABI：2.6.35(W12.12) guest ↔ 3.0.8(sprdroid4) guest 协议漂移 →
   首判据 = 自编译内核在 8056 出 kernel log。
2. 239MB RAM 跑 ICS 紧，zram 必开（kyletd defconfig 已含 CONFIG_ZRAM）。
3. 板级 GPIO 表反解质量决定外设成败。
