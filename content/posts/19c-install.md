+++
date = '2025-03-27T14:32:16-05:00'
draft = false
title = '19c Install'
+++
# How to install Oracle Database 19c on your new OL9 VM
We assume you have a new OL9 VM setup with all the 19c prereqs, and a 100GB /u01 ref scripts in [github](https://github.com/awframpton/19c_install)
## create the directory structures
```
sudo mkdir -p /u01/app/oracle/product/19.0.0.0
sudo mkdir -p /u01/app/oracle/stage/ru
sudo chown -R oracle:oinstall /u01
```
## copy the zips to /u01
```
scp LINUX.X64_193000_db_home.zip oracle@ol9:/u01/app/oracle/stage
scp p37260974_190000_Linux-x86-64.zip oracle@ol9:/u01/app/oracle/stage
scp p6880880_122010_Linux-x86-64.zip oracle@ol9:/u01/app/oracle/stage
```
## unzip them as oracle
```
unzip -q LINUX.X64_193000_db_home.zip -d /u01/app/oracle/product/19.0.0.0
unzip -q p37260974_190000_Linux-x86-64.zip -d ru/
rm -rf /u01/app/oracle/product/19.0.0.0/OPatch
unzip -q p6880880_122010_Linux-x86-64.zip -d /u01/app/oracle/product/19.0.0.0
```
## run the installer
```
bash install_oracle.sh
```
## run the root scripts
```
sudo /u01/app/oracle/product/19.0.0.0/root.sh
sudo /u01/app/oraInventory/orainstRoot.sh
```
