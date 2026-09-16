# F50 VPN 可安装版本

本仓库仅用于发布中兴 ZTE F50 的预编译 VPN 安装包和使用说明，不包含源码、固件源码、构建脚本或私密配置。

## 下载

请从仓库的 Releases / 发行版下载最新版 `f50-vpn-<版本>-release.tar.gz`。压缩包内包含 Android ARM64 一键安装包、UFI-TOOLS 插件入口、Android 归档包、离线安装指南和 SHA256 校验清单。

## 适用范围

- 首要目标设备：中兴 F50（ZTE F50）。
- 已获取 root 权限、可使用 UFI/Android shell 的 ARM64 设备。
- 其他设备未逐一验证；不要将本包当作完整刷机固件或跨机型刷写包。

## 安装步骤

1. 下载并解压 `f50-vpn-<版本>-release.tar.gz`。
2. 进入解压目录，执行 `sha256sum -c SHA256SUMS`。
3. 所有文件显示 `OK` 后，阅读包内 `安装说明.md`。
4. 通过 UFI-TOOLS 导入 `f50-vpn-<版本>-ufi-plugin.txt`，或将 `.run` 安装包上传到已授权的 root shell。
5. 在设备上先执行预检，再执行安装：

```sh
chmod 0755 /data/local/tmp/f50-vpn-<版本>-android-arm64.run
sh /data/local/tmp/f50-vpn-<版本>-android-arm64.run --verify
sh /data/local/tmp/f50-vpn-<版本>-android-arm64.run
```

6. 安装完成后按包内指南打开管理页面并在设备本地完成网络配置。

## 安全与免责

发行包不包含激活码、授权码、账号、密码、订阅链接、API Token、私钥或设备凭据。请仅在你拥有或获授权管理的设备上使用，并自行承担 root、网络配置及设备改造风险。

## 搜索关键词

VPN、中兴 F50、ZTE F50、移动 VPN、5G VPN、5G CPE、5G 随身 WiFi、移动路由器 VPN、UFI、UFI-TOOLS、ARM64、Android VPN、OpenWrt VPN、透明代理、路由器 VPN、随身 WiFi VPN、MiFi VPN、蜂窝网络 VPN、Unisoc、紫光展锐、UMS9620、Tangula、T770、T820、T760、T750、T8300、V510、V620、V517、SC9863A、T618、T616、失败设备兼容性、同芯片设备排查。

上述相近芯片或设备名称仅用于检索和兼容性排查，不代表已验证支持或可刷写。
