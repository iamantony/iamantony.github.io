# My Dev Machine

Steps to prepare Virtual Developer OS (VDOS).

I would assume that as a Host OS we use Linux (Ubuntu) and as a VDOS - also Linux (Ubuntu).

## Install VirtualBox

[VirtualBox][vb_site] - is a hypervisor for x86 computers from Oracle Corporation. Main advantage - it can work on Windows, Linux and other OSes.

1. [Download][vb_download] latest VirtualBox for your OS.
2. Download VirtualBox Extension Pack.
3. Install VirtualBox:

   ``` bash
   cd /path/with/virtualbox/install/file

   # example VirtualBox .deb file: virtualbox-5.1_5.1.12-112440-Ubuntu-wily_amd64.deb
   sudo dpkg -i virtualbox-5.1_5.1.12-112440-Ubuntu-wily_amd64.deb
   ```

4. Run VirtualBox.
5. Install VirtualBox Extension Pack. Open Menu -> File -> Preferences -> Extensions. Press "Add new package" button. In opened window choose VirtualBox Extension Pack file.

## Download OS image

As a VDOS I use [Ubuntu][ubuntu-site]. I would recommend to use latest Desktop LTS release. You can download it [here][ubuntu-download].

## Create virtual machine in VirtualBox

1. Run VirtualBox.
2. Press "New" button.
3. Name and operating system. Set name "UbuntuVDOS", type - Linux, version - Ubuntu (64-bit).
4. Memory size. Set memory size - 2048 Mb.
5. Hard disk. Choose "Create a virtual hard disk now".
6. Hard disk file type. Choose VDI.
7. Storage on physical hard disk. Choose "Dynamically allocated".
8. File location and size. Choose path where VDOS image will be placed. Set size of the virtual hard disk - 20 Gb.

As a result of this steps you will get empty VDOS with name "UbuntuVDOS". Choose it and press button "Settings".

1. Choose General -> Advanced. Set parameter "Shared Clipboard" to "Bidirectional".
2. Choose System.
    * Motherboard tab. Check that "Base memory" is set to 2048 MB. In "Boot order" untick "Floppy", put "Optical" on the first place and "Hard disk" on the second.
    * Processor tab. Set number of processors to 2.
3. Choose Storage. In "Storage tree" choose virtual optical drive (Controller: IDE -> Empty). In "Attributes" section press button with CD and choose Ubuntu .iso file that you previously downloaded.
4. Press OK in "Settings" window.

## Install Ubuntu on virtual machine

Now we made all preparations for installation of Ubuntu on virtual machine. Choose "UbuntuVDOS" in VirtualBox and press "Start" button. Virtual machine will start to load and soon you will see Ubuntu installation window.

1. Choose "Install Ubuntu".
2. Check "Download packages..." and "Install third-party software...".
3. Choose "Erase disk and install Ubuntu".
4. Set time zone, landguage, user name and password.
5. After succeed installation restart VDOS.

## Install software

After successful installation of VOS, let's fill it up with usefull software. 

### Update packages

Open Terminal and type this commands:

``` bash
sudo apt-get update
sudo apt-get upgrade
```

### Install VirtualBox Guest Additions

With VirtualBox Guest Additions VOS will work smoothly. Also this package will enable:
- shared clipboard
- drag-and-drop
- shared folders
- other useful stuff

First of all, let's install additional packages:

``` bash
sudo apt-get intall linux-headers-$(uname -r)
```

I recommend to reboot your VOS after this command so changes in OS could be applied.

After VOS reboot check that virtual CD-drive of your VOS is empty. Then choose in VirtualBox menu: Devices -> Insert Guest Additions CD image. VirtualBox will try to mount its "Guest Additions CD" into VOS CD-drive. After that you will see a small window that will offer you to run installation script. Press "Run" button.

It is very likely that installation will go smoothly. If during installation something went wrong, read program output - you will get some hints what you need to do (for example, install missed packages).

After installation eject installation CD from virtual CD-drive and reboot VOS.

### Set up shared folders

One of the advantages of the VirtualBox are shared folders. It means that you can set some of the folders of your host OS to be shared with VOS. You can share folders in read-only mode or give full access to them. This is very useful feature. For example, in host OS you can create folder with your projects and then share it with VOS. You would have only one copy of your projects and will be able to work with them in both OS-s.

Let's share "Projects" folder with VOS (with the help of [this article][mount_shared_folder]):

1. Turn off your VOS.
2. In VirtualBox Manager open Settings of your VOS.
3. In "Shared folders" section add shared folder "Projects" (don't forget to set "Auto-mount" option).
4. Start your VOS and open Terminal.
4. Add your user to group "vboxsf" to see content of the shared folders:
    ``` bash
        sudo usermod -aG vboxsf user_name
    ```
    
5. Log out from the VOS and log in.
6. Shared folder will be automatically be mounted to **/media/user_name/sf_Projects** or **/media/sf_Projects**. Check that you can see content of shared folder.
7. For easy access / convenience, you may create a symbolic link to the mounted shared folder in your home folder:
    ``` bash
        sudo ln -s /media/sf_Projects /home/user_name/Projects
    ```

### Install Qt and QtCreator
#### Qt 4.8.6
[How to install Qt 4.8.6 on Linux.][qt486inst]

[How to configure Qt 4.8.6 on Linux.][qt486config]

```bash
sudo apt-get intall mc htop screen gcc g++ make gdb git valgrind sqlite3 postgresql pgadmin3 libpq-dev libcups2-dev libdbus-1-dev libx11-dev libxext-dev libicu-dev
sudo mkdir /opt/qt-4.8.6
cd /opt/qt-4.8.6
sudo wget http://download.qt.io/archive/qt/4.8/4.8.6/qt-everywhere-opensource-src-4.8.6.tar.gz
sudo tar xvf qt-everywhere-opensource-src-4.8.6.tar.gz
cd qt-everywhere-opensource-src-4.8.6
sudo ./configure -v -debug -opensource -shared -qt-zlib -qt-libpng -qt-libtiff -qt-libjpeg -dbus -cups -nomake examples -nomake demos
```

After configuration Qt will show you which modules it will support (SQLite, ALSA, zlib and so on).
If you want to *enable* some module, try to install additional libraries and run again configure
procedure. To build and install Qt 4.8.6 run this commands:

``` bash
sudo make
sudo make install
```

Qt 4.8.6 will be installed into */usr/local/Trolltech/Qt-4.8.6/*

#### Qt 5.5.1
Go to [Qt site][qtsite] and download online installer. Then in terminal:
``` bash
cd /path/to/downloaded/file
sudo chmod u+x qt-unified-linux-*.run
sudo ./qt-unified-linux-*.run
```

After that you'll see GUI window of Qt Installer. Log in to the qt.io (or create new user),
choose path to install (I set default - /opt/Qt), choose Qt versions that you want to use
and additional components. Then installer will download all components and that's all.

After installation of desired Qt versions, run QtCreator (find it's shortcut in OS menu),
check the settings (don't forget to check that all installed Qt versions are available) and
try to compile some simple project to check that all is working fine.

### Install Boost
The simpliest way to get Boost is to install it from Debian repository. In Debian 8 there is
available Boost 1.55. It's quite old version of library (11.2013), but stable and sufficient
for most users.
``` bash
sudo apt-get install libboost-all-dev
```

If you want to use latest version of Boost, go to [Boost SourceForge page][boostdownload] and
download latest stable release of Boost. Extract content of the archive to some folder
(for example, */opt/boost_1.X*) and read manuals about how to use it (and build, if necessary):

[Official Boost guide][boostguide].

[Installation guide from Linux From Scratch][lfsguide].

[Installation guide from Ubuntu forums][ubuntuguide].

### Install Eclipse
Eclipse is a very powerful IDE with many features, outdated UI and tricky project
configuration process. I usually use it for plain C++ projects. Try it. Maybe you'll
like it. You can download installer archive [here][eclipsedownload].

``` bash
sudo apt-get install openjdk-7-jdk libcanberra-gtk-module
sudo mkdir /opt/eclipse
cd /opt/eclipse
sudo cp /path/to/eclipse-*.tar.gz /opt/eclipse
sudo tar xvf eclipse-*.tar.gz
sudo cd eclipse
sudo ./eclipse
```

Last command will start Eclipse IDE. And that is all about Eclipse installation. We just
extracted content of the archive to desired folder. Pretty easy. But there is one problem.
In OS applications menu you will not find shortcut for Eclipse. You'll not find it anywhere,
because it were not created. Let's fix it.

#### Eclipse desktop shortcut
Debian 8 use GNOME 3 as default desktop environment. And by default in GNOME 3 desktop
shortcuts are disabled. So our first step - enable desktop shortcuts. In applications
menu find Tweak Tool and run it. Got to the *Desktop* section and turn on slider
*Icons on Desktop*. Here is the short video guide on [Youtube][shortcutenablevideo].

Next, start *Terminal* and type:
``` bash
cd Desktop
touch eclipse.desktop
nano eclipse.desktop
```

We have created empty desktop shortcut with name *eclipse.desktop*. By last command
we opened this file in *nano* text editor. Copy this text to the opened file:
```text
[Desktop Entry]
Name=Eclipse CDT
Comment=Start Eclipse IDE
TryExec=/opt/eclipse/eclipse
Exec=/opt/eclipse/eclipse
Icon=/opt/eclipse/icon.xpm
Type=Application
```

Save changes and exit *nano*. Make this desktop shortcut executable:
``` bash
chmod ugo+x eclipse.desktop
```

After that on your desktop there should be the new shortuct for Eclipse IDE with
official icon. Try to launch it.

#### Setup Eclipse
(Add link to settings files)

### Install PyCharm
I love IDE from JetBrains and I think PyCharm is the best free IDE for Python.
You can download it [here][pycharmdownload] (choose Community Edition).

Installation process of PyCharm is the same as the installation of Eclipse. Create
new folder */opt/pycharm*, extract content of the downloaded archive to this
folder and create desktop shortcut for PyCharm in *~/Desktop* folder. Text
for shortcut:
```text
[Desktop Entry]
Name=PyCharm CE 5.0.3
Comment=Start PyCharm IDE
TryExec=/opt/pycharm/pycharm-community-5.0.3/bin/pycharm.sh
Exec=/opt/pycharm/pycharm-community-5.0.3/bin/pycharm.sh
Icon=/opt/pycharm/pycharm-community-5.0.3/bin/pycharm.png
Encoding=UTF-8
Type=Application
```

### Install IntelliJ IDEA
Another IDE from JetBrains. This time for Java.
You can download it [here][ideadownload] (choose Community Edition and *.tar.gz* extension).

Installation process of IntelliJ IDEA is the same as the installation of PyCharm. Create
new folder */opt/ideaic*, extract content of the downloaded archive to this
folder and create desktop shortcut for IntelliJ IDEA in *~/Desktop* folder. Text
for shortcut:
```text
[Desktop Entry]
Name=IntelliJ IDEA CE
Comment=Start IntelliJ IDEA IDE
TryExec=/opt/ideaic/idea-IC-143.1821.5/bin/idea.sh
Exec=/opt/ideaic/idea-IC-143.1821.5/bin/idea.sh
Icon=/opt/ideaic/idea-IC-143.1821.5/bin/idea.png
Encoding=UTF-8
Type=Application
```

[vb_site]: https://www.virtualbox.org/
[vb_download]: https://www.virtualbox.org/wiki/Downloads
[ubuntu-site]: https://www.ubuntu.com/
[ubuntu-download]: https://www.ubuntu.com/download/desktop
[mount_shared_folder]: http://www.htpcbeginner.com/mount-virtualbox-shared-folder-on-ubuntu-linux/
[qtsite]: http://www.qt.io/download-open-source/
[qt486inst]: http://doc.qt.io/qt-4.8/install-x11.html
[qt486config]: http://doc.qt.io/qt-4.8/configure-options.html
[boostdownload]: http://sourceforge.net/projects/boost/files/boost/
[boostguide]: http://www.boost.org/doc/libs/1_60_0/more/getting_started/unix-variants.html
[lfsguide]: http://www.linuxfromscratch.org/blfs/view/svn/general/boost.html
[ubuntuguide]: http://ubuntuforums.org/showthread.php?t=1180792
[eclipsedownload]:http://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/mars/1/eclipse-cpp-mars-1-linux-gtk-x86_64.tar.gz
[shortcutenablevideo]: https://www.youtube.com/watch?v=hy3r8H39-aU
[pycharmdownload]: https://www.jetbrains.com/pycharm/download/#section=linux
[ideadownload]: https://www.jetbrains.com/idea/#chooseYourEdition
