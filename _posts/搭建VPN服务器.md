---
title: 在VPS上搭建Android/iOS双平台可用的梯子的解决方案
date: 2016-10-14 15:00:00
cover: "2.jpg"
category: "Linux"
tags:
    - Linux
    - OS
---

## 搭建双平台可用的梯子
由于众所周知的原因，本来是爬墙利器的Shadowsocks于iOS的AppStore中被下架。一直找不到什么好用的替代品，幸而Cisco Anyconnect这个很早以前就用过的VPN解决方案仍然可以使用，并且在Android和iOS上都可以使用，因此我准备以此为切入点，搭建自己的Anyconnect服务器。

### 准备工作和踩坑预警
我准备使用的是OpenConnect Server，它是一款开源的VPN服务器，能够很好的支持Cisco Anyconnect。最主要的是它并没有像OpenVPN那样被挂城墙。但是要使用它同样也有一些缺点：
1. 速度比较慢
2. 搭建配置比较麻烦——主要是依赖问题

首先要提及一下系统平台的问题。  
本来我是Centos6，因为提供商的一键搭建Shadowsocks服务器的脚本只支持Centos6，我图方便一开始就没有换。但是开始搭建OCserv之后，发现Centos6这个系统实在是有年代了。很多依赖的库版本都非常老旧，比如下面要用到的GnuTLS只有2.8.x，而Ocserv依赖3.X。当然这也不是绝对的，我们且做且分析。

平台基本信息：  
RAM：512MB  
OS：Ubuntu 16.04 x86_64  
不管你是用Xshell还是PuTTY，总之最好还是要做好ssh登录，调试过程中免不了要断开重启等操作，每一次登录都很麻烦的话会用光耐心的。

<!--more-->

### 安装依赖和OCserv
按照惯例首先
```
sudo apt-get update
sudo apt-get upgrade
```
更新一下。
然后安装相关依赖：
```
sudo apt-get install build-essential pkg-config libgnutls28-dev libreadline-dev libseccomp-dev libwrap0-dev libnl-nf-3-dev liblz4-dev
```
当然其中有些是用来编译安装OCServ所要用到的，我安装的时候并没有自己编译，但是为了保险起见还是安装完全为好。
然后安装 OCServ，ubuntu的源中是可以找到这个软件包的，版本是0.11.x。可以正常使用。
```
sudo apt-get install ocserv
```
安装结束之后可以在`/etc/ocserv/`下找到`ocserv.conf`配置文件，我们记住这个配置文件的位置，一会要对它进行修改。

### 添加证书与配置OCServ
我们常用的Anyconnect连接OCServ有两种方式，一种是通过用户密码登录，一种是通过证书。证书其实类似于公钥秘钥，证书一致的两端连接时可以免去输入用户名密码的麻烦。最终我们总会用到这种方式，因此首先先把证书生成好。

#### 生成CA证书
生成证书需要用到`certtool`命令，它由`gnutls-bin`这个软件包提供。因此要安装它：
```
sudo apt-get install gnutls-bin
```
接着我们在home下创建一个文件夹ocserv-prep，在这个文件夹里面准备好所有用到的东西。
首先创建CA模板，一会要用它来生成CA证书。
```
cd ocserv-prep
vim ca.tmpl
```
输入如下内容：
>cn = "Your CA name"
organization = "Your fancy name"
serial = 1
expiration_days = 3650
ca
signing_key
cert_signing_key
crl_signing_key

这里的"Your CA name"可以随便填写。
然后生成CA秘钥
```
certtool --generate-privkey --outfile ca-key.pem
```
接着用秘钥生成CA证书
```
certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem
```
最后把这个证书放在ssl秘钥文件夹下：
```
sudo cp ca-cert.pem /etc/ssl/private/my-ca-cert.pem
```
#### 生成Server证书
跟上面的过程类似，首先要创建一个模板server.tmpl：
>cn = "Your hostname or IP"
organization = "Your fancy name"
expiration_days = 3650
signing_key
encryption_key
tls_www_server

**这里的"Your host name or IP"就必须要填写你的服务器ip地址了。**
然后生成秘钥和证书：
```
certtool --generate-privkey --outfile server-key.pem
certtool --generate-certificate --load-privkey server-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template server.tmpl --outfile server-cert.pem
```
最后将证书和秘钥复制到ssl秘钥文件夹下：
```
sudo cp server-cert.pem /etc/ssl/private/my-server-cert.pem
sudo cp server-key.pem /etc/ssl/private/my-server-key.pem
```
#### 配置OCServ
还记得刚刚说过的那个`ocserv.conf`吗？现在我们生成好了各种秘钥和证书，要使用它，就需要配置OCserv的启动选项。
```
vim /etc/ocserv/ocserv.conf
```
会发现有非常多的注释内容，其实你可以把它们都删掉，填入如下内容：
{% codeblock lang:bash %}
#登录验证方式 后面要在这个位置创建一个用户
auth = "plain[/etc/ocserv/ocpasswd]"
#auth = "certificate"

#设置TCP UDP端口
tcp-port = 1600
udp-port = 1600

#设置socket文件位置
socket-file = /var/run/ocserv-socket

#设置服务端证书秘钥
server-cert = /etc/ssl/private/my-server-cert.pem
server-key = /etc/ssl/private/my-server-key.pem

#设置用户证书秘钥
ca-cert = /etc/ssl/private/my-ca-cert.pem

#设置用户数和连接数
max-clients = 10
max-same-clients = 2

#使服务器兼容Cisco Anyconnect客户端
cisco-client-compat = true

#设置OID为2.5.4.3不用动
cert-user-oid = 2.5.4.3

#客户端ip地址段 必需
ipv4-network = 192.168.1.0
ipv4-netmask = 255.255.255.0

#为代理指定使用的dns
dns = 8.8.8.8
dns = 8.8.4.4

# 注释掉所有的route，让服务器成为gateway
#route = 192.168.1.0/255.255.255.0

#其他设置 仅供参考
keepalive = 86400
mobile-dpd = 1800
isolate-workers = true
try-mtu-discovery = true
min-reauth-time = 3
auth-timeout = 40
tls-priorities = "NORMAL:%SERVER_PRECEDENCE:%COMPAT:-VERS-SSL3.0"
max-ban-score = 50
ban-reset-time = 300
cookie-timeout = 300
cookie-rekey-time = 14400
deny-roaming = false
rekey-time = 172800
rekey-method = ssl
use-utmp = true
use-occtl = true
pid-file = /var/run/ocserv.pid
device = vpns
predictable-ips = true
default-domain = example.com
ping-leases = false
{% endcodeblock %}
最后在刚才确定的位置创建一个用户和密码
```
sudo ocpasswd -c /etc/ocserv/ocpasswd your-username
```
到这里OCServ端的配置就基本完成了，接下来还要对服务器的防火墙进行一些配置。

### Iptables与系统转发配置
这里的设置主要是打开服务所要占用的端口、允许转发、启用NAT等操作。
我们首先建立一个规则文件`ipv4`输入：
{% codeblock lang:bash %}
*filter

# Allow all loopback (lo0) traffic and reject traffic
# to localhost that does not originate from lo0.
-A INPUT -i lo -j ACCEPT

# Allow ICMP
-A INPUT -p icmpv6 -j ACCEPT

# Allow HTTP and HTTPS connections from anywhere
# (the normal ports for web servers).
# -A INPUT -p tcp --dport 80 -m state --state NEW -j ACCEPT
# -A INPUT -p tcp --dport 443 -m state --state NEW -j ACCEPT

# Allow inbound traffic from established connections.
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Log what was incoming but denied (optional but useful).
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "ip6tables_INPUT_denied: " --log-level 7

# Reject all other inbound.
# -A INPUT -j REJECT

# Log any traffic that was sent to you
# for forwarding (optional but useful).
-A FORWARD -m limit --limit 5/min -j LOG --log-prefix "ip6tables_FORWARD_denied: " --log-level 7

# Reject all traffic forwarding.
# -A FORWARD -j REJECT
-A INPUT -p tcp -m state --state NEW --dport 1600 -j ACCEPT
-A INPUT -p udp -m state --state NEW --dport 1600 -j ACCEPT

COMMIT

*nat
-A POSTROUTING -j MASQUERADE
COMMIT
{% endcodeblock %}
这是一份从Linode提供的安全配置修改而来的iptables配置规则。其中我禁用了80和443端口，是因为总有人来用我的服务器转发垃圾邮件，而我最近又没有打算在上面搭建站点，因此暂时禁用了它们。  
设置好iptables之后要应用到系统防火墙上。但ubuntu每次重启的时候，都将重置iptables规则，我们需要另一个工具来持久化iptables规则。
```
sudo apt-get install iptables-persistent
```
安装过程中选择yes即可。
然后把将上面写好的规则应用到系统上：
```
sudo iptables-restore < ipv4
sudo iptables-save
```
最后打开IPv4流量转发：
```
sudo vim /etc/sysctl.conf
```
修改下面这一条为：
```
net.ipv4.ip_forward=1
```
最后刷新配置：
```
sudo sysctl -p /etc/sysctl.conf
```

至此我们的服务器已经搭建完毕。
接着使用
```
ocserv -fd 1
```
来开启服务器。这时你可以在控制台看到运行信息如下：
```
Parsing plain auth method subconfig using legacy format
Setting 'plain' as primary authentication method
listening (TCP) on 0.0.0.0:1900...
listening (UDP) on 0.0.0.0:1900...
ocserv[1569]: main: initialized ocserv 0.10.11
ocserv[1570]: sec-mod: reading supplemental config from files
ocserv[1570]: sec-mod: sec-mod initialized (socket: /var/run/ocserv-socket.1569)
```
你可以通过Anyconnect客户端，填好服务器地址和端口号，使用之前创建的用户名和密码登录到该服务器了。连接时控制台将显示你的连接信息。要想使服务在后台运行，去掉参数`f`即可。

### 使用证书登录
最后为了避免每次都输入密码，我们想要使用证书登录。创建证书模板和生成证书的过程同上面类似。首先创建证书模板`user.tmpl`:
```
cn = "some random name"
unit = "some random unit"
expiration_days = 365
signing_key
tls_www_client
```
生成User密钥和证书：
```
certtool --generate-privkey --outfile user-key.pem
certtool --generate-certificate --load-privkey user-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template user.tmpl --outfile user-cert.pem
```
将证书和密钥转为PKCS12格式，方便移动设备导入
```
certtool --to-p12 --load-privkey user-key.pem --pkcs-cipher 3des-pkcs12 --load-certificate user-cert.pem --outfile user.p12 --outder
```
Android平台可以直接把证书文件发到手机上打开导入。iOS平台就需要建立本地的http服务器，使用URL导入了。
最后修改OCserv的配置文件`/etc/ocserv/ocserv.conf`找到`auth = "certificate"`这一项取消注释，并注释其他的`auth`项，启用证书验证。

至此双平台都可以使用梯子了。Anyconnect的速度还算可以，用手机的4G网可以有1M多的速度。

## 搭建高速Android/PC可用的Shadowsocks梯子
上面所述的OCserv+Anyconnect是为了照顾iOS没有shadowsocks而不得已为之。它采用全局代理模式，建立连接后所有的访问均要通过vpn。并且它只支持Windows。对于常常在Linux/Windows之间切换的同学来说很不方便。
所以在PC上我们可以使用shadowsocks+kcptun的解决方案，来获得更高速的连接。
### 服务端设置
搭建shadowsocks的方法，在项目的wiki中有非常简单的实现方式。
我们现在要使用KCPtun利用kcp协议来加速shadowsocks。
登录到vps后使用一键安装脚本：
```
wget https://raw.githubusercontent.com/kuoruan/kcptun_installer/master/kcptun.sh
chmod +x ./kcptun.sh
./kcptun.sh
```
现在假设你的shadowsocks运行在8388端口上，密码是sspasswd。那么接下来需要设置的有：
1. KCPtun的服务端口：该端口不是shadowsocks端口。它是供KCPtun客户端连接的端口。比如我们设置成8400。
2. 设置加速IP：如果ss服务端就建立在该服务器上，那么默认即可；否则填写服务器ip。
3. 设置加速端口：这里的端口即shadowsocks端口。设置为8388。接下来可能会出现确认是否有软件使用此端口的问题。选择是即可。
4.设置KCPtun密码：最好修改成没有空格的。比如kcptunpasswd。
5.其余选项暂时使用默认即可。如需修改，使用`./kcptun.sh reconfig`修改配置文件。下列配置仅供参考：
> crypt: aes-192  
datashard: 10  
parityshard: 3
conn: 1  
mtu: 1350  
sndwnd: 1024
rcvwnd: 1024
dscp: 0
autoexpire: 60
mode: fast2
keepalive: 10
sockbuf: 4194304

至此服务端的设置已经完成。下面是本地客户端的设置。

### 客户端设置
到[KCPtun作者的github](https://github.com/xtaci/kcptun/releases)上下载最新版本的release，解压到任意位置。再下载[KCPtun客户端管理工具](https://github.com/dfdragon/kcptun_gclient/releases)，打开客户端管理工具，导入刚刚解压的相应版本的kcptun客户端，各项参数按照刚刚设置好的参数配置，最后启动即可。其中基础参数中本地侦听端口是指shadowsocks客户端运行时填写的服务器端口。

KCPtun和shadowsocks的工作模式是，shadowsocks客户端->KCPtun客户端———>kcptun服务端->shadowsocks服务端。由于客户端都在本地，所以shadowsocks客户端中服务器地址和端口应与本机地址127.0.0.1和本地KCPtun客户端的侦听端口一致。比如KCPtun本地侦听端口为1605，则shadowsocks客户端的服务器端口也应为1605。
启动之后可以查看shadowsocks日志查看链接是否畅通。配置无误的话就可以正常使用了。

### Chrome设置
可以开启Shadowsocks系统代理来进行系统级别代理，也可以安装Chrome插件SwitchySharp来启用浏览器级别代理。在不开启系统代理的情况下后台运行Shadowsocks，SwitchySharp配置为SOCKS5代理，地址为127.0.0.1端口为1080（即Shadowsocks客户端中下方的代理端口）。访问需要梯子的网站时开启该情景模式即可。

使用KCPtun加速之后的梯子速度非常给力，看Youtube720p应该是没有压力的。

参考文献：
https://github.com/xtaci/kcptun
http://www.jianshu.com/p/172c38ba6cee
https://bitinn.net/11084/
