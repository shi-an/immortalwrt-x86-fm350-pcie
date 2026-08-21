# ImmortalWrt x86 FM350-GL PCIe

面向 x86_64 主路由和 MediaTek T700 / FM350-GL PCIe 模组的 ImmortalWrt 自动构建仓库。

[![Build ImmortalWrt x86 FM350-GL PCIe](https://github.com/shi-an/immortalwrt-x86-64-fm350-pcie/actions/workflows/build.yml/badge.svg)](https://github.com/shi-an/immortalwrt-x86-64-fm350-pcie/actions/workflows/build.yml)

## 固定上游

- ImmortalWrt 分支：`openwrt-24.10`
- 固定提交：`fa66185054014e529e412477777fe73ab69a7c05`
- 内核：Linux `6.6.151`
- 目标：`x86/64 generic`
- 默认 LAN 地址：`192.168.1.1`

工作流会在开始时校验上游提交和内核版本。上游发生变化时构建将主动失败，更新时应同时审查内核补丁上下文，而不是静默跟随分支。

## 默认功能

- Bandix
- Netdata
- EasyTier
- QModem Next
- FM350-GL MBIM Toolkit：`kmod-mtk-t7xx`、`umbim`、`mbimcli`
- NetSpeedTest
- UPnP
- EtherWake
- Daed

Lucky、AT WebServer、DockerMan 和 OpenClash 默认关闭，仍可在手动触发工作流时按需启用。

## T7xx 内核补丁

- `681-net-gro-fix-double-aggregation-flush-marked-skbs.patch`：防止 UDP fraglist GRO 的 flush 标记被重复聚合。
- `682-net-wwan-t7xx-keep-tx-ring-moving-on-invalid-skb.patch`：拒绝不在已提交描述符区间内的硬件 TX 读指针，防止旧 completion 反复释放已经清空的 skb bookkeeping。
- `683-net-gro-check-linear-data-before-fraglist-pull.patch`：在 fraglist GRO 调用 `skb_pull()` 前验证线性数据，不满足时丢弃该次聚合，避免 malformed skb 进入 UDP GSO 分段路径。

`682` 是异常 completion 的遏制与诊断措施，不代表已经修复 FM350-GL、PCIe 链路或模组固件导致异常读指针的根因。

## MBIM 使用边界

固件同时提供 QModem Next 与原生 MBIM 命令行工具。不要让 QModem 与其他 MBIM netifd 配置同时管理同一个模块控制口，否则状态查询可能互相抢占。

## 崩溃日志

镜像固定包含 `kmod-netconsole`。模块会自动加载，但接收端的 LAN IP 和 MAC 必须在复现前通过 `rmmod netconsole` 后的 `modprobe netconsole netconsole=...` 设置，不能使用固定的通用目标地址。

## 构建与下载

在 Actions 中运行 `Build ImmortalWrt x86 FM350-GL PCIe`。Release 提供完整 x86 `combined` 镜像、manifest 和校验文件；不会单独分发内核模块，因为 `mtk_t7xx.ko` 必须与固件的精确内核 ABI 匹配。
