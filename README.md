# My-Openwrt
自编译openwrt，基于lede源码
源码地址：https://github.com/coolsnowwolf/lede  感谢Lean
1：首先装好 Ubuntu 64bit，推荐 Ubuntu 20.04 LTS x64
2：命令行输入sudo apt-get update
然后输入sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync
3:下载lede源码git clone https://github.com/coolsnowwolf/lede
4:进入lede目录cd lede 
5:修改feeds.conf.default文件，缝合其他插件,添加下面代码复制到 lede源码根目录 feeds.conf.default 文件.
src-git kenzo https://github.com/kenzok8/openwrt-packages
src-git small https://github.com/kenzok8/small
6:添加自定义app，lede目录运行。
git clone https://github.com/Zxilly/UA2F.git package/UA2F
git clone https://github.com/CHN-beta/rkp-ipid.git package/rkp-ipid
git clone https://github.com/BoringCat/luci-app-mentohust.git package/luci-app-mentohust
git clone https://github.com/liuqun/mentohust.git package/mentohust

7：./scripts/feeds update -a 同步feeds
./scripts/feeds install -a  安装feeds
make menuconfig 选择配置
8：勾选需要的app
勾选上ua2f
network->Routing and Redirection->ua2f
勾选上ipid
kernel-modules->Other modules->kmod-rkp-ipid
选上模块
make menuconfig
# network->firewall->iptables-mod-filter
# network->firewall->iptables-mod-ipopt
# network->firewall->iptables-mod-u32
# network->firewall->iptables-mod-conntrack-extra
# kernel modules->Netfilter Extensions->kmod-ipt-u32
# kernel modules->Netfilter Extensions->kmod-ipt-ipopt
9：内核编译配置 
make kernel_menuconfig -j$(nproc) V=s
Networking support ->Networking options ->Network packet filtering framework (Netfilter) （要先选中再进去）->Core Netfilter Configuration -> 选中:
# Netfilter NFNETLINK interface
# Netfilter LOG over NFNETLINK interface
# Netfilter connection tracking support
# Connection tracking netlink interface
# NFQUEUE and NFLOG integration with Connection Tracking
10：预编译需要的软件包
make download -j$(nproc) V=s
11：编译
make -j$(nproc) V=s
