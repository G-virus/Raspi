# 树莓派Raspi----制作空调遥控器并使用语音(Siri)控制

本篇内容绝大部分参考自此[文档](https://segmentfault.com/a/1190000014135418?utm_source=tag-newest),针对此时最新的Debian树莓派系统做了相应的修改,增加了一些个人经验分享.

[LIRC官网](https://www.lirc.org/html/configuration-guide.html)
[实操教程](https://github.com/AnaviTechnology/anavi-docs/blob/master/anavi-infrared-phat/anavi-infrared-phat.md#setting-up-lirc)



## 准备工作
硬件方面:  
红外接收管（参考型号HS0038B）  
红外发射管（参考型号TSAL6200）  
遥控器（或带有红外的手机）  
面包板、树莓派GPIO延长线(可选)    
面包板和延长线非必须,只是看起来工整,方便接线,推荐使用.
![1.png](https://github.com/G-virus/Raspi/blob/master/images/1.png?raw=true "1.png")

软件方面:
树莓派系统版本 4.19
```shell
uname -a
Linux raspberrypi 4.19.118-v7l+ #1311 SMP Mon Apr 27 14:26:42 BST 2020 armv7l GNU/Linux
```
LIRC版本
```shell
lircd -v
lircd 0.10.1
```

## 安装硬件
![2.png](https://github.com/G-virus/Raspi/blob/master/images/2.png?raw=true "2.png")
按照图示进行安装  
17是发射端
18是接收端
## 软件配置
### 安装LIRC
```shell
sudo apt update
sudo apt install -y vim devscripts dh-exec doxygen expect libasound2-dev libftdi1-dev libsystemd-dev libudev-dev libusb-1.0-0-dev libusb-dev man2html-base portaudio19-dev socat xsltproc python3-yaml dh-python libx11-dev python3-dev python3-setuptools
sudo apt install lirc
```
这里安装lirc可能会出错,先不用管,直接下一步.  
### 启用参数
```shell
sudo cp /etc/lirc/lirc_options.conf.dist /etc/lirc/lirc_options.conf
sudo cp /etc/lirc/lircd.conf.dist /etc/lirc/lircd.conf
```
### 开启i2C
*有人说需要开,有人说不需要,我反正是开了.*  
```shell
sudo raspi-config
```
`Interfacing Options > I2C > enable`
### 修改config.txt
不会用vim的可以用nano,一样
```shell
sudo vim /etc/config.txt

# Uncomment some or all of these to enable the optional hardware interfaces
dtparam=i2c_arm=on
#dtparam=i2s=on
#dtparam=spi=on

# Uncomment this to enable infrared communication.
dtoverlay=gpio-ir,gpio_pin=18
dtoverlay=gpio-ir-tx,gpio_pin=17
```
17是发射端,18是接收端.
###完成设置
lirc0表示发送端也就是17,lirc1表示接收端,接收学习信号.
暂时设置成lirc1接收模式
```shell
sudo vim etc/lirc/lirc_options.conf

driver          = default
device          = /dev/lirc1
```
修复安装
```shell
sudo apt install -y lirc
```
重启树莓派
```shell
reboot
```
###测试接收器是否正常
关闭服务
```shell
sudo systemctl stop lircd
```
打开接收数据
```shell
mode2 -d /dev/lirc1
```
没有报错且出现监听界面,将遥控器对红外接收器随便按
会出现类似
```shell
space 3662230
pulse 2428
space 594
pulse 1201
space 596
pulse 1230
space 595
........
```
说明接收器功能正常,如果不正常可能是配置错误,遥控器问题,也可能接收器元器件损坏.

### 录入红外数据
经过我的测试,一般的空调遥控器在按下一个按钮后,会将此时遥控器上的空调整体信息一并发送给空调(包括此时的温度,模式,风速等等),也就是说一个"数据包"包含着一个"场景".

这样看来是弊大于利,很适合语音控制.

录入数据的目的是将`一个场景`的信息存入内存,需要发送的时候调出,通过红外发射器(GPIO17)发送给空调.但是存储需要特定的格式,而且不同的lirc版本可能存在差异,到目前为止此方法仍然适用.

确保此时的/etc/lirc/lirc_options.conf里面是lirc1,并且关闭了lircd服务.

#### 格式
第一种方法是用我的模板,先创建一个名称为oo的遥控器(目前发现好像一个遥控器只能存放8个场景,具体情况未知)
```shell
sudo touch /etc/lirc/lircd.conf.d/oo.lircd.conf
sudo pcmanfm /etc/lirc/lircd.conf.d/
```
打开文件管理器然后用文本文档打开
格式如下
```

# Please take the time to finish this file as described in
# https://sourceforge.net/p/lirc-remotes/wiki/Checklist/
# and make it available to others by sending it to
# <lirc@bartelmus.de>
#
# This config file was automatically generated
# using lirc-0.10.1(default) on Sun Jun 28 20:59:35 2020
# Command line used: -f -d /dev/lirc1 --disable-namespace
# Kernel version (uname -r): 4.19.118-v7l+
#
# Remote name (as of config file): oo
# Brand of remote device, the thing you hold in your hand:
# Remote device model nr:
# Remote device info url:
# Does remote device has a bundled capture device e. g., a
#     usb dongle? :
# For bundled USB devices: usb vendor id, product id
#     and device string (use dmesg or lsusb):
# Type of device controlled
#     (TV, VCR, Audio, DVD, Satellite, Cable, HTPC, ...) :
# Device(s) controlled by this remote:

begin remote

  name  oo
  flags RAW_CODES
  eps            30
  aeps          100

  gap          19991

      begin raw_codes

          name off_1
             9041     4501      541     1683      547     1675      549      553      539      555      546      560      547      549      560     1664      547     1680      548     1680      551     1675      546     1684      554     1669      540      559      548      550      546      552      555     1670      550      549      552      546      559      542      552      546      536      562      541      564      539      552      555      548      540      555      554      538      557      549      556      545      521      576      546      548      554      549      549      553      550      549      543      553      555      542      559      539      565      535      574     1664      545      549      550      546      552      549      541      558      546      552      546      552      550      548      521      579      549     1686      544      543      557      543      552      549      557      546      549      540      571      531      546     1684      555      546      545      551      544      555      547      550      562      538      548      549      560      541      565      535      552      548      558      542      547      553      553      546      553      546      551      573      520      539      562      545      546      554      554      542      562      540      558      545      560      541      534      565      546      553      547      551      553      547      548      552      555      546      548      549      556      544      553      547      541      556      547      557      546      548      552      549      544     1680      550      548      552     1675      555      538      549      557      545      548      564      540      549      550      548     1677      552     1676      550     1673      546      556      558     1668      547      550      546     1679      558     1670      575   130160

          end raw_codes
```
**注意格式!!!**,保存.


第二种方法我怎么尝试都失败,可能只适合新式遥控器(我猜的).

```shell
irrecord -d /dev/lirc1/ /etc/lirc/lircd.conf.d/ --disable-namespace
```
进入自助服务,根据提示最后会在/etc/lirc/lircd.conf.d/找到此文件,如果名称不对,改成xx.lircd.conf的形式,xx为名
字,然后缺数值的地方改成上面的数字.
#### 录入
在有了格式之后,需要替换上面绿色的数字,这个就是红外包含的信息.依然是lirc1,并且关闭了lircd服务
```shell
mode2 -m -d /dev/lirc1
```
这行代码将会进入监听模式而且列表形式呈现,确保周围没有杂音,在按下遥控器一个(比如关机)按钮后出现类似如下界面
```
16777215

     9059     4432      706     1604      706      528
      679      524      681     1603      703      526
      680     1602      715     1596      704      526
      679      527      679      527      680      527
      679     1604      705      530      673      530
      674      529      682      529      675      530
      674      532      674      532      650      557
      648      556      654     1653      676      533
      649      559      647     1667      639      559
      648      558      656      553      647     1658
      648      558      650     1659      649      559
      647      559      648     1659      648      558
      646    19991pause
```
此时去掉第一个最大值,去掉pause英文.将剩下的数字复制到遥控器格式文本中替换之前我的数据.
我这里只添加了一条名称为off_1的`场景`信息,可以添加多条.值得注意的是现在看到的是列表,但在文件夹中是一行一个场景,不换行.

![oo.png](https://github.com/G-virus/Raspi/blob/master/images/oo.png?raw=true "oo.png")

### 发射测试
切换到红外发射器
```shell
sudo vim etc/lirc/lirc_options.conf

driver          = default
device          = /dev/lirc0
```
```shell
sudo systemctl start lircd
irsend LIST oo ""
```
![list.png](https://github.com/G-virus/Raspi/blob/master/images/list.png?raw=true "list.png")
打开服务,然后看看有没有录入的信息加载到oo中.
如果没成功,就是格式问题或者录入太多条目,继续修改之前的录入格式.
如果成功,看看能不能把空调关掉吧
```shell
irsend SEND_ONCE oo off_1
```
### 优化
理论上一次发射信号空调就可以收到,但是不确定是红外参数不匹配还是什么原因,有时候总会有丢包,所以我就憨憨的用多次录入,多次发送解决这种情况发生...并且制作了个Python脚本.
```shell
vim /home/pi/Documents/remote.py
```
```Python
import os
import sys
import time
tem = sys.argv[1]
a = 1
if tem == "25C":
    while(a % 6 !=0) :
        os.system('irsend SEND_ONCE oo 25_1C')
        os.system('irsend SEND_ONCE oo 25_2C')
        os.system('irsend SEND_ONCE oo 25_3C')
        a+=1
if tem == "23C":
    while(a % 6 != 0):
        os.system('irsend SEND_ONCE oo 23_1C')
        os.system('irsend SEND_ONCE oo 23_2C')
        a+=1
if tem == "20C":
    while(a % 6 != 0):
        os.system('irsend SEND_ONCE oo 20_1C')
        os.system('irsend SEND_ONCE oo 20_2C')
        a+=1
if tem == "off":
    while(a % 6 !=0):
        os.system('irsend SEND_ONCE oo off_2')
        time.sleep(2)
        os.system('irsend SEND_ONCE oo off_1') 
        a+=1

```
我把常用的25度低风录入了三遍,这样就可以确保99.99999%执行成功.
## 语音控制----以Siri为例
首先树莓派开启ssh服务
之后获取树莓派的ip
```shell
ifconfig
```
或者
推荐使用内外穿透,b站看个视频教学几个小时就可以搞定,云服务器最便宜的最低配置也可以用,这样可以使遥控空调的价值最大化.

### IPhone自动化
打开iPhone,在快捷指令中添加自动化,依次选择 当到达 执行 通过SSH运行脚本,再选取距离家多少米的位置执行.输入树莓派的ip 密码等等.
之后在健身完(shangban)回家之前可以自动开启空调,妈妈再也不会担心我回来热的满头大汉浑身难受了.
![auto1.png](https://github.com/G-virus/Raspi/blob/master/images/auto1.png?raw=true "auto1.png")
![auto2.png](https://github.com/G-virus/Raspi/blob/master/images/auto2.png?raw=true "auto2.png")
![auto3.png](https://github.com/G-virus/Raspi/blob/master/images/auto3.png?raw=true "auto3.png")
### Siri控制
只需要快捷指令添加SSH脚本,内容和上述一样,只不过Siri很呆,不仅反应迟钝而且多一个少一个字都识别不了,以后弄了HA什么Homekit再说.
![siri.png](https://github.com/G-virus/Raspi/blob/master/images/siri.PNG?raw=true "siri.png")

其他手机一样,原理就是ssh.


