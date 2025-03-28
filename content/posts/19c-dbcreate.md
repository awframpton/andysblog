+++
date = '2025-03-28T13:53:10-05:00'
draft = false
title = '19c DBcreate'
+++
# How to create a 19c database using scripts
ref scripts in [github](https://github.com/awframpton/19c_install/tree/master/dbcreate)
```
# make sure you are logged in as oracle
# make sure your db environment is set, ORACLE_HOME, ORACLE_SID and all that

# this is the main directory where all the scripts should live, copy them here
cd /u01/app/oracle/admin/orcl/scripts

# create directories
bash create_dirs.sh

# create the database
sqlplus / as sysdba <<EOD
create spfile from pfile='/u01/app/oracle/admin/orcl/scripts/init.ora';
startup nomount
@crdb1.sql <sys_password>
EOD

# run catalog
bash catalog.sh <sys_password>

# install text
bash text.sh <sys_password>

# install cluster views
bash cluster_views.sh <sys_password>

# lock accounts via sqlplus
sqlplus / as sysdba
@lock_accounts
@show_accounts

# run datapatch
cd $ORACLE_HOME/OPatch
./datapatch -verbose

# compile invalids
bash utlrp.sh <sys_password>

# check the registry and patches
sqlplus / as sysdba
@dba_registry
@sqlpatch

# create a PDB
sqlplus / as sysdba
create pluggable database pdb1 admin user pdbadmin identified by "<password>";
alter pluggable database pdb1 open;
```
