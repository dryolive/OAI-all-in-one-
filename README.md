# OAI-all-in-one-
考虑到我们将接入网、核心网的各个网元都在同一个机器上，所以我决定采用最简单方便的all-in-one部署方法，这样用snap即可装载各个网元，然后单独配置各个网元的config文件即可完成接口的匹配。其他的安装方式还有docker安装、源码安装。
## 内核相关处理
考虑到OAI 对内核非常敏感，很多莫名其表的错误都是由内核不适应导致的，所以安装一下低延时内核等模块
### 安装 low-latency kernel（低延时内核）：
```
sudo apt-get install linux-lowlatency
sudo apt-get install linux-image-`uname -r | cut -d- -f1-2`-lowlatency
sudo apt-get install linux-headers-`uname -r | cut -d- -f1-2`-lowlatency
sudo reboot
```
### 加载 GTP 内核模块（for OAI-CN）：
```
sudo modprobe gtp
dmesg | tail # You should see something that says about GTP kernel module
```
为了让OAI支持接入更多的UE，可能会需要修改CPU相关功能来压榨PC的性能，具体涉及到 在 BIOS 中移除电源管理功能（P-states, C-states）、在 BIOS 中关闭超线程（hyper-threading）、禁用 Intel CPU 的 P-state 驱动（Intel CPU 专用的频率调节器驱动）、将 intel_powerclamp（Intel 电源管理驱动程序）加入启动黑名单、关闭 CPU 睿频，这里暂不使用，需要的话查看https://www.cnblogs.com/jmilkfan-fanguiju/p/12789792.html
##OAI核心网部署
来源：https://gitlab.eurecom.fr/mosaic5g/mosaic5g/-/wikis/tutorials/oai-cn
核心网需要MySQL注册UE列表，所以先安装mysql：
###mysql安装
```
$ sudo apt install mysql-server mysql-client
$ sudo mysql_secure_installation # after new installation #root用户先暂停
$ mysql -u root -p
```
注：在第二步中如果是作为root用户