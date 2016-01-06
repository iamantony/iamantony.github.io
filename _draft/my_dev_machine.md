# My Dev Machine

Представьте: вы купили новый компьютер, принесли его домой, подключили
все кабели, нажали кнопку Вкл и... он заработал! Поздравляю, ваше железо
работает! Теперь настало время установить операционную систему. Вы берете
CD-диск или флешку, вставляете в компьютер и начинаете процесс установки.
Быстро пробежавшись по этапам настройки времени и разметки диска Вы
наконец-то добираетесь до медленно-ползущего прогрессбара основного этапа
установки ОС. Вы откидываетесь в кресле, берете кружку с чаем/кофе и
начинаете думать... в какой последовательности вы будете устанавливать
на чистую ОС свой набор софта.

В этот раз точно все будет по-другому!
Вы уже знаете, что сначала скачаете обновления, потом установите драйвера для периферии,
далее... браузер, мессенджер, пара-тройка других полезных программ...
Потом дело доходит до инструментария разработчика: компилятор,
проверенные временем редакторы, любимые IDE. И, наконец, приходит очередь
установки игр.

Вы уверены, что установите только необходимые программы. Никакого мусорного софта!
Увы, но практика показывает, что со временем этот вид программ все-таки появляется
на вашем компьютере. Эта программка понадобилась, чтобы отредактировать фотографию,
этот текстовый редактор был установлен "на попробовать", а эта программа... так,
откуда это вобще взялось?

В какой-то момент меня такое положение дел стало раздрожать. Все это нагромождение
софта вселяет некоторую неуверенность в стабильность работы ОС. Все эти программы
просят различных обновлений, разрешений, появляется куча непонятных папок...
Потом случается первое зависание компьютера на пустом месте. Появляют робкие мысли
"А не снести ли все нафиг и поставить чистую ОС?".

Это замкнутый круг. В конечном итоге Вы переустановите ОС. В качестве причины может
быть окончательное захламление ОС, ее серьезный сбой или выход новой версии
самой операционки. И процесс захламления ОС начнется заново.

И тут я должен сказать что-то вроде "Но я знаю выход!". Нет, выхода я не знаю.
Программы будут устанавливаться и удаляться, появятся новые средства разработки,
вы будете экспериментировать с новым софтом (удачно или не очень), будут
устанавливаться обновления (удачно или не очень). Это закон жизни, если
хотите.

Я знаю только лишь способ как попытаться отгородить только самый важный, самый
необходимый софт от того вороха проблем с ОС, который рано или поздно
случится. Для меня, как для разработчика, самым важным софтом является
мой инструментарий: компилятор, набор фреймворков, библиотек и IDE.
И по аналогии с реальным набором инструментов (молоток, пласкогубцы и т.д)
мне хочется хранить свой набор инструментов разработчика в чистом месте.
В качестве этого чистого места я использую виртуальную ОС.

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


[stuff]: http://www.youtube.com/watch?v=MvgN5gCuLac
[mount_shared_folder]: http://www.htpcbeginner.com/mount-virtualbox-shared-folder-on-ubuntu-linux/
