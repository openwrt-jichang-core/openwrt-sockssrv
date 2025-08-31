# microsocks OpenWrt Compilation & Usage Guide

## 项目简介
microsocks 是一个轻量级的 SOCKS5 代理服务程序，支持用户名密码认证，可运行在各种路由器架构（MIPS/ARM/x86）。适合 OpenWrt 路由器或其他嵌入式设备。

---

## 支持架构
- **MIPS (mipsel, 24kc)**
- **ARM (armv7, armv5)**
- **x86 / x86_64**

### 查询架构信息
```bash
file sockssrv
# 输出示例：ELF 32-bit LSB executable, MIPS, MIPS32 rel2 version 1 (SYSV)
```

---

## SDK 下载与选择
### 常用 SDK 下载链接（OpenWrt 19.07）
- MIPS: [openwrt-sdk-19.07.10-ramips-rt305x](https://downloads.openwrt.org/releases/19.07.10/targets/ramips/rt305x/openwrt-sdk-19.07.10-ramips-rt305x_gcc-7.5.0_musl.Linux-x86_64.tar.xz)
- ARM: 根据目标板选择合适的 SDK
- x86/x86_64: 根据目标选择 x86_64 SDK

### 解压命令
```bash
tar -xf openwrt-sdk-19.07.10-ramips-rt305x_gcc-7.5.0_musl.Linux-x86_64.tar.xz
cd openwrt-sdk-19.07.10-ramips-rt305x_gcc-7.5.0_musl.Linux-x86_64
```

---

## 编译教程
### 克隆源码
```bash
git clone https://github.com/gl-inet/openwrt.git
cd openwrt
```

### 使用 SDK 编译 microsocks
#### 设置交叉编译工具链路径
```bash
export TOOLCHAIN=../staging_dir/toolchain-mipsel_24kc_gcc-7.5.0_musl/bin/mipsel-openwrt-linux-gcc
```

#### 动态编译
```bash
$TOOLCHAIN -Wall -std=c99 -o sockssrv sockssrv.c server.c sblist.c sblist_delete.c
```

#### 静态编译（推荐）
```bash
$TOOLCHAIN -Wall -std=c99 -static -o sockssrv sockssrv.c server.c sblist.c sblist_delete.c
```

#### 注意事项
- 需要 Python 2.x 编译 OpenWrt SDK，如果报错可使用 FORCE=1 跳过
- 编译前确保交叉工具链路径正确

---

## 运行方法
```bash
# 上传到路由器
scp sockssrv root@192.168.8.1:/usr/bin/

# 运行 SOCKS5 服务
./sockssrv -u 用户名 -P 密码 -p 1080

# 示例
./sockssrv -u 123123123 -P 123123123. -p 1080
```

### 测试是否可用
```bash
# 使用 curl 测试
curl --socks5 127.0.0.1:1080 http://ipinfo.io
```

---

## 添加账号密码
- `-u` : 用户名
- `-P` : 密码
- SOCKS5 仅支持用户名密码认证

---

## 支持协议
- **SOCKS5**
- 用户名密码认证
- IPv4/IPv6
- **不支持 HTTP/HTTPS 代理模式**

---

## 常见问题
### -ash: ./sockssrv: not found
- 原因：交叉编译架构与设备不匹配
- 解决：确认工具链和目标架构一致

### 编译报错 Python 2.x
- 安装 Python 2 或使用 FORCE=1

### RPC/EOF 克隆错误
- 可尝试 `--depth=1` 或换 GitHub 镜像

---

## xBoard 添加示例
1. 将 `sockssrv` 上传至路由器
2. 设置启动命令 `/usr/bin/sockssrv -u 用户名 -P 密码 -p 1080`
3. 配置端口与防火墙允许访问

---

## 开机自启脚本示例
```bash
#!/bin/sh /etc/rc.common
START=99
start() {
    /usr/bin/sockssrv -u 123123123 -P 123123123. -p 1080 &
}
```

---

## 参考链接
- [microsocks GitHub](https://github.com/rofl0r/microsocks)
- [OpenWrt SDK](https://downloads.openwrt.org/releases/19.07.10/targets/ramips/rt305x/)
- [OpenWrt Docs](https://openwrt.org/docs/guide-developer/toolchain/use-buildsystem)





MicroSocks - multithreaded, small, efficient SOCKS5 server.
===========================================================

a SOCKS5 service that you can run on your remote boxes to tunnel connections
through them, if for some reason SSH doesn't cut it for you.

It's very lightweight, and very light on resources too:

for every client, a thread with a low stack size is spawned.
the main process basically doesn't consume any resources at all.

the only limits are the amount of file descriptors and the RAM.

It's also designed to be robust: it handles resource exhaustion
gracefully by simply denying new connections, instead of calling abort()
as most other programs do these days.

another plus is ease-of-use: no config file necessary, everything can be
done from the command line and doesn't even need any parameters for quick
setup.

History
-------

This is the successor of "rocksocks5", and it was written with
different goals in mind:

- prefer usage of standard libc functions over homegrown ones
- no artificial limits
- do not aim for minimal binary size, but for minimal source code size,
  and maximal readability, reusability, and extensibility.

as a result of that, ipv4, dns, and ipv6 is supported out of the box
and can use the same code, while rocksocks5 has several compile time
defines to bring down the size of the resulting binary to extreme values
like 10 KB static linked when only ipv4 support is enabled.

still, if optimized for size, *this* program when static linked against musl
libc is not even 50 KB. that's easily usable even on the cheapest routers.

command line options
--------------------

    microsocks -1 -q -i listenip -p port -u user -P passw -b bindaddr -w wl

all arguments are optional.
by default listenip is 0.0.0.0 and port 1080.

- option -q disables logging.
- option -b specifies which ip outgoing connections are bound to
- option -w allows to specify a comma-separated whitelist of ip addresses,
that may use the proxy without user/pass authentication.
e.g. -w 127.0.0.1,192.168.1.1.1,::1 or just -w 10.0.0.1
to allow access ONLY to those ips, choose an impossible to guess user/pw combo.
- option -1 activates auth_once mode: once a specific ip address
authed successfully with user/pass, it is added to a whitelist
and may use the proxy without auth.
this is handy for programs like firefox that don't support
user/pass auth. for it to work you'd basically make one connection
with another program that supports it, and then you can use firefox too.
for example, authenticate once using curl:

    curl --socks5 user:password@listenip:port anyurl


Supported SOCKS5 Features
-------------------------
- authentication: none, password, one-time
- IPv4, IPv6, DNS
- TCP (no UDP at this time)

Troubleshooting
---------------

if you experience segfaults, try raising the `THREAD_STACK_SIZE` in sockssrv.c
for your platform in steps of 4KB.

if this fixes your issue please file a pull request.

microsocks uses the smallest safe thread stack size to minimize overall memory
usage.
