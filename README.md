# Multinode Ambari Cluster on CentOS7s

This is a complete guide on how to setup a multi-node ambari cluster on CentOS7 using VirtualBox. The cluster is developed to design and 
implemenet Bigdata Solutions and provide all the services that are required to serve the purpose.

**`Note:`** During the installation, if you are facing any issues, please visit the `Issues` part of the documentation as you might be facing the same issue that I faced. 

## 1. Downloading Required Stuff:
* [CentOS7 ISO Image](https://www.centos.org/download/)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

Now open virtual box and click on the **new** button in order to create a new VM (Virtual Machine).

## 2. Creating a VM using VirtualBox
* Click on the new button.
* Add the name of the VM as per your choice.
       1. Type: Linux
       2. Version: Red-Hat (64 bit)
* Assign memory according to the current Memory installed on your system. (4GB Recommended)
* Select **Create a virtual hard disk now** option
* Select **VDI (Virtualbox Disk Image)**
* Select **Dynamically Allocated** 
* Assign the storage according to the available storage on your system. (Atleast 8GB)
* Creating Adapters
       1. Goto File > Host Network Manager
       2. There are 3 buttons shown namely: **create, remove** and **properties**
       3. Create 2 Adapters and name them **vboxnet0** and **vboxnet1**
* Now go back to the VirtualBox and select the VM that is just created and click on the **settings** button.
* Goto **Adapter** tab.
       1. Under **Adapter2** tab, select the Enable Network Adapter checkbox and assign vboxnet0.
       2. Goto **Adapter3** tab, select the Enable Network Adapter checkbox and assign vboxnet1.
* Now goto **Storage** tab in settings and Select the **empty** disk. on the right side, add the CentOS7 ISO Image.
* Click **OK** and you are done.
* Now run the VM. 

After the creation of the VM is done, you can repeat step 2 in order to create as many nodes as you need for the cluster.
Note that the more nodes you create, the more resources will be consumed from your system. 
For a normal cluster, you need atleast **1 Master and 2 slaves**

## 3. Installation of CentOS7 on VM
   The Installation is relatively simple.
* Select the language.
* Add the appropriate date and time.
* Goto **Network and Hostname** and select all 3 adapters
* Click on **Begin Installation**
* After the Installation add the **Root Password**. (Creating a user is not that necessary)

After the Installation is done, the system will ask to reboot. after the reboot it will ask for the following credentials
* username: **root**
* password: **password that you just set in step 5**

after entering the following credentials, you are currently in the localhost.

## 4. Setup Static IP Address
   It is important to setup a static ip address in order to access the node on the network.

* To set a static IP address, use the following command to add configurations:

            vi /etc/sysconfig/network-scripts/ifcfg-**enp0s8**

Note that **enp0s8 is the name of the adapter in my machine**
* You will be prompted with the following contents of the file:

            TYPE=Ethernet
            BOOTPROTO=dhcp
            DEFROUTE=yes
            PEERDNS=yes
            PEERROUTES=yes
            IPV4_FAILURE_FATAL=no
            IPV6INIT=yes
            IPV6_AUTOCONF=yes
            IPV6_DEFROUTE=yes
            IPV6_PEERDNS=yes
            IPV6_PEERROUTES=yes
            IPV6_FAILURE_FATAL=no
            NAME=enp0s8
            UUID=c3fb64f0-6ff5-4294-9354-8e1279971d7e
            DEVICE=enp0s8
            ONBOOT=no

* make the following changes, set:

            BOOTPROTO=static
            ONBOOT=yes
            IPADDR=192.168.XXX.XXX
            NETMASK=255.255.255.0

* restert the network by executing the command:

            systemctl restart network

* Static IP will be set after the restart.


## 5. Check the availlable memory and storage
* to check the available memory of the system, use the following command:

            free -m

* to check the available storage, use the following command:

            df

## 5. Maximum number of open file requirements
* To check the current number of available number of open file descriptors, use the following command:

            ulimit -Sn
            ulimit -Hn

* To set the value of open file descriptors. (recommended: 10000)

            ulimit -n 10000

## 6. Changing the hostname of the machine
* By default, the hostname is: `localhost`
* To check the current running hostname, run the following command:

            hostname -f

* To check the details of hostname, run the command:

            hostnamectl

* To change the hostname of the system:

            hostnamectl set-hostname hostname

* reboot the system using the `reboot` command to make the changes.


## 7. Setting the hostname in the hostfile
* To view the hosts in the hosts file, enter the following command:

            vi /etc/hosts

* add the hosts in a fashion of adding an IP of a node and the hostname, the sample is given below:

            127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
            ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
            192.168.XXX.XXX master
            192.168.YYY.XXX datanode1
            192.168.XXX.YYY datanode2

* in the above contents of the file, `master`, `datanode1` and `datanode2` are the names of the hosts.
* make sure to add these settings on each host of the cluster in order for the hosts to work.
* once these hosts are set, we can SSH any hosts using the hostname instead of the IP Address

            ssh master
            ssh datanode1
            ssh datanode2

## 8. Setup Password-less SSH
Password-less enables each node in the cluster to ssh any node without authentication or password. follow the steps given below in order to setup passwordless SSH
* Use the following command to generate a key. 

            ssh-keygen

* Press `enter` for all the default options provided to save the key. The command will generate 2 keys, a public key and a private key.

            Generating public/private rsa key pair.
            Enter file in which to save the key (/root/.ssh/id_rsa): 
            Created directory '/root/.ssh'.
            Enter passphrase (empty for no passphrase): 
            Enter same passphrase again: 
            Your identification has been saved in /root/.ssh/id_rsa.
            Your public key has been saved in /root/.ssh/id_rsa.pub.
            The key fingerprint is:
            97:0e:4c:a4:21:97:42:58:86:11:f1:74:e2:5a:82:cb root@localhost.localdomain
            The key's randomart image is:
            +--[ RSA 2048]----+
            |  +OB +..        |
            | .o=.=.+         |
            |. . +.. .        |
            |.. +   o   .     |
            |.E.     S o      |
            |         +       |
            |          .      |
            |                 |
            |                 |
            +-----------------+

* Now enter the following command to copy the generated key to the `authorized_keys` file:

            cat .ssh/id_rsa.pub >> authorized_keys

* now change the following permissions:

            chmod 700 ~/.ssh
            chmod 600 ~/authorized_keys

* Now use the following command to copy the key to the target host in order to set the password-less SSH:

            ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.XYX.XYX

* Note that in the command above, the target host can also be accessed using the hostname instead of using the IP.
* Enter the password of the target host. And now you can access 192.168.XYX.XYX from the current host
* repeat the above process in order to setup the Password-less SSH on other nodes as well.

## 9. Setup NTP on the server and the browser host
NTP ensures that the clocks of the nodes in the cluster are synchronized. follow the steps given below in order to set the NTP on each node:
* Install NTP using the command: 

            yum install -y ntp

* Enable NTP using the command; 

            systemctl enable ntpd

* following commands are used to start and restart the NTP service.

            systemctl start ntpd
            systemctl restart ntpd

* If you want to check the sync status, use the following command:

            ntpstat

## 10. Disabling Firewalls / Configuring IP tables

* execute the following 2 commands given below to disable the firewall

            systemctl disable firewalld
            service firewalld stop

## 11. Set the Network Config File

* Change the contents of the following file by using the command:

            vi /etc/sysconfig/network

* make the following changes in the file:

            NETWORKING=yes
            HOSTNAME=localhost

## 12. Disable SELinux and PackageKit and check umask value
It is important to disable SELinux for the ambari function to setup. 
* Use the following command to disable SELinux:

            setenforce 0

* to permenantly disable SELinux, make sure to set `SELINUX=disabled` in the following file to make sure that it is not enables after the system is rebooted:

            vi /etc/selinux/config

* For packagekit, set `enabled=0` in the following file:

            vi /etc/yum/pluginconf.d/refresh-packagekit.conf

* To check the current umask, run thhe following command:

            umask

* set the umask just fot the current login session: 

            umask 0022

* permenantly changing the umask for all the users: 

            echo umask 0022 >> /etc/profile

## 13. Configuring MySQL for Ambari

Installing and setting-up MySQL is different as the default database that is provided by ambari is PostgreSQL, so follow the steps bellow to install SQL and Configuring it with Apache Ambari

* We need the package called `wget` to get repositoried and download services from CentOS. Install wget using the commmand:

            yum install wget

* Now we need to download MySQL.
* **Note:** The version of ambari used in this guide is 2.7.3.0 and it does not support MySQL version 8, it has some issues. make sure to install the MySQL version 5 or 6.
* Download MySQL v57 using the following `wget` link and installation commands.

              wget https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
              md5sum mysql57-community-release-el7-9.noarch.rpm
              sudo rpm -ivh mysql57-community-release-el7-9.noarch.rpm

* Now use the following command to install MySQL:

              yum install mysql-server

* Once MySQL is downloaded, check the status of MySQL before and after starting the service, execute the following commands:

              systemctl status mysqld
              systemctl start mysqld
              systemctl status mysqld

* Once MySQL service is running, log in with the `root` user with the following command: 

              mysql -u root -p

* after the above command, it will ask for the password, if the password was not set earlier, then we get get the password using:

              grep 'password' /var/log/mysqld.log

* The first step that you have to perform is to change the password using the `ALTER USER` command, because without changing password you cannot query anything when MySQL is just installed, so execute the following command:

              ALTER USER 'root'@'localhost' IDENTIFIED BY 'Ambari123!';

* Once we are logged in, we have to create a user and set a password that matches the ambari-setup configuration and MySQL, so for that we have to reduce the level of password strength policy, for that we can use the given command to check if the `validate_password_policy` is high or not:

              SHOW VARIABLES LIKE 'validate_password%';

* Following result is shown once the above query is executed: 

              +--------------------------------------+--------+
              | Variable_name                        | Value  |
              +--------------------------------------+--------+
              | validate_password_check_user_name    | OFF    |
              | validate_password_dictionary_file    |        |
              | validate_password_length             | 8      |
              | validate_password_mixed_case_count   | 1      |
              | validate_password_number_count       | 1      |
              | validate_password_policy             | MEDIUM |
              | validate_password_special_char_count | 1      |
              +--------------------------------------+--------+

* now execute the following to reduce the `validate_password_policy` level using the following command: 

              SET GLOBAL validate_password_policy=LOW;

* Once logged in, you have to create a user for ambari, for that execute the following on MySQL: 

              CREATE USER 'ambari'@'%' IDENTIFIED BY 'ambari123';
              GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'%';
              CREATE USER 'ambari'@'localhost' IDENTIFIED BY 'ambari123';
              GRANT ALL PRIVILEGES ON *.* TO 'ambari'@'localhost';
              FLUSH PRIVILEGES

* Now use the `ALTER USER` command again to set the same password for sql ass well:

              ALTER USER 'root'@'localhost' IDENTIFIED BY 'ambari123';

* For installing the JDBC connector, use the following command:

              yum install mysql-connector-java

*  To check the path of java, use the command:

              which java

* Now you can login using the `ambari` user: 

              mysql -u ambari -p

* Check the current logged in user using the command:

              SELECT USER();


## 14. Installation of Ambari Server and Agents

For the proper cluster, we need to install ambari server in the master node and the ambari agent on all the nodes including the master node:

* Login as root and download the ambari-repo using the following wget link: 

              wget -nv http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.7.3.0/ambari.repo -O /etc/yum.repos.d/ambari.repo

* you can check the repo-list using the following command: 

              yum repolist

* to install the ambari-server, execute only on the master node

              yum install ambari-server

* to install ambari agent, execute this on all the nodes (including master node)

              yum install ambari-agent

* Now login into MySQL using the `ambari` user and the password set for that:

              mysql -u ambari -p

* Use the following commands to create a database and run the DDL:

              CREATE DATABASE ambari;
              USE ambari;
              SOURCE /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql;

* Create a Hive user, open MySql from root (`mysql -u root -p`) and execute the following commands:

              CREATE USER 'hiveuser'@'localhost' IDENTIFIED BY 'ambari123';
              GRANT ALL PRIVILEGES ON *.* TO 'hiveuser'@'localhost';
              CREATE USER 'hiveuser'@'%' IDENTIFIED BY 'ambari123';
              GRANT ALL PRIVILEGES ON *.* TO 'hiveuser'@'%' IDENTIFIED BY 'ambari123';
              FLUSH PRIVILEGES;
              CREATE DATABASE hive

* Create a Oozie user, open MySql from root (`mysql -u root -p`) and execute the following commands:

              CREATE USER 'oozieuser'@'%' IDENTIFIED BY 'ambari123';
              GRANT ALL PRIVILEGES ON *.* TO 'oozieuser'@'%';
              FLUSH PRIVILEGES;
              CREATE DATABASE oozie;

* **NOTE:** Keep in mind that the `username`, `database name` is **`ambari`** and the `password` is **`ambari123`**.

* Setup `ambari-setup` using the following command:

              ambari-server setup

* Now make the following choices in the `ambari-server setup`

              Using python  /usr/bin/python
              Setup ambari-server
              Checking SELinux...
              SELinux status is 'disabled'
              Customize user account for ambari-server daemon [y/n] (n)? 
              Adjusting ambari-server permissions and ownership...
              Checking firewall status...
              Checking JDK...
              [1] Oracle JDK 1.8 + Java Cryptography Extension (JCE) Policy Files 8
              [2] Custom JDK
              ==============================================================================
              Enter choice (1):  
              To download the Oracle JDK and the Java Cryptography Extension (JCE) Policy Files you must accept the license terms found at http://www.oracle.com/technetwork/java/javase/terms/license/index.html and not accepting will cancel the Ambari Server setup and you must install the JDK and JCE files manually.
              Do you accept the Oracle Binary Code License Agreement [y/n] (y)? 
              Downloading JDK from http://public-repo-1.hortonworks.com/ARTIFACTS/jdk-8u112-linux-x64.tar.gz to /var/lib/ambari-server/resources/jdk-8u112-linux-x64.tar.gz
              jdk-8u112-linux-x64.tar.gz... 100% (174.7 MB of 174.7 MB)
              Successfully downloaded JDK distribution to /var/lib/ambari-server/resources/jdk-8u112-linux-x64.tar.gz
              Installing JDK to /usr/jdk64/
              Successfully installed JDK to /usr/jdk64/
              Downloading JCE Policy archive from http://public-repo-1.hortonworks.com/ARTIFACTS/jce_policy-8.zip to /var/lib/ambari-server/resources/jce_policy-8.zip

              Successfully downloaded JCE Policy archive to /var/lib/ambari-server/resources/jce_policy-8.zip
              Installing JCE policy...
              Check JDK version for Ambari Server...
              JDK version found: 8
              Minimum JDK version is 8 for Ambari. Skipping to setup different JDK for Ambari Server.
              Checking GPL software agreement...
              GPL License for LZO: https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html
              Enable Ambari Server to download and install GPL Licensed LZO packages [y/n] (n)? y
              Completing setup...
              Configuring database...
              Enter advanced database configuration [y/n] (n)? y
              Configuring database...
              ==============================================================================
              Choose one of the following options:
              [1] - PostgreSQL (Embedded)
              [2] - Oracle
              [3] - MySQL / MariaDB
              [4] - PostgreSQL
              [5] - Microsoft SQL Server (Tech Preview)
              [6] - SQL Anywhere
              [7] - BDB
              ==============================================================================
              Enter choice (1): 3
              Hostname (localhost): master
              Port (3306): 
              Database name (ambari): 
              Username (ambari): 
              Enter Database Password (bigdata): //ambari123
              Re-enter password: //ambari123
              Configuring ambari database...
              Should ambari use existing default jdbc /usr/share/java/mysql-connector-java.jar [y/n] (y)? 
              Configuring remote database connection properties...
              WARNING: Before starting Ambari Server, you must run the following DDL directly from the database shell to create the schema: /var/lib/ambari-server/resources/Ambari-DDL-MySQL-CREATE.sql
              Proceed with configuring remote database connection properties [y/n] (y)? 
              Extracting system views...
              ambari-admin-2.7.3.0.139.jar
              ....
              Ambari repo file contains latest json url http://public-repo-1.hortonworks.com/HDP/hdp_urlinfo.json, updating stacks repoinfos with it...
              Adjusting ambari-server permissions and ownership...
              Ambari Server 'setup' completed successfully.

* Finally, ensure that the MySQL driver is available to Ambari and the Hadoop cluster.On the node running Ambari server, issue the 
following command 

              ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar

* Now when everything is setup, use the following command on eachnode (including master) to start `ambari-agent`:

              ambari-agent start

* Use the command to start `ambari-server` on master:

              ambari-server start;

* To check the status of `ambari-server` and `ambari-agent`, use the commands:

              ambari-server status
              ambari-agent status

* Once server is `On`, it will show that the server is running on port `8080`, then use the following URL on browser to open `Ambari-UI`

              http://192.168.YY.XXX:8080

* Where `192.168.YY.XXX` is the static IP address of the master node.
* use the default username and password `admin/admin`.

## 15 Issues
Following are the issues that I faced during the complete setup of Ambari Multi-Node Server.
### 1. MySQL Asking to reset password using (ALTER USER) Command
One of the issues that I faced during the cluster setup is that when MySQL is installed for the first time, and I logged in with the root user using the `mysql -u root -p` command, it will not execute any query or command by displaying the error: `Change the password using ALTER USER command`. to fix this issue, I used this [link](https://stackoverflow.com/questions/33467337/reset-mysql-root-password-using-alter-user-statement-after-install-on-mac). so basically all you have to do is to change password using the command: 

              ALTER USER 'root'@'localhost' IDENTIFIED BY 'ambari123';

So after using the above command I was able to execute any queries or commands on MySQL.

### 2. MySQL version is not Supported by Ambari
Initially the error that I faced was that when I was creating the ambari database and running the given DDL file, the queries would execute but some of them will give an error showing that there is some kind of dependency or a `FOREIGGN_KEY_CONSTRAINT` error. I looked up in the internet and found this [Link](https://community.cloudera.com/t5/Support-Questions/Problem-setup-ambari-2-7-1-0-with-mysql-8-0-13/td-p/234272). It was told that the `MySQL v8` is not supported by `Ambari 2.7.3.0`, so after that I user the `MySQL v5.7` for the setup, and the issue was fixed.

### 3. Ambari Server Setup Credentials Issue
### 4. Setup of Password-less SSH
### 5. Changing MySQL Password Policy

## 16 References and Links
Help of the following links were helpful during the deployment of the server:

* [Configuring Static-IP on CentOS7](https://www.techrepublic.com/article/how-to-configure-a-static-ip-address-in-centos-7/)
* [Concept Tutorial on Basic Multi-node Cluster](https://www.youtube.com/watch?v=RWzJoTWv9M4&feature=youtu.be)
* [Tutorial on Configuring Static-IP on CentOS7](https://www.youtube.com/watch?v=vI-3isexLrQ)
* [Clouder Documentation on Ambari Installation](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/ch_Getting_Ready.html)
* [Setup Password-less SSH](https://www.itzgeek.com/how-tos/linux/centos-how-tos/ssh-passwordless-login-centos-7-rhel-7.html)
* [IBM Docs on Configuring MySQL](https://www.ibm.com/support/knowledgecenter/SSPT3X_4.2.5/com.ibm.swg.im.infosphere.biginsights.install.doc/doc/bi_install_ranger_db.html)
* [Starting the server and agents](https://github.com/vennilasundar/Step-By-Step-Process-for-Cluster-Implementation-/blob/master/Download%20%26%20Install%20Ambari)
* [Java Installation on CentOS7](https://phoenixnap.com/kb/install-java-on-centos)
* [Ambari Agent Installation](https://docs.cloudera.com/HDPDocuments/Ambari-2.5.1.0/bk_ambari-administration/content/install_the_ambari_agents_manually.html)
* [Creating User in MySQL](https://docs.cloudera.com/HDPDocuments/Ambari-2.7.3.0/administering-ambari/content/amb_using_ambari_with_mysql_or_mariadb.html)
* [Find JDBC Path on CentOS7](https://unix.stackexchange.com/questions/236286/how-to-find-the-path-of-jdbc-driver-in-centos)
* [Using ALLTER USER command to update password](https://stackoverflow.com/questions/33467337/reset-mysql-root-password-using-alter-user-statement-after-install-on-mac)
* [Check Current Logged-In user in MySQL](https://stackoverflow.com/questions/28667890/check-current-user-in-mysql-command-line/28667933)
* [MySQL SOURCE command path error fix](https://stackoverflow.com/questions/14684063/mysql-source-error-2)
* [Check Ambari-server log file](https://community.cloudera.com/t5/Support-Questions/REASON-Ambari-Server-java-process-has-stopped-Please-check/td-p/194615)
* [Completely Clean and Remove ambari for fresh install](https://community.cloudera.com/t5/Support-Questions/How-to-Completely-Clean-Remove-or-Uninstall-Ambari-for-Fresh/td-p/95114)
* [Installation of MySQL v5.7](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-centos-7)
* [Issue with the MySQL v8 not supported by Ambari 2.7.3.0](https://community.cloudera.com/t5/Support-Questions/Problem-setup-ambari-2-7-1-0-with-mysql-8-0-13/td-p/234272)
* [Changing MySQL Password Policy Level](https://tecadmin.net/change-mysql-password-policy-level/)
* [Installation of JDBC Driver on CentOS7](https://linuxadminonline.com/how-to-install-mysql-jdbc-driver-on-centos-7/)



