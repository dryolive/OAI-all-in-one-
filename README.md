# OAI-all-in-one-
考虑到我们将接入网、核心网的各个网元都在同一个机器上，所以我决定采用最简单方便的all-in-one部署方法，这样用snap即可装载各个网元，然后单独配置各个网元的config文件即可完成接口的匹配。其他的安装方式还有docker安装、源码安装。
## 内核相关处理
考虑到OAI 对内核非常敏感，很多莫名其表的错误都是由内核不适应导致的，所以安装一下低延时内核等模块
### 安装 low-latency kernel（低延时内核）：
```bash
sudo apt-get install linux-lowlatency
sudo apt-get install linux-image-`uname -r | cut -d- -f1-2`-lowlatency
sudo apt-get install linux-headers-`uname -r | cut -d- -f1-2`-lowlatency
sudo reboot
```
### 加载 GTP 内核模块（for OAI-CN）：
```bash
sudo modprobe gtp
dmesg | tail # You should see something that says about GTP kernel module
```
为了让OAI支持接入更多的UE，可能会需要修改CPU相关功能来压榨PC的性能，具体涉及到 在 BIOS 中移除电源管理功能（P-states, C-states）、在 BIOS 中关闭超线程（hyper-threading）、禁用 Intel CPU 的 P-state 驱动（Intel CPU 专用的频率调节器驱动）、将 intel_powerclamp（Intel 电源管理驱动程序）加入启动黑名单、关闭 CPU 睿频，这里暂不使用，需要的话查看https://www.cnblogs.com/jmilkfan-fanguiju/p/12789792.html
## OAI核心网部署
来源：https://gitlab.eurecom.fr/mosaic5g/mosaic5g/-/wikis/tutorials/oai-cn
```bash
# Install OAI-CN as a snap:
sudo snap install oai-cn --channel=edge --devmode

# Check the installation:
sudo oai-cn.help 
```
核心网需要MySQL注册UE列表，所以先安装mysql：
### mysql安装
```bash
$ sudo apt install mysql-server mysql-client
$ sudo mysql_secure_installation # after new installation #root用户先暂停这一步
$ mysql -u root -p
```
注：在第二步中如果是作为root用户,则需要提前设置密码，步骤如下：
 ```bash
 sudo mysql  #进入mysql命令行
 ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password by 'mynewpassword';  #mynewpassword改为自己的密码
 exit
 sudo mysql_secure_installation #回归之前暂停的第二步
 ```
### HSS安装部署
1. 初始化 HSS: `sudo oai-cn.hss-init`
2. 找到 configuration 文件: `sudo oai-cn.hss-conf-get`，屏幕会输出该文件所在的路径
3. 修改 hss.conf 文件，修改登录mysql的用户名（root）和密码（之前自己设置的密码），并修改OPERATOR_key = "11111111111111111111111111111111";并且该文件内会有hss_fd.conf的路径
4. 修改 hss_fd.conf, 检查 Identity 来匹配 <hostname>.openair4G.eur ，在命令行查看`/etc/hosts`文件，可以看到hss和mme有相应的identity。
5. 建立证书：`oai-cn.hss-init`
6. 运行HSS： `sudo oai-cn.hss`
7. 最后一行为"Initializing S6a layer: DONE"即部署成功
 
### MME安装部署
1. 初始化 MME: `sudo oai-cn.mme-init`
2. 找到 configuration 文件: `sudo oai-cn.mme-conf-get`
3. 在 mme.conf: 需要检查一些参数，但是现在版本应该默认就是对的不需要修改，在该文件中找到mme_fd.conf的路径
4. 在 mme_fd.conf: 检查identity 和 connect peer的hostname，但默认也是对的
5. 启动MME：`sudo oai-cn.mme`
