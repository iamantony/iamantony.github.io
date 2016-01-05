# My Dev Machine
Why I need such dev machine?
What I will finally get?

## VirtualBox
What it is? 
Why I choosed VirtualBox? 
What about Docker? 
Docker on Windows?

## Choose OS
Call OS that will be installed as Guest OS as GOS, host OS - HOS.
Why I use Linux?

## Install VOS

## Install software
After successful installation of VOS, let's fill it up with usefull software. At the first step
I recommend to do basic stuff.

### Basic
First of all let's install *sudo* package:
``` bash
    # switch to root user
    su
    # enter root password
    
    apt-get install sudo
```

Then we should enable *sudo* for our main user:
``` bash
    usermod -aG sudo user_name
```

Update and upgrade your GOS and install basic software:
``` bash
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get intall mc synaptic top htop gcc g++ make git
```

### Install VirtualBox Guest Additions
Why we should install Guest Additions?

Let's install additional packages:
``` bash
    apt-get intall linux-headers-$(uname -r)
```

After this commands I recommend to reboot your GOS to "apply changes" in OS.

After GOS reboot in VirtualBox menu choose Devices -> Insert Guest Additions CD image... (don't forget
to check if virtual CD-drive of your Virtual OS is empty). VirtualBox will try to
mount its "Guest Additions CD" into GOS CD-drive. 

Find out where "Guest Additions CD" was mounted. I recommend to check this folders:
``` txt
/media/cdrom0
/home/user_name/media
/mnt
```

I found CD image mounted to "/media/cdrom0". It should contain this files:

![List of file in Guest Additions CD]({{ site.url }}/images/my_dev_machine/files_in_guest_additions_cd.png)

Run this command to install VirtualBox Guest Additions:
``` bash
sudo sh ./VBoxLinuxAdditions.run
```

It is very likely that installation will go smoothly and you will see something like this:

![Installation of Guest Additions]({{ site.url }}/images/my_dev_machine/installation_of_guest_additions.png)

If during installation something went wrong, in program output you will get some hints what you need to do
(for example, install missed packages).

After successful installation of VirtualBox Guest Additions reboot your GOS. If after reboot of GOS its screen
size will be adapted to the size of VirtualBox window, then Guest Additions work properly.

#### Set up shared folders
One of the advantages of the VirtualBox are shared folders. In HOS you can set some folders to be shared
so in GOS you will see them. You can share folders in read-only mode or give full access to them.
This is very useful feature. With it I can have only one folder with my projects. I create such folder
in HOS and share it with GOS so I can work with my projects files in both OS-s.

Let's share Projects folder with GOS (with the help of [this article][mount_shared_folder]):

1. In VirtualBox Manager open Settings of your GOS.
2. In "Shared folders" section add shared folder "Projects" (don't forget to set
"Auto-mount" option).

![Add Shared folder]({{ site.url }}/images/my_dev_machine/add_shared_folder.png)

3. Start your GOS and open Terminal.
4. Add your user to group "vboxsf" to see content of the shared folders:
    ``` bash
        sudo usermod -aG vboxsf user_name
    ```
    
5. Shared folder will be automatically be mounted to **/media/user_name/sf_Projects** or
**/media/sf_Projects**. Check that you can see content of shared folder.

6. For easy access / convenience, you may create a symbolic link to the mounted shared
folder in your home folder:
    ``` bash
        cd /home/user_name
        sudo ln -s /media/sf_Projects Projects
    ```

### Install last version of gcc

### Install Qt and QtCreator
#### Qt 4.8.6

#### Qt 5.5.1

### Install Boost

### Install Eclipse

#### Setup Eclipse
(Add link to settings files)

### Install Python
#### Python 2.7.X

#### Python 3.5.X

### Install PyCharm


[mount_shared_folder]: http://www.htpcbeginner.com/mount-virtualbox-shared-folder-on-ubuntu-linux/
