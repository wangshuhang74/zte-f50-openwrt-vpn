# ZTE F50 原生 OpenWrt / iStoreOS：5G 移动 VPN 固件

面向 **中兴 ZTE F50 5G 随身 WiFi / 移动路由器** 的原生 OpenWrt / iStoreOS 移植工程。项目重点是 Linux 系统、5G 移动网络管理、USB 恢复保护、A/B rootfs，以及可选的 VPN、代理和旁路由能力。

> 当前状态：这是一个面向开发者和高级用户的 bring-up 工程。真实刷写前请先完成只读备份、硬件确认和恢复链路验证。不要在不了解分区布局时直接执行擦除或刷写命令。

## 搜索关键词

`VPN` `移动VPN` `5G VPN` `随身WiFi VPN` `移动路由器 VPN` `中兴F50` `ZTE F50` `ZTE F50 5G` `F50 5G随身WiFi` `F50 OpenWrt` `F50 iStoreOS` `中兴随身WiFi` `中兴5G CPE` `OpenWrt 5G` `OpenWrt 移动网络` `OpenWrt 蜂窝网络` `Unisoc` `紫光展锐` `UMS9620` `T770` `T820` `T760` `T750` `唐古拉` `aarch64` `SA` `NSA` `4G/5G` `netifd` `wwan` `sing-box` `OpenClash` `PassWall` `旁路由` `WireGuard` `OpenVPN` `Tailscale` `ZeroTier` `Fastboot恢复` `移动宽带`

## 项目内容

- `target/linux/unisoc/`：Unisoc UMS9620 target 与 Linux 6.6 配置。
- `package/f50/f50-modem/`：F50 原生 modem 服务，通过 UCI、ubus 和 netifd 管理移动网络。
- `package/f50/luci-app-f50-modem/`：LuCI“移动网络 / 5G”页面。
- `package/f50/f50-ab-rootfs/`：F50DATA A/B rootfs 管理与启动保护。
- `package/f50/f50-recovery/`：USB 恢复和受保护分区守卫。
- `scripts/f50/`：备份、构建、验证、Fastboot 只读检查和 VPN 工具链脚本。
- `docs/f50/`：构建、恢复、硬件采集、modem 契约和测试矩阵。

## 安装前须知

1. 本项目不是运营商官方固件，也不保证所有地区、SIM 卡、频段、SA/NSA 组合都可用。
2. 刷写前必须保存原厂分区和校准数据；先阅读 [硬件采集](docs/f50/HARDWARE_ACQUISITION.md) 与 [USB 恢复](docs/f50/USB_RECOVERY.md)。
3. 不要把 IMEI、序列号、校准数据、运营商配置或个人日志上传到公开仓库。
4. 本仓库**不包含、不索取、不公开任何激活码、授权码、订阅密钥、API Token、密码或私有证书**。需要授权的服务请在设备本地填写。
5. 如果设备无法稳定进入恢复模式，请停止操作，保留原厂系统并先提交只读日志。

## 普通用户：安装流程

### 1. 准备主机

推荐 Linux x86_64 构建；macOS 用户建议使用 Docker、Linux 虚拟机或远程 Linux 主机。

```bash
git clone <你的 GitHub 或 Gitee 仓库地址>
cd F50
git submodule update --init --recursive
```

Debian/Ubuntu 依赖：

```bash
sudo apt update
sudo apt install -y build-essential clang flex bison g++ gawk gcc-multilib g++-multilib gettext git libncurses5-dev libssl-dev python3 python3-distutils rsync unzip zlib1g-dev file curl jq
```

### 2. 只读采集和备份

```bash
mkdir -p "$HOME/f50-backups/F50-$(date +%Y%m%d)"
python3 scripts/f50/backup_device.py --out "$HOME/f50-backups/F50-$(date +%Y%m%d)"
python3 scripts/f50/validate_backup.py "$HOME/f50-backups/F50-$(date +%Y%m%d)/backup-manifest.json"
```

备份目录放在仓库之外，不要提交到 Git。

### 3. 构建镜像

```bash
./scripts/feeds update -a
./scripts/feeds install -a
make menuconfig
```

选择 `Unisoc application processors`、`Unisoc UMS9620`、`ZTE F50 5G Mobile WiFi`，按需选择 `luci-app-f50-modem`、`f50-modem`、WireGuard/OpenVPN 等软件包。

```bash
make -j"$(nproc)" V=s
```

完整说明见 [docs/f50/BUILD.md](docs/f50/BUILD.md)。Docker 构建：

```bash
docker build -t f50-openwrt-builder:local tools/f50
./scripts/f50/build_recovery.sh
```

### 4. 恢复启动与刷写原则

先阅读 [USB_RECOVERY.md](docs/f50/USB_RECOVERY.md)，确认 USB 设备、Fastboot 状态、镜像大小和校验值。建议先 RAM-only bring-up，再验证 USB LAN、Wi-Fi、5G modem、LuCI 和日志，最后才考虑写入 F50DATA A/B 槽。

不要对 bootloader、基带、NV、校准、persist 或未知分区执行擦除。保护脚本不能代替人工确认。

## VPN 和移动网络教程

启动后访问 `http://192.168.100.1/`，立即修改管理员密码。进入“网络 / 移动网络 / 5G”，填写 APN，选择自动、4G、5G SA 或 5G NSA，保存并应用。确认 `wwan`、信号和注册状态正常后再配置 VPN。

### WireGuard 客户端示例

以下仅为格式示例；真实私钥、服务器地址和授权信息必须在设备本地填写：

```ini
[Interface]
Address = 10.10.0.2/32
PrivateKey = <仅在本地填写，不要提交>
DNS = 10.10.0.1

[Peer]
PublicKey = <服务端公钥>
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = vpn.example.invalid:51820
PersistentKeepalive = 25
```

```bash
wg show
ip route
ping -c 3 1.1.1.1
```

OpenVPN、sing-box、OpenClash、PassWall 等请通过 LuCI 或设备本地配置目录导入。不要把订阅链接、访问令牌、证书私钥、激活码、账号密码写入 README、Issue、截图或提交记录。

## 故障排查

### 没有 5G 或 `wwan`

检查 SIM、APN、漫游和频段；查看 `logread -e f50-modemd`、`ubus call f50.modem status`。高级 IPC/AT 映射可能仍依赖设备采集结果，见 [MODEM_CONTRACT.md](docs/f50/MODEM_CONTRACT.md)。

### VPN 能连上但不能上网

确认默认路由、DNS、MTU、防火墙 zone、IPv4/IPv6 和策略路由。移动网络可从 MTU 1280 或 1360 开始测试。

### 无法进入 Fastboot 或恢复模式

停止刷写并保留原厂状态，运行只读检查脚本，记录 USB 枚举和错误输出。不要反复擦除或尝试未经验证的 boot 镜像。

## 芯片与兼容性关键词

F50 相关硬件记录指向 **Unisoc UMS9620**，公开资料中常见对应命名包括 **T770 / T820 / T760**。T750、T8300、V510、V620、V517、SC9863A、T618、T616 等名称经常出现在展锐平台或相近设备讨论中，但它们**不是本仓库承诺支持的同型号**。不同芯片即使同属 Unisoc，也可能在 DTB、基带接口、分区、USB recovery、Wi-Fi 和电源管理上完全不同。

搜索关键词也包括：ZTE F50、ZTE F50 5G、中兴 F50、中兴 5G 随身 WiFi、移动版 F50、联通版 F50、海外版 F50、Unisoc UMS9620、Tangula T770、T820、T760、T750、T8300、V510、V620、V517、SC9863A、T618、T616、移动 CPE、MiFi、工业网关。上述相近名称仅用于搜索和失败排查，不代表已验证可刷写。

## 开发与测试

```bash
python3 scripts/f50/check_port.py
python3 -m unittest discover -s tests/f50 -p 'test_*.py'
./scripts/f50/verify_release.sh
```

测试矩阵见 [TEST_MATRIX.md](docs/f50/TEST_MATRIX.md)。实体 F50 的冷启动、Wi-Fi、5G、写入和长期稳定性仍应在真实硬件上完成。

## 贡献、许可证与免责声明

Issue 请包含仓库版本、设备地区、SoC/固件版本、复现步骤和脱敏日志。删除 IMEI、序列号、手机号、SIM 信息、密码、Token、私钥、激活码和完整备份后再发布。

本项目基于 OpenWrt / iStoreOS 及第三方组件，许可证以各目录中的 `LICENSE`、`COPYING` 和包元数据为准。刷写、解锁、改造和 VPN 使用可能影响保修、运营商服务和设备安全，请遵守所在地法律与服务条款。作者不对数据丢失、设备变砖、网络中断或第三方服务不可用负责。
