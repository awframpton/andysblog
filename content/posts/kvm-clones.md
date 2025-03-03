+++
date = '2025-03-02T10:15:54-06:00'
draft = false
title = 'KVM Clones'
+++
# How to build and spin up a VM using an image downloaded from the Internet
I recently started an online class which provided a VM image in an OVA format.  I run Linux as my main OS and like to use libvirt and KVM as they seem as close to native as possible.  In other words, with libvirt and KVM, I am running a virtualization platform that is as close to the Linux OS as possible. ie. it is leveraging virtualization support in the kernel itself, and libvirt seems to be the most commonly used virtualization software on Linux.

Based on other learning I have been doing on KVM, I built a nice little process for taking an OVA file and spinning up lightweight clones of it.

## Extract files from the OVA
You can extract files from an ova file using tar.

First create a directory and copy your ova into it. Then untar
```
[andy@andyarco oraclefree]$ ls -lh
total 6.0G
-rw-r--r-- 1 andy andy 6.0G Mar  2 10:27 Oracle_Database_23ai_Free_Developer.ova
[andy@andyarco oraclefree]$ tar -xvf Oracle_Database_23ai_Free_Developer.ova
Oracle Database 23ai Free.ovf
Oracle Database 23ai Free-disk001.vmdk
```
Now convert the vmdk to a qcow2
```
[andy@andyarco oraclefree]$ qemu-img convert -f vmdk -O qcow2 'Oracle Database 23ai Free-disk001.vmdk' oracle_database_23ai_disk001.qcow2
```
Copy the qcow2 to where you keep your VM images
```
[andy@andyarco oraclefree]$ sudo cp oracle_database_23ai_disk001.qcow2 /vol1/vm
```
Create your `build_vm.sh` script.  The clever bit of this script is the `qemu-img` command which creates the lightweight clone.  The originally image remains untouched.
```
[andy@andyarco oraclefree]$ cat build_vm.sh
TEMPLATE=oracle_database_23ai_disk001.qcow2
VCPUS=2
MEMORY=8192
DOMAIN=$1
sudo qemu-img create -f qcow2 -F qcow2 -b "/vol1/vm/${TEMPLATE}" "/vol1/vm/${DOMAIN}.qcow2"
sudo virt-install --name "${DOMAIN}" \
  --memory ${MEMORY} \
  --vcpus ${VCPUS} \
  --disk "/vol1/vm/${DOMAIN}.qcow2,device=disk,bus=virtio" \
  --os-variant ol8.9 \
  --virt-type kvm --graphics spice \
  --network network=default,model=virtio \
  --noautoconsole \
  --import
```
You may need to tweak os-variant.  Try to get it as close as possible to the OS of the image.  If you aren't sure, you can just make a guess and come back and fix it later, once you have booted it up.

Use `osinfo-query` to see a list of OS names you can use.  Here I am piping it into fuzzy find. [fzf](https://github.com/junegunn/fzf) is a slick tool.  It is basically an interactive grep. [Distrotube](https://youtu.be/Zq3mFU2T9cw?si=dWmq4NTHbMvyPGEe)  does a good job explaining it.
```
[andy@andyarco oraclefree]$ osinfo-query os | fzf
```
Now you can build the VM.
```
[andy@andyarco oraclefree]$ ./build_vm.sh ora23ai
Formatting '/vol1/vm/ora23ai.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off compression_type=zlib size=62914560000 backing_file=/vol1/vm/oracle_database_23ai_disk001.qcow2 backing_fmt=qcow2 lazy_refcounts=off refcount_bits=16

Starting install...
Creating domain...                                                                                                             |         00:00:00
Domain creation completed.

```
Launch `virt-viewer` to view the console.  This should launch another window with the console.
```
[andy@andyarco oraclefree]$ sudo virt-viewer ora23ai & 
```
