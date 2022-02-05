## 对此软件一些关键信息的记录

### 编译

```shell
sudo apt-get install libssl-dev
./build-pkg.sh --platform openwrt --arch x86-64 --static
```

### 我的配置

#### smartdns.conf

smartdns.conf时smartdns-luci的web界面自动生产的，在基础设置上只勾选启用，填上端口号即可。最终的配置会被写到/var/etc/smartdns/smartdns.conf文件中。

```
server-name smartdns
bind :6053
rr-ttl-min 300
log-size 64K
log-num 1
log-level error
conf-file /etc/smartdns/address.conf
conf-file /etc/smartdns/blacklist-ip.conf
conf-file /etc/smartdns/custom.conf
```

#### custom.conf

custom.conf填入自定义设置框中，会被写入/etc/smartdns/custom.conf文件中

```
# set log level
# log-level [level], level=fatal, error, warn, notice, info, debug
log-level debug

# log-size k,m,g
log-size 1000m

log-file /etc/smartdns/smartdns.log
# log-num 2

prefetch-domain yes
cache-size 200000
rr-ttl-min 300
rr-ttl-max 600
cache-persist yes
cache-file /etc/smartdns/smartdns.cache

serve-expired yes
serve-expired-ttl 1800

# 关闭 IPv6 解析 和 双栈优选
force-AAAA-SOA yes
dualstack-ip-selection no

# 国内 DNS
bind-tcp :6153 -group cn speed-check-mode ping,tcp:80
bind :6153 -group cn speed-check-mode ping,tcp:80

server 124.207.160.106 -group cn -group bootstrap
server 223.5.5.5 -group cn -group bootstrap
server 119.29.29.29 -group cn -group bootstrap
server 180.76.76.76 -group cn -group bootstrap
server 114.114.114.114 -group cn -group bootstrap

# 国外DNS
bind-tcp :6253 -group overseas -no-speed-check
bind :6253 -group overseas -no-speed-check

server-tls 8.8.4.4 -group overseas -exclude-default-group
server-tls 1.0.0.1 -group overseas -exclude-default-group
server-tls 101.101.101.101 -group overseas -exclude-default-group
```

### 原理

在smartdns中创建了两个dns服务器，一个国内，一个国外。国内dns服务器配置为dnsmasq的上游，国外配置为passwall的自定义dns。因为dns请求会先到passwall，判断域名是否在GFW列表中，如果在，请求会被转发到passwall的自定义dns（已经配置为smartdns的国外dns服务了）；否则，请求会被转到dnsmasq，最终会转发到smartdns的国内dns服务上。这样就实现了国内外dns的分流。在这种模式下，passwall的工作模式最好选择GFW列表模式。

### 注意事项

1. 国外dns服务器的ip必须填到强制代理列表里
2. passwall的工作模式最好选择GFW列表模式，这样passwall来做国内外dns分组的事
3. 国外的dns服务器配置为不测试，因为测速的ping包不会走代理，网络很有可能是不通的