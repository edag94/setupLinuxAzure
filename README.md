# Setting up a Linux Develop environment

For setting up linux env, you have 2 options:

1. WSL
2. Using a Linux VM in Azure

Using WSL may suffice for some projects, but cannot provide the same experience that a tradition Linux virtual machine can provide, and therefore there are limitations (i.e. GPU support in WSL is still in preview mode).

This documents aims to introduce a developer new to Linux development on how to setup a Linux VM on Azure as a development environment

## Provisioning the Virtual Machine

This process is the same as provisioning any type of virtual machines. The following are the steps to refresh your memory, as well as includes troubleshooting tips and recommended configs.

1. Create a new resource group.

2. Navigate to the [Azure Portal](https://ms.portal.azure.com/) and click the + button to create a new resource. Either type in "virtual machine" or click on "Compute" and you should see the Virtual Machine option. Choose this option and a new blade to configure a VM should show up.
    - Note: If you know what virtual machine image you would like to use (i.e. Ubuntu Server 18.04), you can directly search for this an a template for that image will show up, saving you some configuration.
    - **Recommended Image:** [Data Science Virtual Machine- Ubuntu 18.04](https://ms.portal.azure.com/#create/microsoft-dsvm.ubuntu-18041804)

3. Use the resource group in the subscription you created earlier. Identify the machine size that you need to use (if you don't know start with one that is recommended by the VM image publisher). Depending on the machine size that you need to allocate, you may be restricted to a certain set of regions, depending on availability and quota. 
    - If you are not sure if your machine will be available, it's a good idea to check the subscription quota. 
        1. Search for your subscription name in the top search bar, and select it. Once you are on the subscription pane, navigate to the "Usage + Quotas" tab under "Settings" on the left hand side. 
        2. At the top of the "Usage + Quotas" pane, there will be 4 boxes. The second from the left will say "Select a Provider", from which you will chose "Microsoft.Compute". You may also chose the region if you are interested in a set of regions.
        3. Here you will see a list quotas for each category of machine in a particular region. Based on this, you can decide what region your VM size will be available in. If the quotas don't match your needs, you can check the other "Project Vienna Personal *" subs for their quotas.
    - To see all the sizes available in a particular region, click on "Select size" under the Size dropdown. You can use the filters at the top to help find your exact machine.

4. Select "SSH public key" for Authentication Type and "Generate new key pair" for SSH public key source. This key will be available to download upon allocation of the machine.

5. Open ports 22 (ssh) and 3389 (RDP) on the networking tab

6. We will allocate a data disk later, as it is easier to discover and setup after the machine is created. A data disk is recommended as the VMs have one temp disk which will not be saved across reallocations, and the OS disk. In theory, its fine to save everything to the OS disk as this will persist for the lifetime of the resource, but using a Data disk gives you more flexibility, as you can unmount the disk and move it to another VM, as well as choose a disk capacity that fits your needs. 

7. The rest of the settings can be left as is. When you are ready, you can click on "Review + Create" to create the resource.

## Connecting to the Virtual Machine

By default, Linux servers do not come with a GUI. Therefore, the only way to interface with the machine initially is via SSH. You can install a GUI for future sessions if preferred after you first access the machine via SSH, which will be outlined in a later section.

### Connecting via SSH

Navigate to the VM in azure portal, and click on the "Connect" tab in the left pane. Add the directory of the SSH key that was generated when the VM was allocated (most likely in your Downloads folder). Copy the ssh command, and run in powershell terminal. If successful, you should get a "Welcome to Ubuntu" message!

#### Setup VSCode for SSH

1. Get the "Remote - SSH" extension
2. Open command palette (Ctrl + shift + p) and type "Remote - SSH: Open Configuration File..."

3. setup .ssh file like this:  

```yml
Host <choose a name for your config>
  HostName <your VM IP>
  IdentityFile <your ssh key file location>
  User <your VM Username>
```

4. Run command "Remote - SSH: Connect to Host..." and select the configuration you just created.

5. You should now be connected to VM. Navigate to the file explorer and you should have the option to open a folder. 

6. You can now open folders on the remote machine just like you would be able to do on your windows machine locally! Terminals are also ran remotely on your host machine, try running some bash commands.

### Connecting via xRDP

1. Since the machine uses SSH and RDP can only use password, you need to set a password for the admin user you specified upon creation (if you forgot admin user name its in the connect tab). To set password execute: 
```sh
sudo passwd "<myusername>"
```
2. install xRDP
3. install xfce4
4. open port 3389 (RDP port)
5. Go to connect in VM blade and download RDP file.

## Setup Data Disk

Once you've connected to your VM, its time to attach a data disk.
 
Follow the instructions in the link below:
https://docs.microsoft.com/en-us/azure/virtual-machines/linux/attach-disk-portal

Once you have created the disk, you may run into permissions issues on the disk, and may have to use "sudo" with every command.  
To fix this and allow the current user to have permissions on the disk, run the following command:  
```sh
sudo chown -R <your_unix_username> /<your_datadisk_mountpoint>
```