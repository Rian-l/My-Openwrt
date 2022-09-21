# My-Openwrt基于lede源码
源码地址：https://github.com/coolsnowwolf/lede  感谢Lean

#### 使用步骤

1：首先装好 Ubuntu 64bit，推荐 Ubuntu 20.04 LTS x64

2：命令行输入`sudo apt-get update`

然后输入`sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync`

3:下载lede源码`git clone https://github.com/coolsnowwolf/lede`


4:进入lede目录cd lede


5:修改feeds.conf.default文件，缝合其他插件,添加下面代码复制到 lede源码根目录feeds.conf.default文件.

`src-git kenzo https://github.com/kenzok8/openwrt-packages`

`src-git small https://github.com/kenzok8/small`


6:添加自定义app，lede目录运行。

`git clone https://github.com/Zxilly/UA2F.git package/UA2F` /UA2F

`git clone https://github.com/CHN-beta/rkp-ipid.git package/rkp-ipid`  /ipid

`git clone https://github.com/BoringCat/luci-app-mentohust.git package/luci-app-mentohust` /mentohust图形化界面

`git clone https://github.com/liuqun/mentohust.git package/mentohust` /mentohust主程序

7：`./scripts/feeds update -a` 同步feeds

`./scripts/feeds install -a` 安装feeds

`make menuconfig` 选择配置

8：勾选ua2f/ipid以及你所需要的插件需要的app

勾选上ua2f

network->Routing and Redirection->ua2f

勾选上ipid

kernel-modules->Other modules->kmod-rkp-ipid

选上模块

network->firewall->iptables-mod-filter

network->firewall->iptables-mod-ipopt

network->firewall->iptables-mod-u32

network->firewall->iptables-mod-conntrack-extra

kernel modules->Netfilter Extensions->kmod-ipt-u32

kernel modules->Netfilter Extensions->kmod-ipt-ipopt


9：内核编译配置

`make kernel_menuconfig -j$(nproc) V=s`

Networking support ->Networking options ->Network packet filtering framework (Netfilter) （要先选中再进去）->Core Netfilter Configuration -> 选中:

Netfilter NFNETLINK interface

Netfilter LOG over NFNETLINK interface

Netfilter connection tracking support

Connection tracking netlink interface

NFQUEUE and NFLOG integration with Connection Tracking

10：预编译需要的软件包

`make download -j$(nproc) V=s`

11：编译

`make -j$(nproc) V=s`

/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

### 防检测配置：

1：修改主机名，设定时区，IP地址

命令：`/package/base-files/files/bin/config_generate` //可修改ntp 服务器，openwrt访问地址。

 ntp服务器设置为：`ntp1.aliyun.com、time1.cloud.tencent.com、stdtime.gov.hk 、pool.ntp.org`
 
 2.进入路由器ttyd终端执行(启动ua2f)：
```c 
uci set ua2f.enabled.enabled=1

uci set ua2f.firewall.handle_fw=1

uci set ua2f.firewall.handle_tls=1

uci set ua2f.firewall.handle_mmtls=1

uci set ua2f.firewall.handle_intranet=1

uci commit ua2f`

service ua2f enable`

service ua2f start`
```
3：自启动项配置
```c
iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-ports 53
iptables -t nat -A PREROUTING -p tcp --dport 53 -j REDIRECT --to-ports 53

iptables -t mangle -N ua2f
iptables -t mangle -A ua2f -d 10.0.0.0/8 -j RETURN
iptables -t mangle -A ua2f -d 127.0.0.0/8 -j RETURN
iptables -t mangle -A ua2f -d 192.168.0.0/16 -j RETURN # 不处理流向保留地址的包
iptables -t mangle -A ua2f -p tcp --dport 443 -j RETURN
iptables -t mangle -A ua2f -p tcp --dport 22 -j RETURN # 不处理 SSH 和 https
iptables -t mangle -A ua2f -p tcp --dport 80 -j CONNMARK --set-mark 44
iptables -t mangle -A ua2f -m connmark --mark 43 -j RETURN # 不处理标记为非 http 的流 (实验性)
iptables -t mangle -A ua2f -m set --set nohttp dst,dst -j RETURN
iptables -t mangle -A ua2f -p tcp --dport 80 -m string --string "/mmtls/" --algo bm -j RETURN # 不处理微信的 mmtls
iptables -t mangle -A ua2f -j NFQUEUE --queue-num 10010

iptables -t mangle -A FORWARD -p tcp -m conntrack --ctdir ORIGINAL -j ua2f

//防时钟偏移检测
iptables -t nat -N ntp_force_local
iptables -t nat -I PREROUTING -p udp --dport 123 -j ntp_force_local
iptables -t nat -A ntp_force_local -d 0.0.0.0/8 -j RETURN
iptables -t nat -A ntp_force_local -d 127.0.0.0/8 -j RETURN
iptables -t nat -A ntp_force_local -d 192.168.0.0/16 -j RETURN
iptables -t nat -A ntp_force_local -s 192.168.0.0/16 -j DNAT --to-destination 192.168.1.1

//通过 iptables 修改 TTL 值
iptables -t mangle -A POSTROUTING -j TTL --ttl-set 64

//iptables 拒绝 AC 进行 Flash 检测
iptables -I FORWARD -p tcp --sport 80 --tcp-flags ACK ACK -m string --algo bm --string " src=\"http://1.1.1." -j DROP`
```


### 联系我
Email：9351232462qq.com

## 感激
感谢以下的项目,排名不分先后

* [OpenWrt 编译与防检测部署教程](https://www.notion.so/sunbk201public/OpenWrt-f59ae1a76741486092c27bc24dbadc59) /教程
* [从编译 OpenWrt 到使用 – 重理工校园网](https://blog.iyatt.com/?p=6906) /重理工
* [Applications 添加插件应用说明](https://www.right.com.cn/forum/thread-344825-1-1.html) /插件说明
* [Ubuntu下用Lean源码编译openwrt](https://shimo.im/docs/gYjt9QVr8T9Vhv9X/read) /教程
* [使用openwrt路由过校园网多设备检测](https://github.com/EOYOHOO/Campus-network) /教程
* [OpenWrt/LEDE修改源码编译自定义路由器系统](http://www.5lazy.cn/post-147.html) /教程
* [MentoHUST-OpenWrt-ipk](https://github.com/KyleRicardo/MentoHUST-OpenWrt-ipk) /mentohust主程序
* [LEDE LuCI for MentoHUST](https://github.com/BoringCat/luci-app-mentohust) /mentohust图形化界面
* [UA2F](https://github.com/Zxilly/UA2F) /UA2F
* [rpk ipid]（https://github.com/CHN-beta/rkp-ipid）
* [argon主题]（https://github.com/thinktip/luci-theme-neobird） 



