# OpenWrt SDK & 交叉编译教程

本教程涵盖 OpenWrt 各架构 SDK 下载、交叉编译程序、运行方法以及账号密码配置示例。

---

## 1️⃣ 常见架构及 SDK 下载

| 架构 (Target) | CPU 类型 | SDK 下载链接 |
|---------------|---------|-------------|
| ramips/rt305x | MIPS 24Kc | [SDK 19.07.10](https://downloads.openwrt.org/releases/19.07.10/targets/ramips/rt305x/openwrt-sdk-19.07.10-ramips-rt305x_gcc-7.5.0_musl.Linux-x86_64.tar.xz) |
| ar71xx        | MIPS 74K  | [SDK 19.07.10](https://downloads.openwrt.org/releases/19.07.10/targets/ar71xx/) |
| bcm53xx       | MIPS 1004K | [SDK 19.07.10](https://downloads.openwrt.org/releases/19.07.10/targets/bcm53xx/) |
| x86_64        | Intel/AMD 64-bit | [SDK 19.07.10](https://downloads.openwrt.org/releases/19.07.10/targets/x86/64/) |
| aarch64       | ARM 64-bit | [SDK 19.07.10](https://downloads.openwrt.org/releases/19.07.10/targets/armvirt/64/) |
| armvirt/32    | ARM 32-bit | [SDK 19.07.10](https://downloads.openwrt.org/releases/19.07.10/targets/armvirt/32/) |

> 查询设备架构：
```bash
uname -m
cat /proc/cpuinfo
file <binary_file>
2️⃣ SDK 准备与环境
下载并解压 SDK：

bash
复制代码
wget <SDK下载链接>
tar -xf openwrt-sdk-<version>-<arch>.tar.xz
cd openwrt-sdk-<version>-<arch>
检查交叉编译器：

bash
复制代码
ls staging_dir/toolchain-*/bin/
# 例如 mipsel-openwrt-linux-gcc、arm-openwrt-linux-gcc 等
设置 Python（若 SDK 需要 Python 2）：

bash
复制代码
sudo ln -sf /usr/bin/python2.7 /usr/bin/python
python --version
3️⃣ 编译程序方法（以 microsocks 为例）
bash
复制代码
cd microsocks
../staging_dir/toolchain-<arch>/bin/<target-gcc> \
    -Wall -std=c99 \
    -o sockssrv \
    sockssrv.c server.c sblist.c sblist_delete.c
输出文件：sockssrv

检查架构：

bash
复制代码
file sockssrv
4️⃣ 支持协议
SOCKS5（带账号密码认证）

TCP/UDP 代理

可配置端口和访问控制（通过 -u 用户名和 -P 密码参数）

示例启动命令：

bash
复制代码
./sockssrv -u <username> -P <password> -p <port>
# 例如
./sockssrv -u 123123123 -P 123123123. -p 1080
5️⃣ 在设备上运行
上传二进制：

bash
复制代码
scp sockssrv root@<device_ip>:/root/
给权限并运行：

bash
复制代码
chmod +x sockssrv
./sockssrv -u <username> -P <password> -p <port>
测试代理是否可用：

bash
复制代码
curl --socks5 127.0.0.1:1080 http://example.com
6️⃣ 编译其他软件
步骤基本一致：

找到源码

使用 SDK 交叉编译器编译

上传到目标设备运行

注意：依赖库需在 SDK 中可用，否则需要额外编译或静态链接。

7️⃣ 常见问题
-bash: ./sockssrv: not found → 二进制架构与设备不匹配

undefined reference → 编译命令没有包含所有源文件

Python2 问题 → 对应版本 SDK 需要 Python2 支持

SDK 下载 404 → 需检查 OpenWrt 官网 releases 版本
