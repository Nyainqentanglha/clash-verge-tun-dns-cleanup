# Clash Verge TUN DNS Cleanup

一个用于 macOS 的小型 LaunchAgent：在网络设置变化后，清理 Wi-Fi 上特定的手动 DNS 覆盖值。

## 作用范围

- 脚本只读取名为 `Wi-Fi` 的网络服务的手动 DNS。
- **仅当读取结果恰好是 `114.114.114.114`** 时，才运行 `networksetup -setdnsservers 'Wi-Fi' Empty`。
- 其他 DNS 值、多个 DNS 地址及其他网络服务不会被脚本修改。如果 `114.114.114.114` 是你有意设置的 DNS，不要安装。
- `RunAtLoad` 让任务在用户登录加载时运行；`WatchPaths` 让它在 macOS 网络偏好文件变化时再次运行。`ThrottleInterval=1` 是限流值，并非每秒运行一次。
- 清除手动覆盖后，DHCP 或设备管理提供的 DNS 仍可能出现在 `scutil --dns` 中。脚本不会修改这些来源。

本仓库不包含代理配置、节点凭据或个人路径。

## 文件

| 文件 | 用途 |
| --- | --- |
| [`clear-clash-verge-tun-dns`](clear-clash-verge-tun-dns) | 执行精确匹配和清理 |
| [`io.github.clashverge.clear-tun-dns.plist.template`](io.github.clashverge.clear-tun-dns.plist.template) | 用户级 LaunchAgent 模板 |

## 安装

1. 确认目标机器是 macOS，网络服务名确实为 `Wi-Fi`，并且希望清理上述特定手动 DNS。
2. 将脚本复制到 `~/.local/bin/clear-clash-verge-tun-dns`，赋予执行权限。
3. 将 plist 模板中的 `__HOME__` 替换为目标用户的**绝对用户目录**，保存为 `~/Library/LaunchAgents/io.github.clashverge.clear-tun-dns.plist`。不要直接加载未替换的模板。
4. 先运行 `plutil -lint` 检查新 plist，再用下面的命令加载：

```sh
launchctl bootstrap "gui/$(id -u)" "$HOME/Library/LaunchAgents/io.github.clashverge.clear-tun-dns.plist"
```

检查任务及当前手动 DNS：

```sh
launchctl print "gui/$(id -u)/io.github.clashverge.clear-tun-dns"
networksetup -getdnsservers 'Wi-Fi'
```

`launchctl` 的运行次数表示脚本被触发的次数，并不表示实际清除了同样次数的 DNS。如果 `networksetup` 报权限错误，需要在目标机器上处理授权；不要把 `sudo` 静默写入 LaunchAgent。

## 卸载

```sh
launchctl bootout "gui/$(id -u)" "$HOME/Library/LaunchAgents/io.github.clashverge.clear-tun-dns.plist"
```

随后删除安装到 `~/.local/bin/` 和 `~/Library/LaunchAgents/` 的两个文件。卸载不会改动当前 DNS。

## 验证范围

运行 `zsh -n clear-clash-verge-tun-dns` 和 `plutil -lint io.github.clashverge.clear-tun-dns.plist.template` 可检查语法。此仓库没有修改 Clash Verge、本机已安装的任务或系统 DNS。
