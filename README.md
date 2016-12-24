小米mini路由器刷openwrt
=======================

大致步骤为:
* 将小米路由器刷到开发版的0.4.36
* 获取ssh权限
* 把pandoraBox刷到小米路由器


将小米路由器刷到开发版的0.4.36
==============================
#	准备一个U盘，格式化成fat32格式，拷贝miwifi_r1cm_all_46ed1_0.4.36.bin 到U盘，并重命名为miwifi.bin
#	将U盘插到小米路由器
#	拿牙签捅reset按钮直到黄灯闪烁

获取ssh权限
==========
#	拿手机下载小米路由器客户端，安装并注册小米账号
#	用手机连上小米路由器的wifi(这里需要提前配置好小米路由器并且能正常上网),绑定小米路由器
#	用电脑浏览器打开https://d.miwifi.com/rom/ssh 下载对应的miwifi-ssh.bin 并记住密码
#	将miwifi-ssh.bin拷贝到U盘，其他东西都删掉，U盘里只留着一个文件
#	将U盘插到小米路由器，再拿牙签捅一次reset按钮直到黄灯闪烁
#	路由器重启完后就可以直接在命令行ssh root@192.168.31.1 了

把pandoraBox刷到小米路由器    
=========================
#	scp PandoraBox-ralink-mt7620-xiaomi-mini-squashfs-sysupgrade-r512-20150309_with_shadowsocks.bin root@192.168.31.1:/tmp/
#	ssh root@192.168.31.1
#	cd /tmp/
#	mtr -r write PandoraBox-ralink-mt7620-xiaomi-mini-squashfs-sysupgrade-r512-20150309_with_shadowsocks.bin firmware
#	路由器重启完后可以搜索到一个PandoraBox的wifi热点，连上后在浏览器里打开http://192.168.1.1 用户名是root 密码是admin




