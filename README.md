# immortalwrt
## 一、设置网络
### 1.查看网卡名称，常为eth0
  ```bash
  ip link show
  ```
### 2. Docker 中创建一个 macvlan 网络
  ```bash
  docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  macnet
  ```
### 3. 打印docker中的macvlan网络是否创建成功
  ```bash
  docker network ls
  ```
### 4.创建虚拟接口
  - macvlan 的一个特性是宿主机无法直接与容器通信。如果你的需求是让宿主机与 OpenWrt 容器通信，你需要在宿主机上创建一个虚拟接口（通常称为 macvlan 子接口），并将其加入同一 macvlan 网络。
  ```bash
  ip link add macvlan-shim link eth0 type macvlan mode bridge
  ip addr add 192.168.1.11/24 dev macvlan-shim
  ip link set macvlan-shim up
  ```
### 5.拉取镜像
  ```bash
  docker pull yexundao/immortalwrt:latest
  ```
  - latest/arm：最新arm版本
  - amd：最新amd版本
### 6.创建容器
  ```bash
  docker run --name immortalwrt -d --network macnet --privileged --restart=always yexundao/immortalwrt:latest /sbin/init
  ```
  - 默认为arm版本，启动后通过管理IP直接使用，无需后续操作
### 7.进入容器
  ```bash
  docker exec -it immortalwrt sh
  ```
## 8.设置ip并重启系统
  ```bash
  cat <<EOF > /etc/config/network
  config interface 'loopback'
    option device 'lo'
    option proto 'static'
    option ipaddr '127.0.0.1'
    option netmask '255.0.0.0'
  
  config globals 'globals'
    option ula_prefix 'fd98:9655:39f9::/48'
  
  config interface 'lan'
    option proto 'static'
    option netmask '255.255.255.0'
    option ipbassign '60'
    option ipaddr '192.168.1.100'
    option gateway '192.168.1.1'
    option dns '223.5.5.5 1.1.1.1'
    option device 'eth0'
  EOF
  ```
### 9.在docker 版的immortalwrt中安装一些必备插件
  ```bash
  docker exec -it immortalwrt sh 
  opkg update
  opkg install luci-i18n-ttyd-zh-cn
  opkg install luci-i18n-filebrowser-go-zh-cn
  opkg install luci-i18n-argon-config-zh-cn
  opkg install openssh-sftp-server
  opkg install luci-i18n-samba4-zh-cn
  ```
### 10.安装iStore商店(ARM64 & x86-64通用)
  ```bash
  wget -qO imm.sh https://cafe.cpolar.top/wkdaily/zero3/raw/branch/main/zero3/imm.sh && chmod +x imm.sh && ./imm.sh
  ```
### 11.安装网络向导和首页(ARM64 & x86-64通用)
  ```bash
  is-opkg install luci-i18n-quickstart-zh-cn
  ```
### 12.更新源
- 在https://help.mirrorz.org/immortalwrt/中查找可用源
- 在页面-软件包-配置opkg-/etc/opkg/distfeeds.conf 修改为
  ```bash
  src/gz immortalwrt_core https://mirror.nju.edu.cn/immortalwrt/releases/24.10.0/targets/armsr/armv8/packages
  src/gz immortalwrt_base https://mirror.nju.edu.cn/immortalwrt/releases/24.10.0/packages/aarch64_generic/base
  src/gz immortalwrt_kmods https://mirror.nju.edu.cn/immortalwrt/releases/24.10.0/targets/armsr/armv8/kmods/6.6.73-1-a7eafb055ecc4891236a188e564e21ff
  src/gz immortalwrt_luci https://mirror.nju.edu.cn/immortalwrt/releases/24.10.0/packages/aarch64_generic/luci
  src/gz immortalwrt_packages https://mirror.nju.edu.cn/immortalwrt/releases/24.10.0/packages/aarch64_generic/packages
  src/gz immortalwrt_routing https://mirror.nju.edu.cn/immortalwrt/releases/24.10.0/packages/aarch64_generic/routing
  src/gz immortalwrt_telephony https://mirror.nju.edu.cn/immortalwrt/releases/24.10.0/packages/aarch64_generic/telephony
  ```






