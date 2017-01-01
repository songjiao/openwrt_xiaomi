用shadowsocks+dnsmasq自动翻墙
===================

原理
===
    根据被墙的域名列表gfwlist,在dnsmasq里配置上游dns为境外dns服务器，且非53标准端口。这样可以解决dns污染的问题。同时经过dnsmasq解析的被墙的域名列表的
    IP会加到ipset gfwlist里。然后在iptables里配置gfwlist里的IP走shadowsocks出去。
    
步骤
===
http://vvsage.com/openwrtshi-xian-tou-ming-fan-qiang-yi-ji-jia-su/ 
1.基本原理

ssh到路由器，看看路由器上有没有安装ss，

which ss-redir  
#/usr/bin/ss-redir 
which ss-tunnel  
#/usr/bin/ss-tunnel
如果有打印出来，那么说明这个版本的rom已经包含了shadowsocks-libev， 如果没有，可以在这里下载openwrt版本的ss，这里面包含3个编译好的可执行文件ss-local,ss-redir,ss-tunnel,由于咱们想实现透明翻墙，ss-redir直接支持iptables的转发，基本原理是配置一个gfwlist的ipset表，通过iptables加载这个list，处于list内的网站通过ss-tunnel或其他方式完成DNS解析之后，直接把流量转发到ss-redir，实现透明翻墙。好，我们一步一步来。

2.在你的VPS上部署socks的服务端

这个步骤略过，直接参考socks的作者写的文档

3.在路由器上部署socks的客户端

安装上面提到的shadowsocks-libev,配置/etc/shadowsocks.json

{
  "server":"x.x.x.x",   
  "server_port": 5555, //服务端shadowsocks服务器的端口
  "local_address":"127.0.0.1",
  "local_port":1081, //本地shadows绑定的端口, 我使用1081
  "password":"PASSWORD",//shdowsocks 密码
  "timeout":600, //不用改
  "method":"aes-256-cfb"
}
保存，启动服务

#启动ss-redir
/usr/bin/ss-redir -c /etc/shadowsocks.json -b 0.0.0.0 -f /var/run/ss-redir.pid

#启动ss-tunnel
/usr/bin/ss-tunnel -c /etc/shadowsocks.json -b 0.0.0.0 -u -l $TUNNEL_PORT -L $DNS -f /var/run/ss-tunnel.pid

注意tunnel服务实质就是拿来做DNS解析的，如果你准备使用别人的DNS服务，则把$TUNNEL_PORT和$DNS填好就行，如果你准备使用我的方法，实际上没必要启动tunnel服务
-u是一定要加的，DNS请求都是走的UDP协议
至此服务已经启动完毕，如果你想杀掉进程直接netstat然后grep对应的端口号，找到对应的PID kill掉就好

当然写个shell脚本来做这件事也是可以的

当然现在还并没有鸟用，因为流量还没有转发到1081这个端口，另外DNS污染的问题也还没有解决，下面我们继续

4.配置自己的DNS域名解析服务

如果你使用别人的域名解析服务，则可以略过，自己弄也非常简单，这里我使用的是dnsmasq，连上VPS，配置/etc/dnsmasq.conf,上游dns server填写8.8.8.8就好，注意一定不能开53的端口，不然会被GFW干的很惨！

port=543  
server=8.8.8.8  
server=8.8.8.4  
在自己的机器上dig一把，如果能正确返回则证明域名解析服务已经ok

dig @X.X.X.X -p 543 www.youtube.com  
5.生成gfwlist黑名单

前面说到流量转发的问题，首先我们需要知道哪些流量需要转发，这就需要一个被墙网站的黑名单了

https://github.com/cokebar/gfwlist2dnsmasq

使用python跑一下就好，像我采用自己搭建DNS服务这种情况，把mydnsip,mydnsport改成自己的就好，注意dnsport不能使用53，不然会被GFW干的很惨。

把生成好的dnsmasq_list.conf文件scp到/etc/dnsmasq.d目录，这个步骤就算告一段落了。

（如果/etc/dnsmasq.conf没有inclued dnsmasq.d这个目录，自己修改下就好,conf-dir=/etc/dnsmasq.d）

6.路由器上配置iptables,实现透明翻墙

vim /etc/firewall.user，在末尾追加两句

ipset -N gfwlist iphash  
iptables -t nat -A PREROUTING -p tcp -m set --match-set gfwlist dst -j REDIRECT --to-port 1081
match gfwlist的流量会转到1081端口，就是咱们的socks服务了，大功告成

7.重启dnsmasq和firewall

/etc/init.d/dnsmasq restart
/etc/init.d/firewall restart
如果一切都没有问题，现在任何连上你路由器的设备都已经可以科学上网了，注意某些机器可能存在DNS缓存，比如iphone，断开网络连接或者重启一下就会清掉，其他设备flushdns一下就好

本来该结束了，但是
国内普遍出口的国际带宽都很窄，目前看来最厚道的是移动，像电信这种情况，不管你家是50M还是100M光宽带，国际带宽都是1M小水管，看个youtube基本也就360P这种清晰度，有了kcp协议，官方保守估计能以比TCP浪费10%-20%的带宽的代价，换取平均延迟降低 30%-40%，且最大延迟降低三倍的传输效果，小水管看1080P不是梦

有关KCP协议的C语言实现

https://github.com/skywind3000/kcp

使用kcp的工具kcptun,提供了各种平台的实现，非常给力（ps:小米路由器(rd2)使用的是arm架构）

https://github.com/xtaci/kcptun

这里就不再啰嗦了(手累)，文档上已经写得非常清楚，部署好kcptun，才是性福哦不幸福生活真正开始的时候！！
