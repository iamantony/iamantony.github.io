# AOSP Development Part X. How to debug platform code

## Introduction

## Links to all parts

In previous parts:
1. [] AOSP Development Part 1 - Set up (download aosp, build images)
2. [] AOSP Development Part 2 - Android Studio (installation, usage, emulator, simple app)
3. [X] AOSP Development Part 3 - Debuggin platform code

## Steps

http://ronubo.blogspot.ru/2016/01/debugging-aosp-platform-code-with.html
https://shuhaowu.com/blog/setting_up_intellij_with_aosp_development.html

### Version of software
AOSP -
Android Studio - 
...

1. Build eng (engeneer) images
2. Build idegen

```
mmma development/tools/idegen
```

3. Run idegen script from root dir of aosp

```
development/tools/idegen.sh
```

What we will get (iml and ipr files)? What they contain?

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
