下面是一套基于你当前实际排查结果整理出来的 **WSL2 中 Git 无法走 Windows 代理的完整解决方案**。你的案例里最终定位结果很明确：**Windows 代理软件开启后，浏览器能访问 GitHub，但 WSL2 里的 Git 不能正常访问；原因是 Git 自身没有正确走 Windows 代理端口，并且代理软件最初没有允许 WSL2 访问局域网端口。** 🌿

你打开“允许局域网连接”后，`curl -I -x http://172.24.128.1:7897 https://github.com` 已经返回了 `HTTP/1.1 200 Connection established` 和 `HTTP/2 200`，这说明 WSL2 已经可以通过 `172.24.128.1:7897` 访问 GitHub，代理链路已经打通。

---

## 一、问题背景

问题是：

```text
Windows 中代理软件正常开启；
Windows 浏览器可以正常访问 GitHub；
但是 WSL2 / Ubuntu 中无法执行 git clone、git pull、git push 等 Git 操作。
```

核心原因通常是：

```text
浏览器走的是 Windows 代理配置；
WSL2 里的 Git 不一定自动走 Windows 代理；
Git 的 HTTPS 通信和 SSH 通信还需要分别配置。
```

所以要先判断当前 Git 仓库到底使用的是 **HTTPS 通信** 还是 **SSH 通信**。

---

## 二、先定位当前 Git 通信方式

进入项目目录后执行：

```bash
git remote -v
```

你的返回结果是：

```text
origin  https://github.com/mesiphy/quant-v2.git (fetch)
origin  https://github.com/mesiphy/quant-v2.git (push)
```

这说明你当前仓库使用的是 **HTTPS 通信**。

判断规则如下：

```text
https://github.com/xxx/yyy.git
=> 使用 HTTPS 通信
=> 需要配置 git config http.proxy / https.proxy

git@github.com:xxx/yyy.git
=> 使用 SSH 通信
=> 需要配置 ~/.ssh/config
```

所以在你的当前仓库中，`~/.ssh/config` 暂时不是主线问题，因为你的 Git remote 是 HTTPS 地址。

---

## 三、确认 WSL2 的 Windows 网关地址

WSL2 不能直接假设使用 Windows 的 `127.0.0.1` 代理。通常需要使用 Windows 在 WSL2 网络中的网关地址。

在 WSL2 中执行：

```bash
ip route | grep default
```

可能输出类似：

```text
default via 172.24.128.1 dev eth0
```

其中：

```text
172.24.128.1
```

就是 WSL2 访问 Windows 主机的网关地址。

也可以直接提取：

```bash
ip route | awk '/default/ {print $3}'
```

如果输出：

```text
172.24.128.1
```

那么后续代理地址就应该写成：

```text
172.24.128.1:代理端口
```

在你的情况里，代理软件使用的是 **混合端口 7897**，所以完整代理地址是：

```text
http://172.24.128.1:7897
```

---

## 四、定位通信问题是否存在

先测试 WSL2 能不能通过 Windows 代理访问 GitHub。

你的代理软件混合端口是 `7897`，所以执行：

```bash
curl -I -x http://172.24.128.1:7897 https://github.com
```

一开始你执行时卡住并手动中断：

```text
^C
```

这说明当时：

```text
WSL2 无法成功通过 172.24.128.1:7897 访问 Windows 代理；
问题不在 GitHub，也不在 Git 仓库，而在 WSL2 到 Windows 代理端口这条链路。
```

后来你打开代理软件里的：

```text
允许局域网连接 / Allow LAN
```

再次执行：

```bash
curl -I -x http://172.24.128.1:7897 https://github.com
```

返回：

```text
HTTP/1.1 200 Connection established

HTTP/2 200
```

这说明：

```text
WSL2 -> Windows 代理端口 7897 -> GitHub
```

这条链路已经正常。

这一步非常关键。只有 `curl` 通过代理访问 GitHub 成功之后，再配置 Git 代理才有意义。

---

## 五、处理过程 3.1：先确定 Git 使用 SSH 还是 HTTPS

再次强调，先执行：

```bash
git remote -v
```

如果看到：

```text
origin  https://github.com/mesiphy/quant-v2.git
```

说明使用 HTTPS。

如果看到：

```text
origin  git@github.com:mesiphy/quant-v2.git
```

说明使用 SSH。

你的当前情况是：

```text
HTTPS 通信
```

所以应该优先配置 Git 的 HTTP / HTTPS 代理。

---

## 六、处理过程 3.2：如果 Git 使用 HTTPS，该如何修改

### 1. 配置 Git 的 HTTP 和 HTTPS 代理

因为你的代理软件混合端口是 `7897`，WSL2 网关是 `172.24.128.1`，所以执行：

```bash
git config --global http.proxy http://172.24.128.1:7897
git config --global https.proxy http://172.24.128.1:7897
```

解释：

```text
git config --global
表示配置当前 WSL2 用户的全局 Git 配置。

http.proxy
用于 Git 的 HTTP 请求。

https.proxy
用于 Git 的 HTTPS 请求。

http://172.24.128.1:7897
表示让 Git 通过 Windows 主机上的代理端口 7897 访问外网。
```

注意：虽然你的仓库地址是 `https://github.com/...`，但建议 `http.proxy` 和 `https.proxy` 都配置，避免后续其他 Git 操作出现不一致。

---

### 2. 查看 Git 代理配置是否生效

执行：

```bash
git config --global --get http.proxy
git config --global --get https.proxy
```

理想输出是：

```text
http://172.24.128.1:7897
http://172.24.128.1:7897
```

也可以查看所有 Git 配置来源：

```bash
git config --global --list
```

重点看里面有没有：

```text
http.proxy=http://172.24.128.1:7897
https.proxy=http://172.24.128.1:7897
```

---

### 3. 测试 Git 是否能访问远程仓库

执行：

```bash
git ls-remote https://github.com/mesiphy/quant-v2.git
```

如果成功，会输出类似：

```text
9daed270b0b004055f55ac8771cb9c5330bbe7d4        HEAD
9daed270b0b004055f55ac8771cb9c5330bbe7d4        refs/heads/main
ee6178e33567b84542edaba4fae3e9041d58e2c6        refs/heads/feat/phase2-SQLandVault
```

这说明 Git 已经可以通过代理访问 GitHub。

然后再执行：

```bash
git pull
```

或者：

```bash
git clone https://github.com/mesiphy/quant-v2.git
```

---

### 4. 如果以后想取消 Git HTTPS 代理

执行：

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

再检查：

```bash
git config --global --get http.proxy
git config --global --get https.proxy
```

如果没有输出，说明代理已经取消。

---

## 七、处理过程 3.3：如果 Git 使用 SSH，该如何修改

如果你的 remote 是这种形式：

```text
git@github.com:mesiphy/quant-v2.git
```

那就不是 HTTPS 通信，而是 SSH 通信。

这时下面这种 Git 代理配置不会生效：

```bash
git config --global http.proxy ...
git config --global https.proxy ...
```

因为 SSH 不走 Git 的 HTTP 代理配置。

---

### 1. 检查 SSH 当前配置

执行：

```bash
ssh -G github.com | grep -E "hostname|user|port|proxycommand"
```

你之前的输出是：

```text
user mesiphy
hostname github.com
port 22
canonicalizehostname false
gatewayports no
userknownhostsfile /home/mesiphy/.ssh/known_hosts /home/mesiphy/.ssh/known_hosts2
```

这说明：

```text
SSH 目标地址：github.com
SSH 目标端口：22
SSH 用户：mesiphy
代理：没有 proxycommand，说明没有走代理
```

对于 GitHub SSH，正常用户应该是：

```text
git
```

不是：

```text
mesiphy
```

所以需要配置 `~/.ssh/config`。

---

### 2. 编辑 SSH 配置文件

执行：

```bash
code ~/.ssh/config
```

或者：

```bash
nano ~/.ssh/config
```

如果文件是空的，这是正常的。可以直接写入下面内容：

```sshconfig
Host github.com
  HostName github.com
  User git
  Port 22
  ProxyCommand nc -X 5 -x 172.24.128.1:7897 %h %p
```

解释：

```text
Host github.com
表示这段配置只对 github.com 生效。

HostName github.com
表示真实连接目标是 github.com。

User git
表示 SSH 登录 GitHub 时使用 git 用户，这是 GitHub SSH clone 的标准用户。

Port 22
表示最终连接 GitHub 的 SSH 22 端口。

ProxyCommand nc -X 5 -x 172.24.128.1:7897 %h %p
表示 SSH 连接先通过 Windows 代理端口 7897 转发。
-X 5 表示使用 SOCKS5 代理。
%h 表示目标主机，也就是 github.com。
%p 表示目标端口，也就是 22。
```

因为你的代理软件是 **混合端口 7897**，通常既可以支持 HTTP，也可能支持 SOCKS5。如果 SSH 这里使用 `nc -X 5`，本质上是在用 SOCKS5 方式连接混合端口。

---

### 3. 设置 SSH 配置文件权限

执行：

```bash
chmod 600 ~/.ssh/config
```

解释：

```text
SSH 对配置文件权限比较敏感。
chmod 600 表示只有当前用户可以读写该文件。
```

---

### 4. 检查 SSH 配置是否生效

执行：

```bash
ssh -G github.com | grep -E "hostname|user|port|proxycommand"
```

理想输出应该类似：

```text
user git
hostname github.com
port 22
proxycommand nc -X 5 -x 172.24.128.1:7897 %h %p
```

看到 `proxycommand`，就说明 SSH 已经配置为走代理。

---

### 5. 测试 SSH 是否能连接 GitHub

执行：

```bash
ssh -T git@github.com
```

如果成功，可能看到：

```text
Hi mesiphy! You've successfully authenticated, but GitHub does not provide shell access.
```

这说明：

```text
SSH 认证成功；
GitHub 已识别你的 SSH key；
SSH 代理链路正常。
```

如果想看详细连接过程，可以执行：

```bash
ssh -vT git@github.com
```

重点看是否出现类似：

```text
Executing proxy command: exec nc -X 5 -x 172.24.128.1:7897 github.com 22
```

这说明 SSH 确实通过代理端口连接 GitHub。

---

### 6. 如果想把当前仓库从 HTTPS 改成 SSH

执行：

```bash
git remote set-url origin git@github.com:mesiphy/quant-v2.git
```

然后检查：

```bash
git remote -v
```

应该变成：

```text
origin  git@github.com:mesiphy/quant-v2.git (fetch)
origin  git@github.com:mesiphy/quant-v2.git (push)
```

之后再执行：

```bash
git pull
```

不过基于你当前情况，我更建议先继续使用 HTTPS，因为你已经确认 HTTPS 代理链路通了，配置更简单、稳定。

---

## 八、修改完成之后，如何确认问题已经解决

可以按下面顺序确认。

### 1. 确认 WSL2 能通过代理访问 GitHub

执行：

```bash
curl -I -x http://172.24.128.1:7897 https://github.com
```

成功标志：

```text
HTTP/1.1 200 Connection established
HTTP/2 200
```

你现在已经做到这一步了。🎉

---

### 2. 如果使用 HTTPS，确认 Git 代理配置

执行：

```bash
git config --global --get http.proxy
git config --global --get https.proxy
```

成功标志：

```text
http://172.24.128.1:7897
http://172.24.128.1:7897
```

然后测试远程仓库：

```bash
git ls-remote https://github.com/mesiphy/quant-v2.git
```

成功标志：

```text
能看到 commit hash、HEAD、refs/heads/main 等远程分支信息。
```

最后测试真实操作：

```bash
git pull
```

或者：

```bash
git clone https://github.com/mesiphy/quant-v2.git
```

---

### 3. 如果使用 SSH，确认 SSH 代理配置

执行：

```bash
ssh -G github.com | grep -E "hostname|user|port|proxycommand"
```

成功标志：

```text
user git
hostname github.com
port 22
proxycommand nc -X 5 -x 172.24.128.1:7897 %h %p
```

然后测试：

```bash
ssh -T git@github.com
```

成功标志：

```text
Hi mesiphy! You've successfully authenticated, but GitHub does not provide shell access.
```

最后测试：

```bash
git pull
```

或者：

```bash
git clone git@github.com:mesiphy/quant-v2.git
```

---

## 九、你的当前推荐执行顺序

基于你当前已经确认：

```text
代理混合端口：7897
WSL2 网关地址：172.24.128.1
curl 代理访问 GitHub：已经成功
Git remote：HTTPS
```

所以你现在最推荐执行这组命令：

```bash
git config --global http.proxy http://172.24.128.1:7897
git config --global https.proxy http://172.24.128.1:7897

git config --global --get http.proxy
git config --global --get https.proxy

git ls-remote https://github.com/mesiphy/quant-v2.git
```

如果 `git ls-remote` 成功，再执行：

```bash
git pull
```

或者如果你要重新克隆：

```bash
git clone https://github.com/mesiphy/quant-v2.git
```

---

## 十、最终结论

你的问题不是 GitHub 本身不能访问，也不是仓库不存在，而是：

```text
Windows 浏览器走了代理；
WSL2 中的 Git 没有自动走 Windows 代理；
一开始 Windows 代理软件没有允许 WSL2 通过局域网访问代理端口；
打开“允许局域网连接”后，WSL2 已经能通过 172.24.128.1:7897 访问 GitHub；
接下来只需要给 Git 的 HTTPS 通信配置 http.proxy 和 https.proxy。
```

你当前这类 HTTPS 仓库的最终解决命令就是：

```bash
git config --global http.proxy http://172.24.128.1:7897
git config --global https.proxy http://172.24.128.1:7897
git ls-remote https://github.com/mesiphy/quant-v2.git
```

如果这三步都正常，WSL2 中的 Git clone / pull / push 代理问题就基本解决了。✨