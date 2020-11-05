
# Pull container

docker pull danielguerra/alpine-xfce4-xrdp

# Run it and add wireshark

docker run -d --name rdp -p 3390:3389 danielguerra/alpine-xfce4-xrdp

# Connect with RDP and add wireshark
### rdesktop localhost:3390
### Log in with alpine:alpine
### Install wireshark:

```
sudo apk add wireshark
sudo chown alpine.alpine /usr/bin/dumpcap
```

### commit container to new image:

docker commit <container ID> c9kwireshark:latest

# save image to tar:

docker save -o c9kwireshark.tar <image ID>

# copy c9kwireshark.tar to C9300 usbflash1

scp c9kwireshark.tar admin@10.1.1.5:usbflash1:/c9kwireshark.tar

# connect to C9300 IOS XE 17.3 and configure app hosting

### Set VLAN 47 and IP addresses as needed

```conf t
iox
interface AppGigabitEthernet1/0/1
 switchport mode trunk
no app-hosting appid c9kwireshark
app-hosting appid c9kwireshark
 app-vnic AppGigabitEthernet trunk
  vlan 46 guest-interface 0
   guest-ipaddress 10.1.1.9 netmask 255.255.255.0
 app-default-gateway 10.1.1.3 guest-interface 0
 app-resource docker
 app-resource profile custom
  cpu 7400
  memory 2048
  persist-disk 1024
  vcpu 2
 name-server0 128.107.212.175
```


# Start the container

```
C9300# app-hosting install appid c9kwireshark package usbflash1:c9kwireshark.tar
C9300# app-hosting activate appid c9kwireshark
C9300# app-hosting start appid c9kwireshark
```


# Check the status of the container:

C9300# show app-hosting list

# If you need to stop the container:

```
C9300# app-hosting stop appid c9kwireshark
C9300# app-hosting deactivate appid c9kwireshark
C9300# app-hosting uninstall appid c9kwireshark

```


# Info on the setup:

```
C9300#show inventory
NAME: "c93xx Stack", DESCR: "c93xx Stack"
PID: C9300-24T         , VID: V01  , SN: FCW2129L03Z

NAME: "Switch 1", DESCR: "C9300-24T"
PID: C9300-24T         , VID: V01  , SN: FCW2129L03Z

NAME: "Switch 1 - Power Supply A", DESCR: "Switch 1 - Power Supply A"
PID: PWR-C1-350WAC     , VID: V02  , SN: LIT212445EG

NAME: "Switch 1 FRU Uplink Module 1", DESCR: "2x40G Uplink Module"
PID: C9300-NM-2Q       , VID: V01  , SN: FOC21286K6S

NAME: "usbflash1", DESCR: "usbflash1-1"
PID: SSD-120G          , VID: 3.10 , SN: STP22160GQ1


C9300#show run int gi1/0/24
interface GigabitEthernet1/0/24
 switchport access vlan 46
 switchport mode access
end


C9300#show cdp neighbors
EN-Prog-Infra-SW2	Gig 1/0/24        174              S I   WS-C3750G Gig 1/0/26
Total cdp entries displayed : 1


C9300#show int status  | e notc
Port         Name               Status       Vlan       Duplex  Speed Type
Gi1/0/24                        connected    46         a-full a-1000 10/100/1000BaseTX
Ap1/0/1                         connected    trunk      a-full a-1000 App-hosting port
C9300#
C9300#show ip int brief | e una
Interface              IP-Address      OK? Method Status                Protocol
Vlan46                 10.1.1.5        YES manual up                    up


C9300#dir usbflash1:
Directory of usbflash1:/
12      -rw-       1134255616   Nov 4 2020 21:01:57 -07:00  c9kwireshark.tar
C9300#
```
