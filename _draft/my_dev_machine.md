# My Dev Machine

Представьте: вы купили новый компьютер, принесли его домой, подключили
все кабели, нажали кнопку Вкл и... он заработал! Поздравляю, ваше железо
работает! Теперь настало время установить операционную систему. Вы берете
CD-диск или флешку, вставляете в компьютер и начинаете процесс установки.
Быстро пробежавшись по этапам настройки времени и разметки диска Вы
наконец-то добираетесь до медленно-ползущего прогрессбара основного этапа
установки ОС. Вы откидываетесь в кресле, берете кружку с чаем/кофе и
начинаете думать... какой софт собираетесь установить.

"В этот раз точно все будет по-другому!" - говорите Вы себе.
Вы уже знаете, что сначала скачаете обновления для ОС, потом установите драйвера для периферии,
далее браузер, мессенджер, пара-тройка других полезных программ.
Потом дело доходит до инструментария разработчика: компилятор,
проверенные временем редакторы, любимые IDE. И, наконец, приходит очередь
установки игр.

Вы уверены, что установите только необходимое. Никакого мусорного софта!
Увы, но практика показывает, что со временем ваша ОС обрастает все большим числом программ
(нужных и не очень). "Эта программка понадобилась, чтобы отредактировать фотографию,
этот текстовый редактор был установлен "на попробовать", а эта программа... так,
откуда это вобще взялось?".

В какой-то момент меня такое положение дел стало раздрожать. Все это нагромождение
софта вселяет некоторую неуверенность в стабильность работы ОС. Кажется, что
она стала неповоротливой, медленной. Программы все время просят различных обновлений,
разрешений, появляется куча непонятных папок... Потом случается первое падения ОС.
Появляют робкие мысли "А не снести ли все нафиг и поставить чистую ОС?".

Мы бегаем по замкнутому кругу. Рано или поздно Вы переустановите ОС. В качестве причины может
быть окончательное захламление ОС, ее серьезный сбой или выход новой версии
самой операционки. И процесс захламления ОС начнется заново.

И тут я должен сказать что-то вроде "Но я знаю выход!". Нет, выхода я не знаю.
Программы будут устанавливаться и удаляться, появятся новые средства разработки,
вы будете экспериментировать с новым софтом (удачно или не очень), будут
устанавливаться обновления (удачно или не очень). Это закон жизни, если
хотите.

Но что-то же делать надо! Лично я для себя нашел следующее решение: выделить весь
самый необходимый софт и перенести его в отдельное место, в новую ОС.
У меня, как у разработчика, самым необходимым софтом является мой набор
инструментов - компилятор, несколько библиотек и IDE. Это устоявшийся
список программ, который редко претерпевает кординальные изменения.
Таким образом я пользуюсь двумя ОС - одна из них для работы, другая для
всего остального.

Можно установить эти две ОС на один HDD и переключаться между ними, каждый
раз перезагружая компьютер. Нельзя сказать, что это удобно. Можно под
каждую ОС использовать отдельный компьютер. Это решает проблему перезагрузок,
но удваивает количество компьютеров у вас дома.

Лучшим решением (по крайней мере для себя) я считаю использование гипервизора.
С его помощью можно создать виртуальную ОС, которая будет запускаться
как обычное приложение в вашей основной ОС. 

В эту виртуальную ОС я устанавливаю только программы, связанные с разработкой
и больше ничего. Только самое необходимое (вспоминается скетч [Джорджа Карлина][stuff])!
Такую виртуальную ОС я называю Виртуальной Операционной Системой Разработчика (ВОСР).
Это позволяет разделить миры "действительно полезного ПО" и всего остального и
перестать беспокоиться о состоянии основной ОС. Можно создать копию ВОСР и поместить
ее в надежное место. И даже если основная ОС скажет "пока", вы всегда сможете
за несолько минут восстановить весь свой настроенный набор разработчика, просто развернув
ВОСР из копии.

Если вам приглянулась моя идея ВОСР, то эта статья для вас! Я постараюсь подробно
описать процесс создания ВОСР, разделив его на логические этапы. Возможно, в какой-то
момент наши пути разойдутся, ведь инструментарий у каждого разработчика свой. 
Я, как разработчик С++ и немного Python, использую компилятор gcc, библиотеку Boost,
фреймфорк Qt 4.8.6 и 5.X, Python 2.7, 3.Х и среды разработки Eclipse, QtCreator, PyCharm.
Поэтому в этой статье я буду описывать настройку ВОСР, ориентируюясь на свои потребности.

Итак, пора начинать! Только с чего? Пожалуй, с системы виртуализации. Хм... Скажите,
какую ОС вы используете? Дайте угадаю. Это либо Windows, либо Linux, либо MacOS. 
Угадал? Я вот, к примеру, использую Windows и Ubuntu.
Я бы давно перешел полностью на Ubuntu, только на работе мы используем
корпоративную версию Windows, да и дома не все смогли освоиться в непривычном мире
Linux. Возможно, у Вас такая же ситуация. В этом случае нам необходимо
кросс-платформенное решение - VirtualBox.

## VirtualBox
[VirtualBox][vb_site] - is a hypervisor for x86 computers from Oracle Corporation.
Я не буду подробно описывать свойства этой системы. Возможно, Вы уже с ней
знакомы. Если нет, то вот вам [cсылка на статью в википедии][vb_wiki].

В чем преимущество VirtualBox или почему я выбрал именно это средство виртуализации?
Наверное, потомучто оно отвечает всем моим требованиям плюс имеет удобные фичи:
- кросс-платформенность. Я могу установить VirtualBox на любую ОС и запустить
на ней мою ВОСР.
- возможность переноса виртуальной ОС с одного компьютера на другой путем создания
образа.
- возможность создания снимков виртуальной ОС. Сначала я не пользовался этой
возможностью, но потом смог оценить ее достоинства. Снимки подобны сохранению
в игре. Допустим, Вы хотите установить новую версию библиотеки, но не уверены,
что все пройдет гладко. В этом случае вы делает снимок ВОСР, запоминая таким
образом ее текущее состояние. Затем спокойно экспериментируете с установкой
новой библиотеки. Если все получилось - замечательно! Если нет, то откатываете
состояние ВОСР до ранее сделанного снимка.
- возможность расшаривания папок.


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
    sudo apt-get intall mc htop screen gcc g++ make gdb git valgrind
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

Run this commands to install VirtualBox Guest Additions:
``` bash
mkdir vbox
cp -r /media/cdrom0/* ~/vbox
cd ./vbox
sudo sh ./VBoxLinuxAdditions.run
```

It is very likely that installation will go smoothly and you will see something like this:

![Installation of Guest Additions]({{ site.url }}/images/my_dev_machine/installation_of_guest_additions.png)

If during installation something went wrong, in program output you will get some hints what you need to do
(for example, install missed packages).

After successful installation of VirtualBox Guest Additions reboot your GOS. If after reboot of GOS its screen
size will be adapted to the size of VirtualBox window, then Guest Additions work properly.

Don't forget to remove temporary directory *vbox* from your home directory:
``` bash
sudo rm -r ~/vbox
```

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
    
7. Log out and log in to the system (or just reboot) and check that in your home directory
there is new a *folder* Projects and you can see it's content.

### Install Qt and QtCreator
#### Qt 4.8.6
[How to install Qt 4.8.6 on Linux.][qt486inst]

[How to configure Qt 4.8.6 on Linux.][qt486config]

``` bash
sudo apt-get intall sqlite3 postgresql pgadmin3 libpq-dev libcups2-dev libdbus-1-dev libx11-dev libxext-dev libicu-dev
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

http://www.boost.org/doc/libs/1_60_0/more/getting_started/unix-variants.html#or-build-custom-binaries

http://www.linuxfromscratch.org/blfs/view/svn/general/boost.html

http://ubuntuforums.org/showthread.php?t=1180792

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

Last command start Eclipse IDE. How to create desktop shortcut:

#### Setup Eclipse
(Add link to settings files)

### Install PyCharm
I love IDE from JetBrains and I think PyCharm is the best free IDE for Python.
You can download it [here][pycharmdownload] (choose Community Edition).

### Install Java
#### JVM

#### IntelliJ IDEA
Another IDE from JetBrains. This time for Java.
You can download it [here][ideadownload] (choose Community Edition and *.tar.gz* extension).

[stuff]: http://www.youtube.com/watch?v=MvgN5gCuLac
[vb_site]: https://www.virtualbox.org/
[vb_wiki]: https://en.wikipedia.org/wiki/VirtualBox
[mount_shared_folder]: http://www.htpcbeginner.com/mount-virtualbox-shared-folder-on-ubuntu-linux/
[qtsite]: http://www.qt.io/download-open-source/
[qt486inst]: http://doc.qt.io/qt-4.8/install-x11.html
[qt486config]: http://doc.qt.io/qt-4.8/configure-options.html
[boostdownload]: http://sourceforge.net/projects/boost/files/boost/
[eclipsedownload]:http://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/mars/1/eclipse-cpp-mars-1-linux-gtk-x86_64.tar.gz
[pycharmdownload]: https://www.jetbrains.com/pycharm/download/#section=linux
[ideadownload]: https://www.jetbrains.com/idea/#chooseYourEdition
