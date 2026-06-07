---
layout: post
title: "云环境入侵应对方案：OpenVPN 安全加固实践"
date: 2026-05-09
category: project-review
tags:
  - 安全
  - OpenVPN
  - 服务器
  - 运维
  - 数据库
  - 网络加固
excerpt: "完整的云服务器安全加固方案：通过 OpenVPN 隧道访问数据库，实现数据库不暴露公网、只允许 VPN 或本机访问的最小权限架构。"
type: concept
source_path: "/Users/zengxianming/Documents/Obsidian Vault/wiki/安全/云环境入侵应对方案.md"
---

事情的起因，是我在学习构建智能体的过程中，自己的云服务器被黑客一锅端了。为了加固服务器，我自己动手学习并部署了 OpenVPN，作为内网访问的安全通道。

这件事也让我开始思考：在 AI 蓬勃发展的今天，AI 投毒、模型污染等安全事件屡有发生——未来的 AI 安全会走向何方？我们迫切需要把安全这件事重视起来。以下是我记录用服务器内网部署服务，采用 OpenVPN 来加固的一个具体的实战方案

# 服务器部署 + OpenVPN 安全加固方案

---

## 一、最终架构（目标）

```
公网
  ↓ OpenVPN（1194/udp）
  ↓ VPN 内网（10.8.0.x）
  ↓ MySQL / PostgreSQL / Redis（仅本机或 VPN 访问）
```

**核心原则：**

> 数据库不暴露公网，只允许 VPN 或本机访问。

---

## 二、系统准备（Ubuntu）

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 三、安装数据库（全部本机模式）

### 3.1 安装 MySQL

```bash
sudo apt install -y mysql-server
```

**修改监听地址：**

```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

改成：

```
bind-address = 10.8.0.1  # 自己的内网 IP
```

启动：

```bash
sudo systemctl restart mysql
```

### 3.2 安装 PostgreSQL

```bash
sudo apt install -y postgresql
```

#### 修改监听地址

```bash
sudo vim /etc/postgresql/*/main/postgresql.conf
```

改成：

```
listen_addresses = 'localhost,10.8.0.1'   # 监听内网 IP
```

#### 修改 pg_hba.conf，允许 VPN 网段访问

```bash
# 找到配置文件路径
find /etc/postgresql -name 'pg_hba.conf' | head -1

# 查看现有配置
sudo cat /etc/postgresql/16/main/pg_hba.conf

# 编辑配置文件
sudo vim /etc/postgresql/16/main/pg_hba.conf
```

在文件末尾添加：

```
host   all     all      10.8.0.0/24     scram-sha-256
```

---

## 四、安装 OpenVPN（不用 Docker）

### 4.1 安装

```bash
sudo apt install -y openvpn easy-rsa
```

### 4.2 初始化 PKI

```bash
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
```

在新版 Easy-RSA（Ubuntu 20+ / 22+ / 24+）中，使用 `./easyrsa` 命令。

### 4.3 一步一步正确流程

#### 1️⃣ 进入目录

```bash
cd ~/openvpn-ca
```

#### 2️⃣ 初始化 PKI

```bash
./easyrsa init-pki
```

#### 3️⃣ 生成 CA（替代 build-ca）

```bash
./easyrsa build-ca
```

会提示输入：

```
Enter New CA Key Passphrase:
```

可以设置密码（或回车跳过）。

#### 4️⃣ 生成服务端证书

```bash
./easyrsa build-server-full server nopass
```

#### 5️⃣ 生成客户端证书

```bash
./easyrsa build-client-full client1 nopass
```

#### 6️⃣ 生成 DH 参数

```bash
./easyrsa gen-dh
```

---

## 五、确认证书文件

在 `~/openvpn-ca/pki` 里应该有：

```
ca.crt
issued/server.crt
private/server.key
issued/client1.crt
private/client1.key
dh.pem
```

如果这些都有，说明 easy-rsa 没问题。

---

## 六、配置 OpenVPN 服务端

### 6.1 创建服务端目录

```bash
sudo mkdir -p /etc/openvpn/server
```

### 6.2 拷贝证书

```bash
sudo cp ~/openvpn-ca/pki/ca.crt /etc/openvpn/server/
sudo cp ~/openvpn-ca/pki/issued/server.crt /etc/openvpn/server/
sudo cp ~/openvpn-ca/pki/private/server.key /etc/openvpn/server/
sudo cp ~/openvpn-ca/pki/dh.pem /etc/openvpn/server/
```

### 6.3 创建配置文件

```bash
sudo vim /etc/openvpn/server/server.conf
```

填入（最小可用版本）：

```conf
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
server 10.8.0.0 255.255.255.0
keepalive 10 120
persist-key
persist-tun

# 允许客户端访问服务器本机
push "route 127.0.0.1 255.255.255.255"

# 可选：走全流量
# push "redirect-gateway def1 bypass-dhcp"

cipher AES-256-GCM
auth SHA256
user nobody
group nogroup
verb 3
```

---

## 七、开启 IP 转发（必须）

临时生效：

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

永久生效：

```bash
sudo vim /etc/sysctl.conf
```

取消注释：

```
net.ipv4.ip_forward=1
```

---

## 八、启动 OpenVPN

```bash
sudo systemctl start openvpn-server@server
sudo systemctl enable openvpn-server@server
```

查看状态：

```bash
sudo systemctl status openvpn-server@server
```

必须是：

```
active (running)
```

---

## 九、防火墙配置（非常关键）

### 云安全组

放开：

```
1194 / UDP
```

### 本机（ufw）

```bash
sudo ufw allow 1194/udp
```

---

## 十、生成客户端配置

创建文件：

```bash
vim client1.ovpn
```

填入（替换证书内容）：

```conf
client
dev tun
proto udp
remote 你的服务器IP 1194
resolv-retry infinite
nobind
persist-key
persist-tun
cipher AES-256-GCM
auth SHA256
verb 3

<ca>
（粘贴 ca.crt 内容）
</ca>

<cert>
（粘贴 client1.crt 内容）
</cert>

<key>
（粘贴 client1.key 内容）
</key>
```

---

## 十一、Mac 使用 Viscosity

1. 打开 Viscosity
2. Import → 选择 `client1.ovpn`
3. 点击连接

---

## 十二、验证是否成功

连接后：

```bash
ping 10.8.0.1
```
