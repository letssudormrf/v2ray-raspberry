# v2ray-raspberry
Installing v2ray on raspberry pi 3B

### Raspberry Pi 3B
#### Login to raspberry pi 3B via SSH
    ssh root@192.168.1.1
#### Update OpenWRT repos by opkg command
    opkg update
#### Install Package(ca-certificates/curl/unzip) for downloading v2ray binary
    opkg install ca-certificates curl unzip iptables-mod-tproxy
#### Download and Unzip the v2ray archive file
    export V2_GIT_PATH="https://github.com/v2ray/v2ray-core"
    export V2_VERSION="latest"
    export VER=$(curl --silent https://api.github.com/repos/${V2_GIT_PATH#**//*/}/releases/${V2_VERSION} | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')
    curl -L -H "Cache-Control: no-cache" -o /tmp/v2.zip ${V2_GIT_PATH}/releases/download/$VER/v2ray-linux-arm64.zip
    unzip /tmp/v2.zip -d /tmp/v2/
#### Copy v2ray binary to bin path
    mkdir -p /usr/bin/v2ray/
    cp /tmp/v2/v2ray /tmp/v2/v2ctl /tmp/v2/geoip.dat /tmp/v2/geosite.dat /usr/bin/v2ray/
    chmod -R a+x /usr/bin/v2ray/
    
#### Create v2ray config file
    mkdir -p /etc/v2ray/
```
cat <<'EOF' > /etc/v2ray/config.json
{
  "log": {
    "loglevel": "warning"
  },
  "inbound": {
    "listen": "0.0.0.0",
    "port": 1080,
    "protocol": "socks",
    "settings": {
      "auth": "noauth",
      "udp": true
    },
    "domainOverride": [
      "http",
      "tls"
    ]
  },
  "outbound": {
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
          "address": "www.example.com",
          "port": 443,
          "users": [
            {
              "id": "117ff1a7-d810-4ec7-b368-6fc4491a4435",
              "alterId": 0,
              "security": "none",
              "level": 1
            }
          ]
        }
      ]
    },
    "tag": "proxy",
    "streamSettings": {
      "network": "ws",
      "security": "tls",
      "tlsSettings": {
        "serverName": "www.example.com",
        "allowInsecure": true
      },
      "tcpSettings": {
        "header": {
          "type": "none",
          "request": {}
        }
      },
      "kcpSettings": {
        "mtu": 1350,
        "tti": 50,
        "uplinkCapacity": 5,
        "downlinkCapacity": 20,
        "congestion": false,
        "readBufferSize": 2,
        "writeBufferSize": 2,
        "header": {
          "type": "none"
        }
      },
      "wsSettings": {
        "path": "/v2/",
        "headers": {
          "Host": "www.example.com"
        }
      },
      "sockopt": {
        "mark": 255,
        "tcpFastOpen": false,
        "tproxy": "redirect"
      }
    },
    "mux": {
      "enabled": false,
      "concurrency": 8
    }
  },
  "inboundDetour": [
    {
      "domainOverride": ["tls","http"],
      "port": 1088,
      "protocol": "dokodemo-door",
      "settings": {
        "network": "tcp,udp",
        "followRedirect": true 
     }
   }
  ],
  "outboundDetour": [
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct",
      "sockopt": {
        "mark": 255,
        "tcpFastOpen": false,
        "tproxy": "redirect"
      }
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "block"
    }
  ],
  "routing": {
    "strategy": "rules",
    "settings": {
      "domainStrategy": "IPIfNonMatch",
      "rules": [
        {
          "type": "field",
          "ip": [
            "geoip:private",
            "geoip:cn"
          ],
          "outboundTag": "direct"
        }
      ]
    }
  }
}
EOF
```

#### Create init.d file for starting up
```
cat <<'EOF' > /etc/init.d/v2ray
#!/bin/sh /etc/rc.common
#
# Copyright (C) 2017 Ian Li <OpenSource@ianli.xyz>
#
# This is free software, licensed under the GNU General Public License v3.
# See /LICENSE for more information.
#

START=90

USE_PROCD=1

start_service() {
        mkdir /var/log/v2ray > /dev/null 2>&1
        procd_open_instance
        procd_set_param respawn
        procd_set_param env V2RAY_RAY_BUFFER_SIZE="500" V2RAY_VMESS_PADDING="1"
        procd_set_param limits core="unlimited" nofile="92963 92963"
        procd_set_param command /usr/bin/v2ray/v2ray -config /etc/v2ray/config.json
        procd_set_param file /etc/v2ray/config.json
        procd_set_param stdout 1
        procd_set_param stderr 1
        procd_set_param pidfile /var/run/v2ray.pid
        procd_close_instance
}
EOF
```
    chmod a+x /etc/init.d/v2ray
    /etc/init.d/v2ray enable
    /etc/init.d/v2ray stop
    /etc/init.d/v2ray start
#### Insert iptables rules
```
cat <<'EOF' > /etc/firewall.user
# This file is interpreted as shell script.
# Put your custom iptables rules here, they will
# be executed with each firewall (re-)start.

# Internal uci firewall chains are flushed and recreated on reload, so
# put custom rules into the root chains e.g. INPUT or FORWARD or into the
# special user chains, e.g. input_wan_rule or postrouting_lan_rule.
iptables -t nat -N V2RAY
iptables -t nat -A V2RAY -d 0/8 -j RETURN
iptables -t nat -A V2RAY -d 127/8 -j RETURN
iptables -t nat -A V2RAY -d 10/8 -j RETURN
iptables -t nat -A V2RAY -d 169.254/16 -j RETURN
iptables -t nat -A V2RAY -d 172.16/12 -j RETURN
iptables -t nat -A V2RAY -d 192.168/16 -j RETURN
iptables -t nat -A V2RAY -d 224/4 -j RETURN
iptables -t nat -A V2RAY -d 240/4 -j RETURN
iptables -t nat -A V2RAY -p tcp -j RETURN -m mark --mark 0xff
iptables -t nat -A V2RAY -p tcp -j REDIRECT --to-ports 1088
iptables -t nat -A PREROUTING -i br-lan -p tcp -j V2RAY

iptables -t mangle -N V2RAY_MASK
iptables -t mangle -A V2RAY_MASK -d 0/8 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 127/8 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 10/8 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 169.254/16 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 172.16/12 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 192.168/16 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 224/4 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 240/4 -j RETURN
iptables -t mangle -A V2RAY_MASK -p udp -j TPROXY --on-port 1088 --tproxy-mark 0x01/0x01
iptables -t mangle -A PREROUTING -i br-lan -p udp -j V2RAY_MASK
EOF
```
```
cat <<'EOF' > /etc/rc.local
# Put your custom commands here that should be executed once
# the system init finished. By default this file does nothing.
ip route add local default dev lo table 100
ip rule add fwmark 1 lookup 100
exit 0
EOF
```
#### Restart Firewall settings for apply iptables rules
    /etc/init.d/firewall restart

### DNS
#### Configure DNS settings for DoH
    opkg update
    opkg install https_dns_proxy
    sed -i 's#https://dns.google.com/resolve?#https://cloudflare-dns.com/dns-query?ct=application/dns-json\&#g' /etc/config/https_dns_proxy
    /etc/init.d/https_dns_proxy enable
    /etc/init.d/https_dns_proxy stop
    /etc/init.d/https_dns_proxy start

#### Configure Dnsmasq settings for DoH
    sed -i "s/option\ noresolv.*/option\ noresolv '1'/" /etc/config/dhcp
    sed -i "s/list\ server.*/list\ server '127.0.0.1#5053'/" /etc/config/dhcp
    sed -i "s/list\ interface.*/list\ interface 'br-lan'/" /etc/config/dhcp
    /etc/init.d/dnsmasq stop
    /etc/init.d/dnsmasq start

### Aria2
#### Aria2 settings for downloading via v2ray
    opkg update
    opkg install aria2 luci-app-aria2 iptables-mod-extra
```
cat <<'EOF' >> /etc/firewall.user
iptables -t nat -A OUTPUT -p tcp -m owner --uid-owner aria2 -j V2RAY
EOF
```

### 4G network
#### Sharing Android 4G network via usb cable
    opkg update
    opkg install kmod-usb-net kmod-usb-net-rndis kmod-usb-net-cdc-ether usbutils
#### Sharing usb stick network via usb port
    opkg update
    opkg install usb-modeswitch kmod-mii kmod-usb-net kmod-usb-wdm kmod-usb-net-qmi-wwan uqmi
