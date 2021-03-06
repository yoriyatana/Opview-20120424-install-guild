# Opview-20120424-install-guide

## Prerequisites

### Disable SeLinux
To permanently disable SELinux, use your favorite text editor to open the file `/etc/sysconfig/selinux` as follows:
```
root# vim /etc/sysconfig/selinux
```
![SELinux Enforcing Mode](https://www.tecmint.com/wp-content/uploads/2016/07/SELinux-Enforcing-Mode.png)

Then change the directive `SELinux=enforcing` to `SELinux=disabled` as shown in the below image.
```
SELINUX=disabled
```
![Disable SELinux Permanently](https://www.tecmint.com/wp-content/uploads/2016/07/Disable-SELinux.png)

Then, save and exit the file, for the changes to take effect, you need to reboot your system and then check the status of SELinux using sestatus command as shown:
```
root# sestatus
```
![Check SELinux Status](https://www.tecmint.com/wp-content/uploads/2016/07/Check-SELinux-Status.png)

### Install the required packages.
```
root# yum install -y epel-release vim mysql-server
```

## OPSview RPM Installation (Offline)
Download the following files and transfer these to your Opsview server.
```
opsview-3.20120424.0.8487-1.el6.noarch.rpm
opsview-base-4.0.0.8487-1.el6.x86_64.rpm
opsview-core-3.20120424.0.8487-1.el6.noarch.rpm
opsview-perl-4.0.0.660-1.el6.x86_64.rpm
opsview-web-3.20120424.0.8487-1.el6.noarch.rpm
```

As root you should then install these packages by running
```
root# yum localinstall -y --nogpgcheck opsview-*
```
## Configuration
After the Opsview packages have been installed, it is necessary to configure Opsview and its databases.
### MySQL
Add the following to the `[ mysqld]` section of `'/etc/my.cnf'`:
```
innodb_file_per_table=1
innodb_flush_log_at_trx_commit=2

innodb_buffer_pool_size=3G # Recommend 80% RAM
thread_cache_size = 20
query_cache_size=0
query_cache_type=0
query_cache_limit=1M
```
![MySQL Performance Tuning](https://lh3.googleusercontent.com/mxaOSp7pNDlG280dSdq8Ch8LEr2CqNOUJ5IN442ayOJbk8lED-9h0-k2aM7xqK36zQxzPczg3Hs67QpCCgK4u3ycv42UBRq2vkxZAesbfB9nkuJ0j1o8x9bksLEY6BZtlxaYbTQaulpT4ZQpBSs4JqZp6lROFTfagpC6XgmI8Zy_P-R7FYE--dkTAso7jNx5Ch5D-0NucDp56hND7B3kgHc57PFoAXpx031UANz1I-v54xdcqMsjbmC5TjmhwSqThhiwgF6DmwL9-JoS5h1_iApofsmhmN5ou1TrkGKnuglyUHV53cnstJXFu2m5PwdkXgoQPt1wkBz3MgeurnL83TdTIfGH5Mc5Wl_5CGGFB13h2qrx3ScOSFEC82irHFICl3BKcuR6V4Ci_LVcLms1V3pGCBbiTBCGuhvKmQbcSeMEoS9nBcbqY5qaWRG8YxWxq91J3kNNKZs-1-Z0sJypHcvBWT9jQLiQXDFeOUtNqfdQCJrRqQCI6xw65YLSRJtjIbNEllZNBMepXpM02Zx9UhL_Dgn1gthpmPO9h_hwj6doB4G10sMXSYYDMyorHHlemIco7vgS_uWYSb7j5kytHcZVN-JrFPDIJ9KzRI8=w619-h300-no)

 Start the MySql server:
```
root# chkconfig mysqld on
root# service mysqld start
```
Change the MySql root password:
```
root# /usr/bin/mysqladmin -u root password 'mysql_root_password'
```

### Adding the Nagios User and Group (optional)

Next add the appropriate user and groups for the Nagios process to run:
```
root# useradd nagios
root# groupadd nagcmd
root# usermod -a -G nagcmd nagios
```

For RHEL/CentOS users:
```
root# usermod -a -G nagios,nagcmd apache
```
### Create the nagios profile
The rest of the steps should be performed as the nagios user
```
root# su nagios
```
Add the following line to the nagios users `'.bash_profile'`.
```
nagios$ echo "test -f /usr/local/nagios/bin/profile && . /usr/local/nagios/bin/profile" >> ~/.bash_profile
```
### Create the Nagios database
Edit the opsview configuration file and change the password as you see fit to secure the system (those
passwords that should be changed as set to changeme by default)
```
nagios$ vim /usr/local/nagios/etc/opsview.conf
```

Set up the Opsview mysql database users with the necessary permissions
```
nagios$ /usr/local/nagios/bin/db_mysql -u root -p 'mysql_root_password'
```

Install the Opsview databases
```
nagios$ /usr/local/nagios/bin/db_opsview db_install
nagios$ /usr/local/nagios/bin/db_runtime db_install
nagios$ /usr/local/nagios/bin/db_odw db_install
nagios$ /usr/local/nagios/bin/db_reports db_install
```

Generate all the necessary configuration files for Opsview and Nagios to run
```
nagios$ /usr/local/nagios/bin/rc.opsview gen_config
```
### Start the web service
You can now start up the web application server:
```
# service opsview-web start
```
The Opsview server is now listening on port 3000.
### Initial setup
Use a web browser to view the web interface on port 3000 of the host. The initial credentials are:
```
Username:	admin
Password:	initial
 ```
![Login sreenshot](https://plone.lucidsolutions.co.nz/web/management/images/screensnap-2011-09-29-174514.png/image_preview) 

## Source

* https://www.tecmint.com/disable-selinux-temporarily-permanently-in-centos-rhel-fedora/
* https://plone.lucidsolutions.co.nz/web/management/ebony-an-opsview-centos-6-monitoring-vm
* https://knowledge.opsview.com/v5.0/docs/rpm-installation-offline
* https://assets.nagios.com/downloads/nagioscore/docs/Installing_Nagios_Core_From_Source.pdf
