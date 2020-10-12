# Multinode Ambari Cluster on CentOS7s

This is a complete guide on how to setup a multi-node ambari cluster on CentOS7 using VirtualBox. The cluster is developed to design and 
implemenet Bigdata Solutions and provide all the services that are required to serve the purpose.

> 1. ## Downloading Required Stuff:
>   1. [CentOS7 ISO Image](https://www.centos.org/download/)
>   2. [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

Now open virtual box and click on the **new** button in order to create a new VM (Virtual Machine).

> 2. ## Creating a VM using VirtualBox
>   1. Click on the new button.
>   2. Add the name of the VM as per your choice.
>       1. Type: Linux
>       2. Version: Red-Hat (64 bit)
>   3. Assign memory according to the current Memory installed on your system. (4GB Recommended)
>   4. Select **Create a virtual hard disk now** option
>   5. Select **VDI (Virtualbox Disk Image)**
>   6. Select **Dynamically Allocated** 
>   7. Assign the storage according to the available storage on your system. (Atleast 8GB)
>   8. Creating Adapters
>       1. Goto File > Host Network Manager
>       2. There are 3 buttons shown namely: **create, remove** and **properties**
>       3. Create 2 Adapters and name them **vboxnet0** and **vboxnet1**
>   9. Now go back to the VirtualBox and select the VM that is just created and click on the **settings** button.
>   10. Goto **Adapter** tab.
>       1. Under **Adapter2** tab, select the Enable Network Adapter checkbox and assign vboxnet0.
>       2. Goto **Adapter3** tab, select the Enable Network Adapter checkbox and assign vboxnet1.
>   11. Now goto **Storage** tab in settings and Select the **empty** disk. on the right side, add the CentOS7 ISO Image.
>   12. Click **OK** and you are done.
>   13. Now run the VM. 

After the creation of the VM is done, you can repeat step 2 in order to create as many nodes as you need for the cluster.
Note that the more nodes you create, the more resources will be consumed from your system. 
For a normal cluster, you need atleast **1 Master and 2 slaves**

> 3. ## Installation of CentOS7 on VM
>   The Installation is relatively simple.
>   1. Select the language.
>   2. Add the appropriate date and time.
>   3. Goto **Network and Hostname** and select all 3 adapters
>   4. Click on **Begin Installation**
>   5. After the Installation add the **Root Password**. (Creating a user is not that necessary)

After the Installation is done, the system will ask to reboot. after the reboot it will ask for the following credentials
1. username: **root**
2. password: **password that you just set in step 5**

after entering the following credentials, you are currently in the localhost.

> 4. ## Setup Static IP Address
>   It is important to setup a static ip address in order to access the node on the network.
>   1. To set a static IP address, use the following command to add configurations:
>           vi /etc/sysconfig/network-scripts/ifcfg-**enp0s8**
>       Note that **enp0s8 is the name of the adapter in my machine**
>   2. You will be prompted with the following contents of the file:
>           
>
>
>
>
>
>
>
>
>
>
>


