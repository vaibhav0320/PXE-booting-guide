# PXE-booting-guide
#### This is a guide to setup pxe booting on virtualbox with optional kickstart setup to automatic OS provisioning<br />

All the commands here are executed on Cent-OS-7. Modify the commands if you are using other flawor <br />
### First step is setup DHCP, tftp and webserver 
1. Create a virtual machine with 2 network interfaces
    1. NAT/Bridge network for internet
    2. internal network ( give it a name like "mynetwork")

2. Install the required services httpd(apache) webserver, tftp-server and dhcp server
```bash
#On cent os 
yum install httpd tftp-server dhcp
```
3. Now have to setup a Static ip on our host server  for internal network of virtualbox
    1. Choose any ip range of your choice. In the example here I will use the range 192.168.110.0/24.
    2. do ```ip -a ``` on console to find out the interface for internal network. There should be one interface starting with ```eth``` without any ip address. Note down the interface. In my case it is ``` eth1```.
    3. Need to edit network config file for this network interface for that in centos I've edited ``` vim /etc/sysconfig/network-scripts/ifcfg-eth1```. In debian path might be diffrent but works the same. 
    4. Content of the file should be
    ``` bash
        NM_CONTROLLED=yes
        BOOTPROTO=none
        ONBOOT=yes
        IPADDR=192.168.110.2 #your choice of IP. It will be static 
        NETMASK=255.255.255.0 #netmask for you ip range
        DEVICE=eth1
        PEERDNS=no
    ```
    5. Restart the network service. In centos ``` systemctl restart network.service```. Now if you do ```ip -a``` you should see your ip. If now troubleshoot further.


<br /><br />
### Some theroy part of PXE boot for nerds
1. When machine booted into pxe boot the first thing pxe will do is search for dhcp server and get ip for itself and find the tftp server to get the pxelinux config file.
2. So for that when we configure dhcp server we will also mention the ip address of tftp server in dhcp config file.
3. Once it knows the tftp server we tell pxe to find ```pxelinux.0``` file which is hardcoded to get the content from ```pxelinux.cfg``` directory
4. In pxelinux.cfg directory we have location for diffrent os kernals, initramfs and  url for installation media
5. Once kernal is found it will load it and start installtion. If OS support kickstart then it is possible to automate entire installtion process


### Setting up DHCP server
1. Edit ```/etc/dhcp/dhcpd.conf```. I've used basic config only just for pxe boot.
```bash
ddns-update-style interim;
ignore client-updates;
authoritative;
allow booting;
allow bootp;
allow unknown-clients;

# internal subnet for my DHCP Server
subnet 192.168.110.0 netmask 255.255.255.0 {
range 192.168.110.50 192.168.110.60;   #any range in subnet
option domain-name-servers 192.168.110.2;
option domain-name "websrv.local.vm";
option routers 192.168.110.1;
option broadcast-address 192.168.110.255;
default-lease-time 600;
max-lease-time 7200;

# IP of PXE Server
next-server 192.168.110.2; #tftp server ip
filename "pxelinux.0";  
}
```
2. start dhcpd service ``` systemctl start dhcpd.service```.


### Test dhcp server
1. Create a new virtual machine without any ISO and one internal network adapter which should have same name as internal network of dhcp server. Boot the machine into pxe ( change boot order ) and see if it get assiged ip.
If not troubleshoot dhcp server and new vm else move to setup tftp server.

