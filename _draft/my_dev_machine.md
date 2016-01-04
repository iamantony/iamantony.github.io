# My Dev Machine

## VirtualBox
## Choose OS
Call OS that will be installed as Virtual OS as VOS.

## Prepare software

First of all lets switch to root user.
``` bash
    su
    # enter root password
```

Under root user we can perform upgrade of OS and install additional packages:
``` bash
    apt-get update
    apt-get upgrade
    apt-get intall sudo mc synaptic gcc g++ git
```

In VirtualBox menu choose Devices -> Insert Guest Additions CD image... (don't forget
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
(add image)

Run this command to install VirtualBox Guest Additions:
``` bash
sudo sh ./VBoxLinuxAdditions.run
```

It is very likely that installation will go smoothly and you will see something like this:
(add image)

If during installation something went wrong, in program output you will get some hints what you need to do.
