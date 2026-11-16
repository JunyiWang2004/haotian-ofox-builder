# 用 GitHub Actions 编译 haotian（小米 15 Pro）的 OrangeFox

## 这是什么

一个自包含的 GitHub Actions 流水线，用**橙狐官方 `fox_16.0` 完整清单**编译 recovery，
不占你本机的磁盘、内存和网络。

仓库必须是 **public**：public runner 是 4 核 16 GB 且不扣分钟数；private 只有 2 核 8 GB 还扣额度。

## 两个仓库

| 仓库 | 内容 |
|---|---|
| `twrp_device_xiaomi_sm8750_thales`（fork，公开） | 设备树，分支 `fox_16.0` |
| 本仓库（新建，公开） | 只有 `.github/workflows/build.yml` |

设备树分支 `fox_16.0` 基于 `YuKongA/twrp_device_xiaomi_sm8750_thales` 的 `twrp-16.0`，
只加了三处橙狐配置：

- `vendorsetup.sh`（新增）—— 橙狐的 `OF_*` / `FOX_*` 编译变量
- `BoardConfig.mk` —— `TARGET_OTA_ASSERT_DEVICE`，让卡刷包认 `haotian` / `sm8750_thales` / `dada` / `xuanyuan`
- `twrp_sm8750_thales.mk` —— `TW_Y_OFFSET` / `TW_H_OFFSET`，状态栏偏移

TWRP 原有的解密、触控、振动配置**一行没动**。

## 流水线做了什么

1. `slimhub` 清理磁盘，加 16 GB swap
2. `repo init --depth=1 -u https://gitlab.com/OrangeFox/Manifest.git -b fox_16.0`
3. `repo sync -c -j$(nproc) --no-tags --no-clone-bundle --force-sync`，同步后打印 `df -h`
4. `rm -rf .repo` 回收空间
5. clone 设备树 `fox_16.0` 到 `device/xiaomi/sm8750_thales`
6. `FOX_BUILD_DEVICE=sm8750_thales` → `lunch twrp_sm8750_thales-bp2a-eng` → `mka adbd recoveryimage`
7. 校验产物头部：`kernel_size == 0` 且 ≤ 104,857,600 字节，不满足直接失败
8. 上传 `OrangeFox*.img` / `OrangeFox*.zip` / `recovery.img` / `build.log`

## 产物怎么用

- **刷之前必须备份。** 当前槽位的 recovery 分区先 `dd` 出来。
- 只刷 recovery，不要碰 `boot` / `vendor_boot` / `vbmeta`。
- 刷之前确认槽位：`fastboot getvar current-slot`。
- 恢复：关机后音量下 + 电源进 fastboot，`fastboot flash recovery_a <备份>`。

## 参考来源

- OrangeFox 官方清单：https://gitlab.com/OrangeFox/Manifest （分支 `fox_16.0`）
- 官方同步脚本：https://gitlab.com/OrangeFox/sync
- 编译变量文档：https://gitlab.com/OrangeFox/infrastructure/doc/-/blob/main/dev/build_vars.md
- 设备树上游：https://github.com/YuKongA/twrp_device_xiaomi_sm8750_thales
- 本机型已知可用的橙狐设备树（fox_14.1 基座）：https://github.com/adontoo/device_xiaomi_sm8750_OFRP
- 同平台参考：https://github.com/daixu0502/ofrp_device_xiaomi_sm8750_bixi
- 官方参考设备：https://gitlab.com/OrangeFox/device/mondrian （分支 `fox_16.0`）
