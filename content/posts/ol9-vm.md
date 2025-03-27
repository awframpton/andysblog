+++
date = '2025-03-26T20:50:08-05:00'
draft = false
title = 'OL9 VM'
+++
# How to build an Oracle Linux 9 VM for installing Oracle Database 19c
This doc will cover how to build an Oracle Linux 9 VM for installing the Oracle database. Ref scripts in [github](https://github.com/awframpton/19c_install)
## Build the VM
This step leverages the build_vm_ol9.sh script to build the VM
```
bash build_vm_ol9.sh ol9
```
## Disable selinux
Set selinux to permissive.  This will print warnings, but will not deny anything
```
sudo vi /etc/selinux/config
SELINUX=permissive
```
## Disable firewall
Disable the firewall
```
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo reboot
```
## Set the ip
First get the connection name.  In the below example it is "System eth0"
```
sudo nmcli device status
DEVICE  TYPE      STATE                   CONNECTION
eth0    ethernet  connected               System eth0
lo      loopback  connected (externally)  lo
```
Now we are going to give "System eth0" a static ip
```
sudo nmcli con mod "System eth0" ipv4.addresses 192.168.122.33/24
sudo nmcli con mod "System eth0" ipv4.gateway 192.168.122.1
sudo nmcli con mod "System eth0" ipv4.method manual
sudo nmcli con mod "System eth0" ipv4.dns "192.168.122.1"
sudo reboot
```
## Create a disk for /u01
Now we will use `qemu-img` to create a 100GB disk
```
sudo qemu-img create -f qcow2 /vol1/vm/ol9-u01.qcow2 100G
```
## Add the disk to the VM
```
sudo virsh attach-disk ol9 /vol1/vm/ol9-u01.qcow2 vdb --subdriver qcow2 --persistent
```
## partition the disk
```
sudo fdisk /dev/vdb
```
## format the disk
```
sudo mkfs.xfs /dev/vdb1
```
## get uuid
```
sudo blkid
```
## add a new line to /etc/fstab with uuid
```
sudo vi /etc/fstab
UUID=d2826585-bd79-4219-9026-acd5ea9740e3 /u01                   xfs     defaults        0 0
```
## mount /u01
```
sudo systemctl daemon-reload
sudo mkdir /u01
sudo mount /u01
```
## eject cdrom
```
sudo virsh change-media ol9 sda --eject
```
## install prereqs
```
sudo dnf install oracle-database-preinstall-19c
```
