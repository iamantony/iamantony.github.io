---
title: AOSP Development: how to debug platform code
number: X
---

## Intro

This article is intended for developers, who decided to explore Android world
in more detail. It is assumed that you have already downloaded AOSP sources
and tried to build images for emulator or Nexus-device. 

# TODO: rewrite
If you want to know
how Android works inside, debugging of platform code will help you a lot.
But first of all I want to say many thanks to [ronubo][ronubo-site] and
[shuhaowu][shuhaowu-site]!

## Prepare environment

These instructions were written on machine with following software:

- Ubuntu 15.10 x64
- AOSP: 6.0 revision r67
- OpenJDK: 1.8.0
- Android Studio: 2.3.3

1. Let's start with build of AOSP eng (engeneer) images

    ``` bash
    cd /path/to/aosp

    # remove previous build
    rm -r ./out

    # setup aosp environment
    . build/envsetup.sh

    # choose aosp_arm64-eng for emulator or '-eng' version of configuration of your device
    lunch

    # build aosp. Use -j[number_of_cpu_cores_to_use] to speed up build process.
    make -j4
    ```

2. Build idegen

    After successful build of images, we can move to the next step - preparation of
    project file for Android Studio. AOSP contains usefull tool for this purpose - `idegen`.

    ``` bash
    mmma development/tools/idegen
    ```

3. Run idegen script

    idegen.sh creates special `.ipr` and `.iml` files that contain information
    about aosp-based project: list of source folders, which folders should be excluded
    from project, path to libraries and other settings. `".ipr"` file could be interpreted by
    Android Studio as project files. I would also recommend you to read
    `development/tools/README` file.

    ``` bash
    development/tools/idegen/idegen.sh
    ```

    Check that two new files appeared in the AOSP root: `android.ipr` and `android.iml`.

4. Edit Android Studio config files

    Before opening AOSP project in Android Studio, we need to edit settings of IDE.
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

    Do not forget to restart Android Studio to apply changes in configuration!

5. Load AOSP project in Android Studio.

    In Android Studio go to menu `File -> Open` and choose file `/path/to/aosp/android.ipr`.
    IDE will start indexing files in aosp directory. It could take some time, so be patient.

6. Open Project Settings.

    In Android Studio go to menu `File -> Project Structure ...`. 

    Choose `SDKs` entry on the left side of the opened window. Add new JDK entry.
    Set up home path to JDK (in my case it is folder `/usr/lib/jvm/java-1.8.0-openjdk-arm64`).
    Give some meaningful name to new SDK (for example `1.8_debug`).
    Remove all entries in tab `Classpath`.

    [Insert as-project-sdk.png]

    Next choose `Project` section. Setup `Project SDK` to newly created SDK.

    [Insert as-project-project.png]

    Finally choose `Modules` section. Here you will see name of opened project - aosp?? - and
    its sources tree. By default most of the files in aosp folder would be imported to
    our project. But if we are going to debug only platform code, we would need only some of them.
    Fortunately we can mark such folders as `Sources` and all others as `Excluded`.

    `Sources` folders: frameworks, external, google, system, libcore, out/target/common/R

    [Insert as-project-modules.png]


## Debug AOSP platform

1. Set up brakepoints in some sources (PackageManagerService.java for example)

2. Load images to emulator or phone. Start device

3. Start monitor and wait for system_service process

4. Choose system_service and remember its port number

5. In Android Studio open Run -> Edit Configuration

6. Create new Remove config and set up port number. Click OK.

7. Start debugging in Android Studio. You should see message that debugger connected to port.

8. Copy some apk on device and try to install it.

[ronubo-site]: http://ronubo.blogspot.ru/2016/01/debugging-aosp-platform-code-with.html
[shuhaowu-site]: https://shuhaowu.com/blog/setting_up_intellij_with_aosp_development.html
[google-as-config]: https://developer.android.com/studio/intro/studio-config.html
[intellij-as-config]: https://intellij-support.jetbrains.com/hc/en-us/articles/206544869-Configuring-JVM-options-and-platform-properties
[jvm-opts-explained]: https://github.com/FoxxMD/intellij-jvm-options-explained
