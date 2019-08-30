# v2ray-raspberry
    Installing v2ray on raspberry pi 4B

### Raspberry Pi 4B
#### Login to raspberry pi 4B via SSH
    ssh root@192.168.1.1
#### Update OpenWRT repos by opkg command
    opkg update
#### Install Package(ca-certificates/curl/unzip) for downloading v2ray binary
    opkg install ca-bundle ca-certificates libustream-openssl curl unzip iptables-mod-tproxy
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

#### Download luci-app-v2ray ipk file
    curl -L  -H "Cache-Control: no-cache" -o /tmp/luci-app-v2ray.ipk https://github.com/kuoruan/luci-app-v2ray/releases/download/v1.2.2-2/luci-app-v2ray_1.2.2-2_all.ipk

#### Install luci-app-v2ray using opkg install command
    opkg install /tmp/luci-app-v2ray.ipk



#### V2Ray - Global Settings
    - [x] V2Ray file: /usr/bin/v2ray/v2ray
    - [x] V2Ray asset location: /usr/bin/v2ray/

    - [x] socks_proxy
    - [x] transparent_proxy
    - [x] upstream_vmess
    - [x] direct
    - [x] block

#### V2Ray - Outbound - upstream_vmess
    Settings
```
{
   "vnext": [
     {
       "port": 443,
       "users": [
         {
           "id": "117ff1a7-d810-4ec7-b368-6fc4491a4435",
           "level": 1,
           "alterId": 0,
           "security": "none"
         }
       ],
       "address": "www.example.com"
     }
   ]
 }
```
    Stream settings
```
{
   "security": "tls",
   "tlsSettings": {
     "serverName": "www.example.com",
     "allowInsecure": true
   },
   "network": "ws",
   "wsSettings": {
     "path": "\/v2\/",
     "headers": {
       "Host": "www.example.com"
     }
   }
 }
```
#### V2Ray - Routing
    - [x] enabled

### Aria2
#### Aria2 settings for downloading via v2ray
    opkg update
    opkg install aria2 luci-app-aria2 ariang iptables-mod-extra
```
cat <<'EOF' >> /etc/firewall.user
iptables -t nat -A OUTPUT -p tcp -m owner --uid-owner aria2 -j V2RAY
EOF
```

### USB Automount
#### USB will be automount when input

### mount-utils
```
opkg update
opkg install kmod-usb-ohci kmod-usb2 kmod-usb-uhci kmod-usb-storage
opkg install kmod-fs-vfat kmod-fs-ntfs ntfs-3g ntfs-3g-utils
opkg install block-mount mount-utils
opkg install fdisk
opkg install kmod-nls-cp437 kmod-nls-iso8859-1
```

### Modules for USB 1.1
```
opkg update
opkg install kmod-usb-uhci
insmod usbcore
insmod uhci
```

### Modules for USB 2.0
```
opkg update
opkg install kmod-usb2
insmod ehci-hcd
```

### Modules for USB 3.0
```
opkg install kmod-usb3
insmod kmod-usb3
```

### 4G network
#### Sharing Android 4G network via usb cable
    opkg update
    opkg install kmod-usb-net kmod-usb-net-rndis kmod-usb-net-cdc-ether usbutils
#### Sharing usb stick network via usb port
    opkg update
    opkg install usb-modeswitch kmod-mii kmod-usb-net kmod-usb-wdm kmod-usb-net-qmi-wwan uqmi

