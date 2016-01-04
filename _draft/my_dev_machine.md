# My Dev Machine

## VirtualBox
## Choose OS
Call OS that will be installed as Virtual OS as VOS.

## Install VOS

## Install software
### Install VirtualBox Guest Additions
Why we should install Guest Additions?

First of all lets switch to root user.
``` bash
    su
    # enter root password
```

Under root user we can perform upgrade of OS and install additional packages:
``` bash
    apt-get update
    apt-get upgrade
    apt-get intall sudo mc synaptic top htop linux-headers-$(uname -r) gcc g++ make git
```

After this commands I recommend to reboot your VOS to "apply changes" in OS.

After VOS reboot in VirtualBox menu choose Devices -> Insert Guest Additions CD image... (don't forget
to check if virtual CD-drive of your Virtual OS is empty). VirtualBox will try to
mount its "Guest Additions CD" into VOS CD-drive. 

If your VOS is **Ubuntu**, then you will see mount dialog (add picture) with question should the VOS run
autorun.sh file on CD. Just press Run. After that Terminal window will be opened and
installation of VirtualBox Additions will be started (maybe at first you will be asked
for admin password) (add picture).

If your VOS is **Debian**, then find out where "Guest Additions CD" was mounted. I recommend to
check this folders:
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

After successful installation of VirtualBox Guest Additions reboot your OS. If after reboot of VOS its screen
size will be adapted to the size of VirtualBox window, then Guest Additions work properly.

#### Set up shared folders

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
