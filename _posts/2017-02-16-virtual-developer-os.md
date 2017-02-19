---
title: Virtual Developer OS
layout: default
number: 3
author:
  name: Antony Cherepanov
  url: {{ site.url }}
---

Every developer has set of software and tools that he uses for work or hobby. This set could contain favorite IDEs, packages, libraries and useful tools. Over time, this list of software grows. At some point the set gets on so large that developer can not hold all of it in his memory. That is how "developer memory dump" appears - text file with list of necessary software.

Time goes by. Developer gets new work, upgrade his (or her) PC... After several iterations of installation of tons of software on clean OS, developer asks himself - how he can optimize this routine? One of the answers is to use virtual machine. Main idea is simple: you set up virtual machine with OS and all required software only once and then use it on (almost) every computer. I call such virtual machine *Virtual Developer OS (VDOS)*.

Advantages of VDOS:
- set up all necessary software only once
- portability - image of virtual machine could be saved on flash drive
- do not depend on virtual OS reliability - with snapshots of virtual machine you could revert any dangerous changes
- do not depend on host OS reliability - all necessary/important software installed on virtual machine

Disadvantages:
- virtual machine could be run only on processors that support virtualization technology
- virtual machine could not use all power (CPU, RAM) of computer
- ...

In this post I will describe how to create VDOS. In the first part you will learn how to create virtual machine using VirtualBox. The second part will contain setup commands that I use when create my own VDOS.

I assume that host OS and virtual OS are Linux (Ubuntu).

## VDOS creation

### Install VirtualBox

[VirtualBox][vb_site] - is a hypervisor for PC from Oracle Corporation. Main advantage - it can work on Windows, Linux and other OSes.

1. [Download][vb_download] latest VirtualBox for your OS and VirtualBox Extension Pack.
2. Install VirtualBox:

   ``` bash
   cd /path/with/virtualbox/install/file
   sudo dpkg -i virtualbox.deb
   ```

3. Run VirtualBox.
4. Install VirtualBox Extension Pack. Open Menu -> File -> Preferences -> Extensions. Press "Add new package" button. In opened window choose VirtualBox Extension Pack file.

### Download OS image

As a VDOS we will use [Ubuntu][ubuntu-site]. I would recommend to use latest Desktop LTS release. You can download it [here][ubuntu-download].

### Create virtual machine in VirtualBox

1. Run VirtualBox.
2. Press "New" button.
3. *Name and operating system*: set name "UbuntuVDOS", type - Linux, version - Ubuntu (64-bit).
4. *Memory size*: set memory size - 2048 Mb.
5. *Hard disk*: choose "Create a virtual hard disk now".
6. *Hard disk file type*: choose VDI.
7. *Storage on physical hard disk*: choose "Dynamically allocated".
8. *File location and size*: choose path where VDOS image will be placed. Set size of the virtual hard disk >= 15 Gb.

As a result of these steps you will get empty VDOS with name "UbuntuVDOS". Choose it and press "Settings" button.

1. Choose General -> Advanced. Set parameter "Shared Clipboard" to "Bidirectional".
2. Choose System.
    * Motherboard tab. Set "Base memory" to 2048 MB. In "Boot order" untick "Floppy", put "Optical" on the first place and "Hard disk" on the second.
    * Processor tab. Set number of processors to 2 (or more). _**Be careful**_: if you want to use all cores of your CPU, don't forget to decrease "Execution Cap" to at least to 90%.
3. Choose Storage. In "Storage tree" choose virtual optical drive (Controller: IDE -> Empty). In "Attributes" section press button with CD and choose Ubuntu *.iso* file that you previously downloaded.
4. Press OK in "Settings" window.

### Install Ubuntu on virtual machine

Now we made all preparations for installation of Ubuntu on virtual machine. Choose "UbuntuVDOS" in VirtualBox and press "Start" button. Virtual machine will start to load and soon you will see Ubuntu installation window.

1. Choose "Install Ubuntu".
2. Check "Download packages during installation" and "Install third-party software...".
3. Choose "Erase disk and install Ubuntu".
4. Set time zone, landguage, user name and password.
5. After succeed installation restart VDOS.

## Setup VDOS: install software

After successful installation of VDOS, let's fill it up with useful software.

### Update packages

Open Terminal and type this commands:

``` bash
sudo apt-get update
sudo apt-get upgrade
```

### Install VirtualBox Guest Additions

With VirtualBox Guest Additions VDOS will work smoothly. Also this package will enable:
- shared clipboard
- drag-and-drop
- shared folders
- other useful stuff

First of all, let's install additional packages:

``` bash
sudo apt-get intall linux-headers-$(uname -r)
```

I recommend to reboot your VDOS after this command so changes in OS could be applied.

After VDOS reboot check that virtual CD-drive of your VDOS is empty. Then choose in VirtualBox menu: Devices -> Insert Guest Additions CD image. VirtualBox will try to mount its "Guest Additions CD" into VDOS CD-drive. After that you will see a small window that will offer you to run installation script. Press "Run" button.

It is very likely that installation will go smoothly. If during installation something went wrong, read program output - you will get some hints what you need to do (for example, install missed packages).

After installation eject installation CD from virtual CD-drive and reboot VDOS.

### Set up shared folders

One of the advantages of the VirtualBox is support of shared folders. It means that you can set some of the folders of your host OS to be shared with VDOS. You can share folders in read-only mode or give full access to them. This is very useful feature. For example, in host OS you can create folder with your projects and share it with VDOS. You would have only one copy of your projects and will be able to work with them in both OS-s.

Let's share "Projects" folder with VDOS (with the help of [this article][mount_shared_folder]):

1. Turn off your VDOS.
2. In VirtualBox Manager open Settings of your VDOS.
3. In "Shared folders" section add shared folder "Projects" (don't forget to set "Auto-mount" option).
4. Start your VDOS and open Terminal.
4. Add your user to group "vboxsf" to see content of the shared folders:
    ``` bash
    sudo usermod -aG vboxsf user_name
    ```

5. Log out from the VDOS and log in.
6. Shared folder will be automatically mounted to **/media/user_name/sf_Projects** or **/media/sf_Projects**. Check that you can see content of shared folder.
7. For easy access/convenience, you may create a symbolic link to the mounted shared folder in your home folder:

    ``` bash
    sudo ln -s /media/sf_Projects /home/user_name/Projects
    ```

### Create snapshot of VDOS

Before we start to install additional software to VDOS, I would recommend you to create snapshot of your VDOS. If something went wrong during software installation, you will be able to roll back any changes and start again. How to create snapshot:

- Turn off VDOS.
- In VirtualBox window go to the "Snaphots" tab and press button "Take a snapshot".

### Install basic packages

``` bash
sudo apt-get install mc htop synaptic screen gcc g++ \
make cmake gdb git subversion valgrind ubuntu-sdk \
icu-doc kdelibs5-data
```

### Install Qt and QtCreator

``` bash
sudo apt-get install qt4-bin-dbg qt4-default qt4-designer \
qt4-dev-tools qt4-doc qt4-doc-html qt4-linguist-tools \
qt4-qmake qt4-qmlviewer qt4-qtconfig lighttpd firebird-dev \
libmysqlclient-dev libpq-dev libsqlite0-dev libsqlite3-dev \
unixodbc-dev libxcb-doc libxext-doc qt-assistant-compat \
libqt4-sql-mysql libqt4-sql-psql libqt4-sql-odbc

sudo apt-get install qt5-default qt5-doc qt5-doc-html \
qt5-image-formats-plugins qt5-qmake \
qt5-qmake-arm-linux-gnueabihf qt5-style-plugins \
qttools5-dev-tools qtbase5-dbg qtbase5-dev-tools-dbg \
qtbase5-private-dev libqt5sql5-mysql libqt5sql5-psql \
libqt5sql5-odbc libqt5svg5-dev

sudo apt-get install qtcreator qtcreator-data \
qtcreator-dbg qtcreator-dev qtcreator-doc
```

### Install Boost

The simpliest way to get Boost is to install it from Ubuntu repository. Current available version - 1.58. It's quite old (04.2015), but stable and sufficient for most users.

``` bash
sudo apt-get install libboost-all-dev
```

If you want to use latest version of Boost, go to [Boost SourceForge page][boostdownload] and download latest stable release of Boost. Extract content of the archive to */opt* folder (for example, */opt/boost_X.X* where *X.X* is a version number) and read manuals about how to use it (and build, if necessary):
- [Official Boost guide][boostguide]
- [Installation guide from Linux From Scratch][lfsguide]
- [Installation guide from Ubuntu forums][ubuntuguide]

### Install PyCharm

I love IDEs from JetBrains and I think PyCharm is the best free IDE for Python. You can download it [here][pycharmdownload] (choose Community Edition). How to install PyCharm:

``` bash
cd /path/with/pycharm/archive

# Extract archive
tar -zxvf pycharm-archive.tar.gz

# As a result we will get pychram folder. Let's move it into /opt
sudo mv ./pycharm /opt

# Run PyCharm. You will see set up window. Don't forget to choose option "Create Desktop shortcut for all users"
sudo /opt/pycharm/bin/pycharm.sh

# Add path to PyCharm */bin* folder to $PATH variable :
# PATH="$PATH:/opt/pychram/bin"
nano ~/.profile

# Log out and log in into OS. In terminal try to execute:
pycharm.sh
```

### Install IntelliJ IDEA

Another IDE from JetBrains. This time for Java. You can download it [here][ideadownload] (choose Community Edition and *.tar.gz* extension).

Installation process of IntelliJ IDEA is the same as the installation of PyCharm:

``` bash
cd /path/with/idea/archive

# Extract archive
tar -zxvf idea-archive.tar.gz

# As a result we will get Idea folder. Let's move it into /opt
sudo mv ./idea /opt

# Run Idea. You will see set up window. Don't forget to choose option "Create Desktop shortcut for all users"
sudo /opt/idea/bin/idea.sh

# Add path to Idea */bin* folder to $PATH variable :
# PATH="$PATH:/opt/idea/bin"
nano ~/.profile

# Log out and log in into OS. In terminal try to execute:
idea.sh
```

### Install Google Chrome

Download Google Chrome package from [here][chrome-download].

``` bash
sudo apt-get install libindicator7 libappindicator1

cd /path/with/chrome
sudo dpkg -i google-sudochrome.deb
```

## Conclusion

Congratulations! Now you have your own VDOS! First of all, I would recommend you to create new snapshot of VDOS. If you want to transfer VDOS to another computer:
- Go to the VirtualBox Menu -> File -> Export Appliance and follow instructions.
- VirtualBox will create *.ova* file with your VDOS.
- Copy it to target computer.
- Install VirtualBox on target computer.
- Go to the VirtualBox Menu -> File -> Import Appliance and follow instructions.

[vb_site]: https://www.virtualbox.org/
[vb_download]: https://www.virtualbox.org/wiki/Downloads
[ubuntu-site]: https://www.ubuntu.com/
[ubuntu-download]: https://www.ubuntu.com/download/desktop
[mount_shared_folder]: http://www.htpcbeginner.com/mount-virtualbox-shared-folder-on-ubuntu-linux/
[boostdownload]: http://sourceforge.net/projects/boost/files/boost/
[boostguide]: http://www.boost.org/doc/libs/1_60_0/more/getting_started/unix-variants.html
[lfsguide]: http://www.linuxfromscratch.org/blfs/view/svn/general/boost.html
[ubuntuguide]: http://ubuntuforums.org/showthread.php?t=1180792
[pycharmdownload]: https://www.jetbrains.com/pycharm/download/#section=linux
[ideadownload]: https://www.jetbrains.com/idea/#chooseYourEdition
[chrome-download]: https://www.google.ru/chrome/browser/desktop/
