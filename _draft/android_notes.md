# Android notes (todo: change name of the article)

## Shell commands

List of the available commands in Android shell: https://developer.android.com/studio/command-line/adb.html

### Packages

Install package:

```
cd /path/to/package
pm install com.package.name
```

List of the packages:

```
pm list packages -f (https://gist.github.com/davidnunez/1404789)
cat /data/system/packages.xml (https://android.stackexchange.com/questions/8452/how-can-i-find-app-name-by-uid)
cat /data/system/packages.list
```

### Processes

Stop process:

```
am force-stop com.my.app.package
am kill com.my.app.package
```

