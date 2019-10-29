# Raspberry Pi access point for receiving data from ESP8266

1- Install ```hostapd``` and ```dnsmasq```

```sudo apt-get install hostapd dnsmasq```

2- Uncomment and edit these lines in ```/etc/dnsmasq.conf``` :

```interface=lo,uap0```

```no-dhcp-interface=lo,wlan0```

```dhcp-range=192.168.2.100,192.168.2.200,12h```

3- Edit: ```/etc/hostapd/hostapd.conf``` and add 
```
interface=uap0
ssid=APssid
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=APpassword
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

4- Use the SSID of the access point and its password to generate a  wpa passphrase:

```sudo wpa_passphrase ssidofAP yourWPApass```

5- Edit ```/etc/network/interfaces``` and add:
```
auto wlan0
iface wlan0 inet dhcp
wpa-ssid APssid
wpa-psk (copy wpa passphrase here)

auto uap0
iface uap0 inet static
address 192.168.2.1
netmask 255.255.255.0
```
6- Create a new file ```/usr/local/bin/hostapdstart``` and add:
```
iw dev wlan0 interface add uap0 type __ap
service dnsmasq restart
sysctl net.ipv4.ip_forward=1
iptables -t nat -A POSTROUTING -s 192.168.2.0/24 ! -d 192.168.2.0/24 -j MASQUERADE
ifup uap0
hostapd /etc/hostapd/hostapd.conf
```
7- Change permissions on ```/usr/local/bin/hostapdstart```

```sudo chmod 667 /usr/local/bin/hostapdstart```

8- Type ```hostapdstart``` to start the access point or run it at startup by adding this line to the /etc/rc.local:

```hostapdstart >1&```


