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

