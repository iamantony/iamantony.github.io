# AOSP Development Part 3. How to debug platform code

## Links to all parts

In previous parts:
1. Set up (download aosp, build images)
2. Android Studio (installation, usage, emulator, simple app)
3. Debuggin platform code (you are reading it)

## Introduction

Next step is ...

I will base my article on these blog posts:
- http://ronubo.blogspot.ru/2016/01/debugging-aosp-platform-code-with.html
- https://shuhaowu.com/blog/setting_up_intellij_with_aosp_development.html

## Instructions

These instructions were written using the following software

- AOSP: 6.0 on branch ...
- OpenJDK: 1.8.0
- Android Studio: 2.3.3

Here we go!

1. Build AOSP eng (engeneer) images

``` bash
cd /path/to/aosp

# remove previous build
rm -r ./out

# setup aosp environment
. build/envsetup.sh

# choose aosp_arm64-eng for emulator or '-eng' version of configuration of your device
lunch

# build aosp
make -j4
```

2. Build idegen

After successful build of images, we can move to the next step - preparation of
project file for Android Studio. To generate it we need special tool - idegen.
Let's build it.

``` bash
mmma development/tools/idegen
```

3. Run idegen script

idegen.sh creates special .ipr and .iml files that contain information
about aosp-based project: list of source folders, which folders should be excluded
from project, path to libraries and other settings. These files could be interpreted by
Android Studio and Eclipse as project files. I would also recommend you to read
development/tools/README file.

```
development/tools/idegen.sh
```

4. Edit config file of Android Studio

Basic settings of Android Studio were not supposed to work with such huge projects as AOSP.
So we need to edit them!....

4. Open Android Studio. Open iml file. IDE will start indexing files in aosp directory. It could take some time.
5. Open Project Settings.

In SDK tab:
Add new SDK with no libraries (add screenshot). Give it a useful name. Remove all entries in Classpath tab.

In Project tab:
Set which folders will be "Source folders" and which should be excluded. Example:
Source folders: frameworks, external, system, ...
Excluded folders: build, developers, out, ...

6. Set up brakepoints in some sources (PackageManagerService.java for example)
7. Load images to emulator or phone. Start device
8. Start monitor and wait for system_service process
9. Choose system_service and remember its port number
10. In Android Studio open Run -> Edit Configuration
11. Create new Remove config and set up port number. Click OK.
12. Start debugging in Android Studio. You should see message that debugger connected to port.
13. Copy some apk on device and try to install it.
