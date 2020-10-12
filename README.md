# Multinode Ambari Cluster on CentOS7s

This is a complete guide on how to setup a multi-node ambari cluster on CentOS7 using VirtualBox. The cluster is developed to design and 
implemenet Bigdata Solutions and provide all the services that are required to serve the purpose.

> ### Downloading Required Stuff:
> 1. [CentOS7 ISO Image](https://www.centos.org/download/)
> 2. [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

Now open virtual box and click on the **new** button in order to create a new VM (Virtual Machine).

> ### Creating a VM using VirtualBox
> 1. Click on the new button.
> 2. Add the name of the VM as per your choice.
>   1. Type: Linux
>   2. Version: Red-Hat (64 bit)
> 3. Assign memory according to the current Memory installed on your system. (4GB Recommended)
> 4. Select **Create a virtual hard disk now** option
> 5. Select **VDI (Virtualbox Disk Image)**
> 6. Select **Dynamically Allocated** 
> 7. Assign the storage according to the available storage on your system. (Atleast 8GB)
> 8. Creating Adapters
>   1. Goto File > Host Network Manager
>   2. There are 3 buttons shown namely: **create, remove** and **properties**
>   3. Create 2 Adapters and name them **vboxnet0** and **vboxnet1**
> 9. Now go back to the VirtualBox and select the VM that is just created and click on the **settings** button.
> 10. Goto **Adapter** tab.
>   1. Under **Adapter2** tab, select the Enable Network Adapter checkbox and assign vboxnet0.
>   2. Goto **Adapter3** tab, select the Enable Network Adapter checkbox and assign vboxnet1.
> 11. Now goto **Storage** tab in settings and Select the **empty** disk. on the right side, add the CentOS7 ISO Image.
> 12. Click **OK** and you are done.
> 13. Now run the VM. 