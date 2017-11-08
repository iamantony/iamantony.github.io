---
title: AOSP Development: how to debug platform code
number: X
---

## Intro

This article is intended for developers, who decided to explore Android world
in more detail.

As a basis of this post I used articles by [ronubo][ronubo-site] and [shuhaowu][shuhaowu-site].

## Prepare environment

These instructions were written on machine with following software:

- Ubuntu 15.10 x64
- AOSP: 6.0 revision r67
- OpenJDK: 1.8.0
- Android Studio: 2.3.3
- Android SDK: 26

1. Download AOSP

    Use official guide: https://source.android.com/source/requirements

2. Let's start with build of AOSP eng (engeneer) images

    ``` bash
    cd /path/to/aosp

    # remove previous build
    rm -r ./out

    # setup aosp environment
    . build/envsetup.sh

    # choose aosp_arm-eng for emulator or '-eng' version of your device configuration
    lunch

    # build aosp. Use -j[number_of_cpu_cores_to_use] to speed up build process.
    make -j4
    ```

2. Build *idegen*

    After successful build of images, we can move to the next step - preparation of
    project files for Android Studio. AOSP contains usefull tool for this purpose - `idegen`.

    ``` bash
    mmma development/tools/idegen
    ```

3. Run *idegen* script

    idegen.sh creates special `.ipr` and `.iml` files that contain information
    about aosp-based project: list of source folders, folders to exclude,
    path to libraries and other settings. `.ipr` file could be interpreted by
    Android Studio as project files. I would also recommend you to read
    `development/tools/README` file for additional info.

    ``` bash
    development/tools/idegen/idegen.sh
    ```

    Check that two new files appeared in the AOSP root: `android.ipr` and `android.iml`.

4. Edit Android Studio config files

    Before opening AOSP project in Android Studio, we need to edit IDE settings.
    AOSP contains several thousands of files, so Android Studio would need more memory
    to load and index such huge project.

    Basic information about config files of Android Studio you can find [here][google-as-config]
    and [here][intellij-as-config].

    Open Android Studio, go to menu `Help -> Edit Custom Properties ...`. IDE will open file
    `idea.properties`. Add this line to the end of it:

    ```
    idea.max.intellisense.filesize=100000
    ```

    Next, go to the menu `Help -> Edit Custom VM Options ...`. Add these settings to the opened
    file `studio64.vmoptions`:

    ```
    -Xms1g
    -Xmx2g
    ```

    Where:

     - Xms[amount_of_memory] - initial memory allocation of the heap for the VM.
     - Xmx[amount_of_memory] - maximum memory allocation of the heap for the VM.

    If you want to know more about Android Studio config options, check out
    [intellij-jvm-options-explained project on Github][jvm-opts-explained].

    *__Don't forget__* to restart Android Studio to apply changes in configuration!

5. Load AOSP project in Android Studio.

    In Android Studio go to menu `File -> Open` and choose file `/path/to/aosp/android.ipr`.
    IDE will start indexing files in aosp directory. It could take some time, so be patient.

6. Open Project Settings.

    In Android Studio go to menu `File -> Project Structure ...`. 

    Choose `SDKs` section on the left side of the opened window. Add new JDK entry.
    Set up home path to JDK (in my case it is folder `/usr/lib/jvm/java-1.8.0-openjdk-arm64`).
    Give some meaningful name to new SDK (for example `1.8_debug`).

    [Insert as-project-sdk.png]

    Next choose `Project` section. Setup `Project SDK` to newly created SDK.

    [Insert as-project-project.png]

    Finally choose `Modules` section. Here you will see name of opened project - *android* - and
    its sources tree. By default most of the files in aosp folder would be imported to
    our project. But if we are going to debug only platform code, we would need only some of them.
    Fortunately we can mark such folders as `Sources` and all others as `Excluded`.

    `Sources` folders: external, frameworks, libcore, out/target/common, packages, system.
    
    `Excluded` folders: all others

    [Insert as-project-modules.png]


## Debug AOSP platform

What we will debug? Well, for example let's investigate process of package installation.

1. Open in Android Studio *PackageManagerService.java* file. You can use *Ctrl + Shift + N* hotkey to search files in project.

2. Find function *installPackageLI()* and place breakpoint at the start of it.

3. Open menu *Run -> Edit Configurations*

    3.1 Add new *Remote* configuration using *+* sign at the top left corner of the window
    
    3.2 Set up name of configuration (for example, *emulator_debug*) and set port to *8700*
    
    [Insert as-debug_config.png]
    
4. Start *monitor* utility that is shipped with Android SDK. For example in my system it is located at */home/username/Android/Sdk*:

    ``` bash
    cd /path/to/android_sdk/tools
    monitor &
    ```
    
    This command will start *Android Device Monitor* (ADM).

5. Start emulator or flash images to your device. To start emulator type in console with
the configured AOSP environment:

    ``` bash
    emulator
    ```

6. Switch to ADM and wait till device bootup.

7. Click the process you would like to debug (in our case it is *system_process*),
and you will see that there is a */ 8700* next to its debugging port.

    [Insert adm_with_device.png]

8. Switch to Android Studio and start debugging. If IDE successfully connected to
*system_process* on device, you will see in *Console* tab something like this:

    ``` text
    Connected to the target VM, address: 'localhost:8700', transport: 'socket'
    ```

9. Copy test apk on device and try to install it:

    ``` bash
    # on PC
    adb push /path/to/test.apk /data/local/tmp
    adb shell
    
    # on device
    pm install /data/local/tmp/test.apk
    ```
    
10. PROFIT!

[ronubo-site]: http://ronubo.blogspot.ru/2016/01/debugging-aosp-platform-code-with.html
[shuhaowu-site]: https://shuhaowu.com/blog/setting_up_intellij_with_aosp_development.html
[google-as-config]: https://developer.android.com/studio/intro/studio-config.html
[intellij-as-config]: https://intellij-support.jetbrains.com/hc/en-us/articles/206544869-Configuring-JVM-options-and-platform-properties
[jvm-opts-explained]: https://github.com/FoxxMD/intellij-jvm-options-explained
