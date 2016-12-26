# padavan-complic
路由器固件编译


配置编译环境

安装虚拟机

安装Ubuntu 16.04LTS

安装 git
<pre>
sudo apt-get update 
sudo apt-get install git
</pre>
转到/opt目录，并且用命令克隆 Padavan 仓库
<pre>
cd /opt
sudo git clone https://bitbucket.org/padavan/rt-n56u.git
</pre>

安装编译固件所需要的工具
<pre>

sudo apt-get install autoconf automake bison build-essential flex gawk gettext gperf libtool pkg-config zlib1g-dev libgmp3-dev libmpc-dev libmpfr-dev texinfo python-docutils mc
</pre>

编译一个交叉编译的工具链
<pre>
cd /opt/rt-n56u/toolchain-mipsel
sudo ./clean_sources
sudo ./build_toolchain
</pre>

编译3.0内核的固件

如果你需要编译3.0内核的固件就需要执行下面的步骤，默认支持3.4内核
<pre>

cd /opt/rt-n56u/toolchain-mipsel
sudo ./clean_sources
sudo ./build_toolchain_3.0.x
</pre>

编译好后的工具链会放在/opt/rt-n56u/toolchain-mipsel/toolchain-3.0.x

以后在更新或者升级工具链的情况下才需要执行以上步骤。

机型适配（newifi mini）

进行入固件源码目录

<pre>
cd /opt/rt-n56u/trunk
</pre>

生成配置文件

删除默认的配置文件 opt/rt-n56u/trunk下的.config文件，默认是隐藏的，可以使用Ctrl+H显示与隐藏配置文件，或者用以下命令进行删除

<pre>
sudo rm -f opt/rt-n56u/trunk/.config
</pre>
从opt/rt-n56u/trunk/configs/templates里拷贝出 ac54u_base.config到 opt/rt-n56u/trunk下并重命名成.config，或者用以下命令进行操作

<pre>
sudo cp opt/rt-n56u/trunk/configs/templates/ac54_base.config /opt/rt-n56u/trunk/.config
</pre>
修改配置文件

由于修改后默认的.config 配置文件里文档写的很详细，就不多解释，根据自己需要开启与关闭需要与不需要的功能，#这个符号默认是注释代码，也就是关闭某个功能，反之去掉就是开启（可以把文档内容拷出来到翻译里翻译一遍，可以很清楚的看到每个功能是干什么用的）

适配灯与复位键

编辑 /opt/rt-56u/trunk/configs/boards/RT-AC54U/board.h
<pre>
/* ASUS RT-AC54U */

#define BOARD_PID		"RT-AC54U"
#define BOARD_NAME		"RT-AC54U"
#define BOARD_DESC		"ASUS RT-AC54U Wireless Router"
#define BOARD_VENDOR_NAME	"ASUSTek Computer Inc."
#define BOARD_VENDOR_URL	"http://www.asus.com/"
#define BOARD_MODEL_URL		"http://www.asus.com/Networking/RTAC54U/"
#define BOARD_BOOT_TIME		25
#define BOARD_FLASH_TIME	120
#undef BOARD_GPIO_BTN_RESET
#define BOARD_GPIO_BTN_WPS	11
#undef  BOARD_GPIO_BTN_WLTOG
#undef BOARD_GPIO_LED_ALL
#define BOARD_GPIO_LED_WIFI	72
#undef  BOARD_GPIO_LED_SW2G
#define BOARD_GPIO_LED_SW5G	50
#define BOARD_GPIO_LED_POWER	9
#define BOARD_GPIO_LED_LAN	55
#define BOARD_GPIO_LED_WAN	51
#define BOARD_GPIO_LED_USB	52
#undef  BOARD_GPIO_LED_ROUTER
#undef  BOARD_GPIO_PWR_USB
#define BOARD_HAS_5G_11AC	1
#define BOARD_NUM_ANT_5G_TX	2
#define BOARD_NUM_ANT_5G_RX	2
#define BOARD_NUM_ANT_2G_TX	2
#define BOARD_NUM_ANT_2G_RX	2
#define BOARD_NUM_ETH_LEDS	1
#define BOARD_HAS_EPHY_L1000	0
#define BOARD_HAS_EPHY_W1000	0
</pre>
//上面是newifi mini的，由于每个机型不同，仅供参考
调用原厂无线参数

编辑 /opt/rt-56u/trunk/configs/boards/RT-AC54U/kernel-3.4.x.config查找修改成以下代码，并删除目录下SingleSKU开头的文件。
<pre>
CONFIG_RT_SINGLE_SKU=n
CONFIG_RT2860V2_AP_GREENAP=y
CONFIG_INTERNAL_PA_INTERNAL_LNA=n
CONFIG_INTERNAL_PA_EXTERNAL_LNA=n
CONFIG_EXTERNAL_PA_EXTERNAL_LNA=y
CONFIG_MT76X2_AP_GREENAP=y
CONFIG_MT76X2_AP_INTERNAL_PA_INTERNAL_LNA=n
CONFIG_MT76X2_AP_INTERNAL_PA_EXTERNAL_LNA=n
CONFIG_MT76X2_AP_EXTERNAL_PA_EXTERNAL_LNA=y
CONFIG_RALINK_UART_BRATE=57600
</pre>
适配WAN口与LAN口

编辑 /opt/rt-56u/trunk/configs/boards/RT-AC54U/kernel-3.4.x.config
<pre>
CONFIG_RAETH_ESW_PORT_WAN=4
CONFIG_RAETH_ESW_PORT_LAN1=1
CONFIG_RAETH_ESW_PORT_LAN2=0
CONFIG_RAETH_ESW_PORT_LAN3=3
CONFIG_RAETH_ESW_PORT_LAN4=2
</pre>
//上面是newifi mini的，由于每个机型不同，仅供参考
开启外置PA&LAN支持

在 /opt/rt-56u/trunk/configs/boards/RT-AC54U/kernel-3.4.x.config里删除# CONFIG_EXTERNAL_PA_EXTERNAL_LNA is not set 这一行代码，并加入下面的代码

<pre>
CONFIG_EXTERNAL_PA_EXTERNAL_LNA=y
</pre>
扩展内存

编辑/opt/rt-56u/trunk/configs/boards/RT-AC54U/kernel-3.4.x.config配置文件，以AC-54U为例，默认是64M，所以只需要将以下代码进行如下修改
<pre>
// 没修改前是64M内存
CONFIG_RT2880_DRAM_16M is not set
CONFIG_RT2880_DRAM_32M is not set
CONFIG_RT2880_DRAM_64M=y
CONFIG_RT2880_DRAM_128M is not set
CONFIG_RT2880_DRAM_256M is not set
CONFIG_RALINK_RAM_SIZE=64

// 修改成128M
CONFIG_RT2880_DRAM_16M is not set
CONFIG_RT2880_DRAM_32M is not set
CONFIG_RT2880_DRAM_64M is not set
CONFIG_RT2880_DRAM_128M=y
CONFIG_RT2880_DRAM_256M is not set
CONFIG_RALINK_RAM_SIZE=128

</pre>
设置时区

编辑/opt/rt-56u/trunk/user/shared/defaults.h

<pre>
#define DEF_TIMEZONE		"CST-8"
#define DEF_NTP_SERVER0		"ntp1.aliyun.com"
</pre>
关闭 telnet 开启 ssh

编辑/opt/rt-56u/trunk/user/shared/defaults.c
<pre>
{"telnetd","0"}.
{"sshd_enable","1"}
</pre>
汉化

编辑trunk/user/www/dict/EN.header文件，找到LANG_EN=English修改成以下

<pre>
LANG_EN=简体中文
</pre>
编辑trunk/user/www/dict/EN.footer文件，把里面内容翻译成中文（也可以从已经汉化的固件里提取/www/EN.dict文件，删除以下的内容，然后保存并重新命名成EN.footer；
<pre>
[Language type]
LANG_EN=简体中文
LANG_RU=English
</pre>
编辑trunk/user/www/Makefile文件，找到echo "LANG_RU=Pусский" >> $(ROMFS_DIR)/www/EN.header修改成以下

<pre>
echo "LANG_RU=English" >> $(ROMFS_DIR)/www/EN.header
</pre>
编辑trunk/user/www/dict/RU.dict文件，找到LANG_EN=English修改成以下

<pre>
LANG_EN=简体中文
</pre>
生成固件

清除源码树

<pre>
sudo ./clear_tree
</pre>
开始编译生成固件

<pre>
sudo ./build_firmware
</pre>
编译完成后生成的固件在/opt/rt-56u/trunk/images里面

更新源码

转到/opt目录，克隆最新源码

<pre>
cd /opt
sudo git pull
</pre>
如果你对本地存仓库进行了更改，那么在更新源码的时候，你必须运行以下命令

<pre>
sudo git stash
sudo git pull
</pre>
如果更改了工具链，则必须重新构建它
<pre>
cd /opt/rt-n56u/toolchain-mipsel
sudo ./clean_sources  
sudo ./clean_toolchain  
sudo ./build_toolchain
</pre>
以上的修改建议使用手工进行修改，如果直接使用文件进行覆盖的话很容易出错。而且一但编译出错的话需要花很多时间进行调整，所以还是自己一步步慢慢改吧。

更多请参考Padavan官网编译教程
