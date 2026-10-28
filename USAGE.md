# 使用说明

本工具只处理 macOS `Wi-Fi` 网络服务中**恰好等于 `114.114.114.114`** 的手动 DNS。它不会清理其他值，也不会修复所有 DNS 故障。先阅读 [README 中的原因和运行时边界](README.md#为什么清理)：当前脚本不检测 Clash Verge TUN 是否开启，自动任务可能在 TUN 运行时清除其 DNS 覆盖值。

## 1. 先确认是否适用

关闭 Clash Verge TUN 后运行：

```sh
networksetup -getdnsservers 'Wi-Fi'
```

如果结果不是单独一行 `114.114.114.114`，脚本不会做任何修改，也没有必要为此问题安装自动任务。如果这个 DNS 是你有意配置的，也不要使用本工具。网络服务不叫 `Wi-Fi` 时，先确认真实服务名；脚本当前只针对 `Wi-Fi`。

## 2. 先手动运行一次

在仓库目录执行：

```sh
zsh -n ./clear-clash-verge-tun-dns
./clear-clash-verge-tun-dns
networksetup -getdnsservers 'Wi-Fi'
scutil --dns
```

脚本仅在精确匹配时调用 `networksetup -setdnsservers 'Wi-Fi' Empty`。`networksetup` 显示没有手动 DNS 后，仍可能在 `scutil --dns` 中看到当前网络提供的 DNS，这是正常的。若提示权限不足，先在 macOS 网络设置中手动处理或确认系统授权；不要把无交互的 `sudo` 加入自动任务。

## 3. 确有重复残留时再安装自动任务

在仓库目录复制文件：

```sh
mkdir -p "$HOME/.local/bin" "$HOME/Library/LaunchAgents"
cp ./clear-clash-verge-tun-dns "$HOME/.local/bin/clear-clash-verge-tun-dns"
chmod 700 "$HOME/.local/bin/clear-clash-verge-tun-dns"
cp ./io.github.clashverge.clear-tun-dns.plist.template "$HOME/Library/LaunchAgents/io.github.clashverge.clear-tun-dns.plist"
chmod 600 "$HOME/Library/LaunchAgents/io.github.clashverge.clear-tun-dns.plist"
```

把复制后的 plist 中 `__HOME__` 替换为当前用户的绝对用户目录，确保 `ProgramArguments` 指向刚安装的脚本。**不要加载含 `__HOME__` 的模板。**

检查 plist 并加载：

```sh
plutil -lint "$HOME/Library/LaunchAgents/io.github.clashverge.clear-tun-dns.plist"
launchctl bootstrap "gui/$(id -u)" "$HOME/Library/LaunchAgents/io.github.clashverge.clear-tun-dns.plist"
launchctl print "gui/$(id -u)/io.github.clashverge.clear-tun-dns"
```

该 LaunchAgent 在用户登录加载时，以及 macOS 网络偏好文件变化时执行；它不是每秒运行的定时器。安装后分别检查 **TUN 开启时** 和 **TUN 关闭后** 的 DNS 与实际访问。如果 TUN 开启时受影响，立即卸载自动任务。

## 4. 卸载

```sh
launchctl bootout "gui/$(id -u)" "$HOME/Library/LaunchAgents/io.github.clashverge.clear-tun-dns.plist"
```

然后删除安装到 `~/.local/bin/` 和 `~/Library/LaunchAgents/` 的两个文件。卸载任务本身不会修改当前 DNS；如需更改当前 DNS，请在系统网络设置中操作。
