# 需要搭建远程工作流吗

我确实工作需要
需要进行技术积累

# 这篇指南写作初衷

在搭建自己的工作流的过程中，试图参考网络上的一些教程，结果由于半吊子+心态急躁，导致安装过程坎坷不断，包括但不限于：wsl2连接本地网络出错、本地端口，通信过程不懂、代理转发弄错。
在借助AI解决问题的过程中，问题就像乱麻一样越理越多，最后 人崩溃、AI幻觉。

**信息也许获取门槛降低，但优质的信息难得，更难得的是处理信息的能力**
因此，我想借这个机会梳理一下搭建流程，丰富一下自己的知识。
# Mac VS Code 连接 Windows WSL2

## 总体架构
```text
Mac VS Code
   ↓ SSH
Windows 局域网 IP:22
   ↓ Windows 端口转发 portproxy
WSL2 IP:22222 或 22
   ↓
WSL2 里的 sshd
   ↓
WSL2 里的 VS Code Server
   ↓
Mac VS Code 显示远程文件、终端、插件环境
```
![[Pasted image 20260603145018.png]]
## 设备与软件清单
本地主机：windows11 
远程主机： mac air M1
### 相关软件

| 系统/软件               | 电脑位置  | 配置过程 | 备注                                    |
| ------------------- | ----- | ---- | ------------------------------------- |
| WSL2                | 本地    |      |                                       |
| WSL2 的SSH服务         |       |      |                                       |
| codex cli           | 本地&远程 |      | 可选                                    |
| VSCode codex插件      | 本地&远程 |      | 可选，远程启动的Mac端调用的codex窗口使用的是本地的codex配置？ |
| Vscode Remote-SSH插件 | 远程    |      | 通过Vscode远程连接必需插件                      |
| Clash 代理软件          |       |      | WSL2访问外界                              |
|                     |       |      |                                       |
|                     |       |      |                                       |

# 具体过程

**tips**: shell 表示在windows的powershell 输入的命令； bash 表示在wsl2 或者mac输入的命令

## 1. Windows 与 WSL2 基础环境
### 1.1 安装WSL2
```shell
wsl --install
```
这条命令会自动启用所需 Windows 功能、安装 WSL2（默认会安装 Ubuntu），并提示重启。重启后，首次启动 Ubuntu 时会要求你创建 Linux 的用户名和密码。
如果你使用的是最新版 Windows 11，`wsl --install` 通常是最简单的方式。
- 检查安装是否成功
```shell
wsl -l -v
```
列出已安装的 Linux 发行版并查看版本,正常输出结果应当是：
```shell
NAME STATE VERSION  
*Ubuntu Running 2  
docker-desktop Stopped 2
```
其中参数意义：
```shell
wsl --list --verbose # 列出已安装的 Linux 发行版;显示详细信息
前面的 `*` 表示默认发行版
```
### 1.2 初始化 Ubuntu
第一次进入会提示创建用户名、密码，正常设置即可，此时的用户名、密码后续在连接的时候需要使用，需要记住
- **登录后建议先更新系统包索引并升级**
```bash
sudo apt update && sudo apt upgrade -y
```
解释：
```markdown
`sudo`：用管理员权限执行命令。  因为更新系统软件包需要修改系统目录，所以普通用户权限不够，需要加 `sudo`
`apt`：Ubuntu / Debian 系统的软件包管理工具。  
你可以把它理解成 Ubuntu 里的“应用商店命令行版”，用来安装、更新、卸载软件
`update`：更新“软件包索引列表”。  
注意，它不是升级软件本体，而是告诉系统：“去软件源服务器看看，现在有哪些软件、版本是多少。”  
类似于刷新软件商店的商品目录。
根据刚才 `apt update` 获取到的新版本信息，把当前系统里已经安装的软件升级到更新版本
输出结果会询问是否升级 yes即可， -y 表示自动同意
```
补充linux常用命令（自己用的多了就记住了，记不住的说明用的少，不用记
```bash
pwd            # 显示当前目录
ls -la         # 列出当前目录文件（含隐藏）
cd ~           # 回到家目录
mkdir myproj   # 新建文件夹
cd myproj
touch hello.c  # 新建文件
nano hello.c   # 用一个简单编辑器打开文件（不熟可以用 nano）
cat hello.c    # 显示文件内容

```
### 1.3 安装常用的工具
注意，这里的工具是在wsl2系统里面的，与windows不互通也不应当互通。
尤其注意 npm 工具
如果使用vscode的话，这里只需要在windows打开vscode 安装对应的插件即可。
一般对于编程开发，必须的插件有 git python etc.
	-  **Remote - WSL**
	- docker destop (非必需)
### 1.4 安装并启动 SSH 服务

#### 1.4.1 设置自动启动


## 2. WSL2 网络与代理配置（可选）

### 2.1 检查当前是否需要配置代理
在配置之前，需要想清楚是否需要配置代理，我的目的是为了处理：
	- 在wsl2中使用codex连接太慢
	- Git同步有时候可以有时候不可以
	- sudo apt update 在开启电脑代理TUN模式的时候，无法连接

检查网站连接情况,
测试网络是否能连通 Ubuntu 软件源服务器，以及服务器返回了什么状态
让系统按照当前的 DNS / hosts / NSS 解析规则，查询 `archive.ubuntu.com` 对应的 IP 地址。
```bash
curl -I http://archive.ubuntu.com/ubuntu
getent hosts archive.ubuntu.com
```
### 2.2 配置系统级别代理
#### 2.2.1 查看当前代理
cat /etc/resolv.conf 查看当前LinuxDNS 解析配置文件，DNS 的作用是把域名转换成 IP 地址
```bash
env | grep -i proxy
```
检查git代理
```bash
git config --global --get http.proxy
git config --global --get https.proxy
```


检查vscode进程情况
```bash
pgrep -af vscode-server
```
返回结果解释：
```text
1784 /home/mesiphy/.vscode-server/code-8761a5560cfd65fdd19ce7e2bd18dab5c0a4d84e --cli-data-dir /home/mesiphy/.vscode-server/cli agent host
PID 用户目录 VS Code Server 的可执行程序 CLI 数据目录 VS Code Remote 使用的后台代理服务
```
**访问外网时的公网出口 IP**
```bash 
curl https://api.ipify.org
```
Windows 和 WSL2 的出网路径不同。最常见原因是：Windows 走了代理/VPN，但 WSL2 没走；

得到的路由表。它的作用是告诉系统：**访问不同 IP 地址时，数据包应该从哪里出去**。
`default` 表示默认路由
```bash
ip route
```

问外网时会先走 Windows 宿主机网关 `172.24.128.1`
#### 2.2.2 临时配置测试是否能够成功通信

#### 2.2.3 永久配置
**修改 ~/.zshrc文件**
在文件末尾加入： 注意这里的7897 应当与代理软件显示的端口保持一致

```
export HTTP_PROXY="http://172.24.128.1:7897"
export HTTPS_PROXY="http://172.24.128.1:7897"
export ALL_PROXY="socks5://172.24.128.1:7897"
export http_proxy="$HTTP_PROXY"export https_proxy="$HTTPS_PROXY"export all_proxy="$ALL_PROXY"
```
**检查配置是否生效**
```bash
echo $HTTP_PROXY  
echo $HTTPS_PROXY  
echo $ALL_PROXY
```


## 3. SSH 通信链路与端口转发
### 3.1 远程连接的原理
**在mac的使用逻辑**
```text
Mac
  |
  | ssh mesiphy@192.168.1.5 -p 22
  v
Windows 局域网 IP:22
  |
  | portproxy / wslrelay 转发
  v
WSL2 Ubuntu 的 SSH 服务
```

### 3.2 查看 Windows 相关情况
#### 3.2.1 查看 Windows 局域网 IP
```shell
ipconfig
ipconfig | findstr IPv4

无线局域网适配器 WLAN:  
  
IPv4 地址 . . . . . . . . . . . . : 192.168.1.5   # Windows 局域网 IP，外部电脑连接使用
子网掩码 . . . . . . . . . . . . : 255.255.255.0  
默认网关 . . . . . . . . . . . . : 192.168.1.1 #通常是 Windows 给 WSL2 虚拟网络用的网关 IP，内部通信使用
```

```shell
grep -n '^Port\|^#Port' /etc/ssh/sshd_config
```
检查某个端口是否被监听
```shell
netstat -ano | findstr LISTENING | findstr :22 
```

#### 3.2.2 配置 Windows portproxy
```SHELL
netsh interface portproxy show all
# 更现代的命令
Get-NetTCPConnection -State Listen
Get-NetTCPConnection -LocalPort 22
```

输出解释
(Windows 在 0.0.0.0:22 上监听  ; 收到连接后，转发到 WSL2 的 172.24.138.246:22222)
```shell
侦听 ipv4:                 连接到 ipv4:

地址            端口        地址            端口
--------------- ----------  --------------- ----------
0.0.0.0         22          172.24.138.246  22222
```
#### 3.2.3 查看 WSL2 IP 与 sshd 端口
```shell
netstat -ano | findstr :22 # 返回进程PID
tasklist | findstr PID
```
#### 3.2.4 查看防火墙规则
```shell
Get-NetFirewallRule | findstr OpenSSH
Get-NetFirewallPortFilter | Where-Object {$_.LocalPort -eq 22}
```

#### 3.2.5 配置 SSH
检查 SSH 配置文件里的端口
```bash
grep -n '^Port\|^#Port' /etc/ssh/sshd_config
```
#### 3.2.6 检查 sshd 情况
**sshd 服务是否运行、监听端口是否正确、能否从本机 SSH 登录**
sudo systemctl status ssh --no-pager -l
ss -tlnp | grep sshd
## 4. Mac 端 VS Code Remote-SSH

### 4.1 生成 SSH 密钥——远程电脑的密钥
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
```text
SSH 自带的密钥生成工具  -type 指定密钥类型为 `ed25519`  给这个密钥添加一个注释
```
执行后，它通常会问你密钥保存到哪里,直接按回车，它会保存到默认路径
```bash
# mac
/Users/mesiphy/.ssh/id_ed25519 # 本机密钥
/Users/mesiphy/.ssh/id_ed25519.pub # 公钥
# linux
/home/mesiphy/.ssh/id_ed25519
/home/mesiphy/.ssh/id_ed25519.pub
```

Enter passphrase:这是给私钥设置密码。你可以设置，也可以直接回车留空

### 4.2 把 Mac 公钥加入 WSL2 的 authorized_keys
在 Mac 终端查看公钥并复制：
```bash
cat ~/.ssh/id_ed25519.pub
```
然后进入 WSL2 Ubuntu，创建 .ssh 目录：
```bash
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
```
把刚才复制的 Mac 公钥粘贴进去，保存退出。
然后设置权限：
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
#### 4.2.1 权限编码介绍
Linux 文件权限：
```text
7 = 当前用户：可读、可写、可进入
0 = 同组用户：没有任何权限
0 = 其他用户：没有任何权限
4 = 读 read
2 = 写 write
1 = 执行 execute
6 = 可以读、可以写
7 = 可以读、可以写、可以执行
```
### 4.3 配置 SSH config
保存密钥到本地主机的SSH配置文件以免密码登录
```bash
nano ~/.ssh/config
```
保存
```text
	Host my-wsl2
    HostName 192.168.1.23 # windows的局域网ip
    User mesiphy # WSL2 用户名
    Port 22 # 应当是windows的端口，默认是22
```
### 4.4 测试连接
在使用 VS Code Remote-SSH 之前，建议先在 Mac 终端里直接测试 SSH 是否能连上。

```bash
ssh my-wsl2
ssh mesiphy@192.168.1.23 -p 22
```
```text
mesiphy        # WSL2 Ubuntu 用户名
192.168.1.23   # Windows 的局域网 IP
-p 22          # Windows 对外暴露的 SSH 端口
```
### 4.5 用 VS Code 连接
确认普通 SSH 可以连接后，再打开 Mac 上的 VS Code。插件 Remote - SSH
Command + Shift + P : Remote-SSH: Connect to Host...

### 4.6 VScode调用插件的问题
#### 通信问题
由于未知原因，shell 的代理在被codex进程调用时，会被强制修改成 localhost 的形式，因此需要强制覆盖：
要使用显示代理，配置文件 remote settings.json 文件

```text
{
  "http.useLocalProxyConfiguration": false,
  "http.proxy": "http://172.24.128.1:7897",
  "http.proxySupport": "override"
}
```

```text
在 Remote-SSH 场景下，**不要把本地 VS Code，也就是 Mac 端的代理配置带到远端扩展宿主里使用**。
尽量阻止 VS Code 把 Mac 本地代理或 sandbox localhost 代理注入给远端 Codex，并强制远端 Codex 使用 `http://172.24.128.1:7897`
```

#### 确认 codex插件通信

```bash
ps aux | grep -i "codex app-server" | grep -v grep
tr '\0' '\n' < /proc/新CodexPID/environ | grep -i proxy
```

# 写在最后
通过梳理这篇文章，我的收获是：
- 更清晰的理解了整个通信的过程
	- 对IP，port proxy有了了解
	- 对整个系统通信交流的复杂性有了直观的认识——CS真是博大精深
- 感受到了无比的喜悦、坚定的平静，
	- 在克服所谓的困难中行动，**在行动中平静、在行动中寻找自己**
