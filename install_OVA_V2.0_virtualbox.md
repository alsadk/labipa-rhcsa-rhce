## Install instructions Lab Environment _Red Hat RHCSA-RHCE 7 Cert Guide_

Sander van Vugt offers an lab environment created with VMware fusion. It is available for download at http://rhatcert.com or through the book’s website
at [http://pearsonitcertification.com](http://www.pearsonitcertification.com/promotions/book-registration-red-hat-rhcsa-rhce-7-cert-guide-premium-140923#vm). Importing this in VirtualBox on macOS is not a trivial case as VMware Fusion uses different network settings. This guide explains how to import Sander's Lab and get a working environment (and maybe learn to understand `nmcli` better).

Notice that on Sander's site there is a updated version with two major changes
- updated to RHEL 7.2
- server1 & server2 use the repository ftp://labipa.example.com/pub and have disabled all online repositories.

This guide does **not** apply to the prerelease version 3.0: ![rhatcert prerelease-vms3 0](https://cloud.githubusercontent.com/assets/16225624/18987281/476202fc-8701-11e6-9ef0-85eb0eb21c1e.png)

Alternatively create a lab with Vagrant and Ansible by building your [RHCSA RHCE Lab environment](../../../RHCSA-RHCE-Lab-Environment/) from scratch.

# Create a host-only network
Open the setting of VirtualBox, go to Network, create an new Host-only Network ![create host-only network](https://cloud.githubusercontent.com/assets/16225624/18674769/a0e60e08-7f50-11e6-9518-a41f4e08bad6.png)
<br />and assign it an network `192.168.4.0/24` with an host ip eg `192.168.4.4` ![subnet192 168 4](https://cloud.githubusercontent.com/assets/16225624/18674930/0fe6f42a-7f51-11e6-9da0-4bd75428a341.png)

## Import _Server1_ into VirtualBox
Download the ovf v2.0 and extract the zip file.

Import the each of the ovf files in VirtualBox
![import labipa server1](https://cloud.githubusercontent.com/assets/16225624/18675014/5266025a-7f51-11e6-96db-aebb874096b7.png)


Before pressing Import, rename the VM and remove dvd & usb ![rename vm and remove dvd usb](https://cloud.githubusercontent.com/assets/16225624/18675161/cb83c7bc-7f51-11e6-8871-d0e84ae8ce4a.png)
<br/>On the FreeIPA server update memory to 2GB.

Before starting the VM, edit the Settings ![edit vm](https://cloud.githubusercontent.com/assets/16225624/18677881/9d7f3fdc-7f5a-11e6-9c87-d6e1cc06352c.png)

Add an host adapter and connect it to host-only network `192.168.4.0/24` ![add host adapter and connect to host-only network](https://cloud.githubusercontent.com/assets/16225624/18675323/4a892ca0-7f52-11e6-8783-0ec111df9d2b.png)

The VM now has 2 adapters:
- Adapter 2 connected to eg. `enp0s8`/`eth0` on subnet 192.168.4.0/24 and will **not** be the default gateway.
- Adapter 1 connected to eg. `enp0s17`/`eth1` will be NAT for internet access (eg 10.0.2.2/24) but will **not** be setting DNS

<!-- ![2 adapters](https://cloud.githubusercontent.com/assets/16225624/18675550/fd218998-7f52-11e6-90ed-e40c4724fdd4.png) -->

Then start the new vm ![start server1](https://cloud.githubusercontent.com/assets/16225624/18676011/8b31155e-7f54-11e6-86ee-d4c739acd62b.png) In Terminal connect with ssh and excute these commands for Server1:
```bash
ssh root@192.168.4.210
cat /etc/centos-release; nmcli --version
mkdir /etc/openldap/cacerts # make the lab easier ;-)
nmcli connection add con-name eth1 type ethernet ifname enp0s17 autoconnect yes save yes
nmcli connection modify eth1 ipv4.ignore-auto-dns yes
nmcli device disconnect enp0s17;nmcli connection up eth1
cat /etc/resolv.conf
ip r
# RHEL 7.0 uses NetworkManager v0.9.9.1 and doesn't support ipv4.gateway, therefor use ipv4.addresses without 'gw'
nmcli connection modify eth0 ipv4.never-default yes ipv6.never-default yes ipv4.addresses "192.168.4.210/24"
systemctl restart NetworkManager; systemctl restart NetworkManager
ip r
# systemctl set-default graphical.target # optional
yum -y update --security ; shutdown now
```
![untitled 23](https://cloud.githubusercontent.com/assets/16225624/18674509/c3c944ea-7f4f-11e6-9a2b-967423186654.png)

To save the current state create a snapshot ![take snapshot copy](https://cloud.githubusercontent.com/assets/16225624/18677143/213924d0-7f58-11e6-9eb1-f9f164a6b16c.png)
<!-- ![take snapshot](https://cloud.githubusercontent.com/assets/16225624/18676516/25bcaac4-7f56-11e6-9ef8-ed19e4b13a61.png) -->

Your server1 is ready for exercises ![snapshot created](https://cloud.githubusercontent.com/assets/16225624/18674637/292f860a-7f50-11e6-9325-848844b7b7a6.png)

## FreeIPA
Like Server1, with these commands
```bash
ssh root@192.168.4.220
cat /etc/centos-release; nmcli --version
grep -B2 -A1 8.8.8.8 /etc/named.conf
ip r
# RHEL 7.1+ uses NetworkManager v1.0.0 which does support ipv4.gateway
nmcli connection modify eth0 ipv4.never-default yes ipv6.never-default yes ipv4.gateway "" ipv4.dns 192.168.4.200
ip r del default via 192.168.4.2 dev eth0
nmcli connection add con-name eth1 type ethernet ifname eth1 save yes autoconnect no
nmcli connection modify eth1 ipv4.ignore-auto-dns yes ipv6.ignore-auto-dns yes connection.autoconnect yes
sleep 2
ip route add default via 10.0.2.2 dev eth1
ip r
echo $'password\npassword\npassword' | kinit admin # password expired
klist
yum -y update --security  ; shutdown now
```


## Server2
Like Server1, with these commands
```bash
ssh root@192.168.4.220
cat /etc/centos-release; nmcli --version
mkdir /etc/openldap/cacerts
nmcli connection add con-name eth1 type ethernet ifname eth1 autoconnect yes save yes
nmcli connection modify eth1 ipv4.ignore-auto-dns yes
nmcli device disconnect eth1;nmcli connection up eth1
cat /etc/resolv.conf
ip r
nmcli connection modify eth0 ipv4.never-default yes ipv6.never-default yes ipv4.addresses "192.168.4.220/24"
systemctl restart NetworkManager; systemctl restart NetworkManager
ip r
yum -y update --security ; shutdown now
```
Alternatively
```bash
ssh root@192.168.4.220
cat /etc/centos-release; nmcli --version
nmcli connection add con-name eth1 type ethernet ifname eth1 save yes autoconnect no
nmcli connection modify eth1 ipv4.ignore-auto-dns yes connection.autoconnect yes
sleep 1
nmcli connection modify eth0 ipv4.never-default yes ipv6.never-default yes ipv4.addresses "192.168.4.220/24"
sleep 1
ip r del default via 192.168.4.2 dev eth0
sleep 1
ip r
cat /etc/resolv.conf # yum only works if FreeIPA @ 192.168.4.200 is up and serving DNS
# 'yum -y groupinstall Directory\ Client' fails on server2
yum -y update --security ; shutdown now
```


# VMware Fusion
Tested with VMWare Fusion 8.5 on macOS.

## labipa V2.0
```bash
nmcli connection modify eth0 ipv4.dns 192.168.4.200 ipv6.dns fd00::200
systemctl restart NetworkManager
sleep 2
cat /etc/resolv.conf
echo $'password\npassword\npassword' | kinit admin
# https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/Configuring_IPA_Users-Specifying_Default_User_Settings.html
ipa config-mod --homedirectory /home/ldap --defaultshell /bin/bash
for i in `ipa user-find | grep "User login: " | sed 's/User login: //g'`; do echo $'password'| kinit admin; ipa user-mod $i --homedir=/home/ldap/$i --shell=/bin/bash | grep -E "Modified|Home|Login"; echo $'password\npassword\npassword'| kinit $i ; done
```
shutdown labipa and in VMware Fusion create a snaphot and call it 'begin'

## server1
```bash
yum whatprovides */host | grep bind
mkdir -p /etc/openldap/cacerts
```

## server2
```bash
yum -y install bash-completion
yum whatprovides */host | grep bind
mkdir -p /etc/openldap/cacerts
```
