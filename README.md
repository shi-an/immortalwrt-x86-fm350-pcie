# ImmortalWrt x86 FM350-GL PCIe

面向 x86_64 主路由和 MediaTek T700 / FM350-GL PCIe 模组的 ImmortalWrt 自动构建仓库。

[![Build ImmortalWrt x86 FM350-GL PCIe](https://github.com/shi-an/immortalwrt-x86-64-fm350-pcie/actions/workflows/build.yml/badge.svg)](https://github.com/shi-an/immortalwrt-x86-64-fm350-pcie/actions/workflows/build.yml)

## 上游来源

- ImmortalWrt 分支：`openwrt-24.10`
- 源码：每次构建拉取该分支最新提交，并在产物中记录实际 revision
- 内核：Linux `6.6.x`
- 目标：`x86/64 generic`
- 默认 LAN 地址：`192.168.1.1`

工作流会在构建开始时解析并拉取上游分支最新 HEAD，仍会校验 Linux 6.6 补丁上下文和修复是否进入实际内核源码。上游补丁栈发生不兼容时，构建应失败并提示重新审查，而不是静默使用错误的补丁。

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
- SQM / CAKE，以及 CAKE-autorate（预装，配置后启用）

Lucky、AT WebServer、DockerMan 和 OpenClash 默认关闭，仍可在手动触发工作流时按需启用。

## T7xx 内核补丁

- `681-net-gro-fix-double-aggregation-flush-marked-skbs.patch`：防止 UDP fraglist GRO 的 flush 标记被重复聚合。新版 Linux 6.6 已包含该修复，680 不再搬移 GRO 函数时 CI 跳过此补丁，仍校验实际内核源码中的防护。
- `682-net-wwan-t7xx-keep-tx-ring-moving-on-invalid-skb.patch`：拒绝不在已提交描述符区间内的硬件 TX 读指针，防止旧 completion 反复释放已经清空的 skb bookkeeping。
- `683-net-gro-check-linear-data-before-fraglist-pull.patch`：在 fraglist GRO 调用 `skb_pull()` 前验证线性数据，不满足时丢弃该次聚合，避免 malformed skb 进入 UDP GSO 分段路径。
- `684-net-wwan-t7xx-fix-rx-buf-alloc-off-by-one.patch`：记录 T7xx RX 缓冲分配失败路径的 off-by-one 修复；当前 Linux 6.6 上游源码已包含该修复，CI 不重复安装。
- `686-net-udp-gro-validate-socket-length.patch`：补齐 Linux 6.6 UDP socket GRO 入口的长度校验，拒绝 malformed skb 进入 fraglist GSO 分段路径。
- `688-net-gso-reject-missing-fraglist-entry.patch`：在 `skb_segment()` 处理完最后一个 fraglist 项后验证链表仍存在，拒绝畸形长度元数据触发的第二次空指针解引用；不影响合法 fraglist GRO。
- `689-net-gso-reject-empty-fraglist-segment-list.patch`：在 `skb_segment_list()` 入口拒绝空 `frag_list`，避免 UDP fraglist 完成路径把单个 skb 当成分段链表访问；不影响合法 fraglist GRO。
- `690-net-wwan-t7xx-ratelimit-tx-full-log.patch`：限制 TX 队列满日志频率，保留队列停止/唤醒逻辑，不改变 GRO 或 TX 吞吐路径。

`682` 是异常 completion 的遏制与诊断措施，不代表已经修复 FM350-GL、PCIe 链路或模组固件导致异常读指针的根因。

构建保持 FM350-GL fraglist GRO 开启，依靠 681、683、686、688 和 689 校验 GRO 输入与 GSO 分段条件。

## MBIM 使用边界

固件同时提供 QModem Next 与原生 MBIM 命令行工具。不要让 QModem 与其他 MBIM netifd 配置同时管理同一个模块控制口，否则状态查询可能互相抢占。

## 崩溃日志

镜像固定包含 `kmod-netconsole`。模块会自动加载，但接收端的 LAN IP 和 MAC 必须在复现前通过 `rmmod netconsole` 后的 `modprobe netconsole netconsole=...` 设置，不能使用固定的通用目标地址。

## CAKE-autorate

固件预装 [CAKE-autorate](https://github.com/lynxthecat/cake-autorate/tree/v3.2.2) 稳定版 `3.2.2` 及 `bash`、`fping` 依赖，根据流量和延迟动态调整 CAKE 带宽，适用于 FM350-GL 等带宽波动的蜂窝线路。源码版本和 SHA256 固定在本地软件包中。

默认 `/etc/config/cake-autorate` 的 `main.enabled=0`，配置好线路后再开启：

1. 在 LuCI 的 SQM 页面为实际 WAN 接口配置并启用 CAKE；用 `tc qdisc show` 确认上行和 IFB 下行接口已有 `cake` 队列。
2. 编辑 `/etc/cake-autorate/config.primary.sh`，设置 `ul_if`、`dl_if` 以及各自的最小、基础、最大带宽（`min_*`、`base_*`、`max_*_shaper_rate_kbps`，单位 Kbit/s）。文件中的接口和速率是上游示例，需要按实测修改；`reflectors` 应选用该线路可稳定响应的探测地址，其他参数参考 `/usr/lib/cake-autorate/defaults.sh`。
3. 执行以下命令启用服务和开机启动：

```sh
uci set cake-autorate.main.enabled='1'
uci commit cake-autorate
/etc/init.d/cake-autorate enable
/etc/init.d/cake-autorate restart
```

修改参数后重启服务生效；运行日志位于 `/var/log/cake-autorate.primary.log`。`/etc/cake-autorate/` 和 UCI 开关配置会随系统配置备份保留。

## 构建与下载

在 Actions 中运行 `Build ImmortalWrt x86 FM350-GL PCIe`。Release 提供完整 x86 `combined` 镜像、manifest 和校验文件；不会单独分发内核模块，因为 `mtk_t7xx.ko` 必须与固件的精确内核 ABI 匹配。
