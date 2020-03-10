# APK反编译流程


----------

## 一、反编译代码 （dex2jar反编译 && jd-gui查看）

1. 将要反编译的apk文件重命名为zip格式并解压缩
2. 将classes.dex文件（存放了全部的java代码）拷贝到dex2jar解压后的根目录下（可选）
3. 进入dex2jar解压后的根目录，空白处 shift + 鼠标右键 打开cmd，执行命令：d2j-dex2jar.bat classes.dex
4. 在对应目录下会生成classes-dex2jar.jar文件
5. 用jd-gui.exe打开上边反编译出来的classes-dex2jar.jar文件

----------

## 二、反编译资源 （apktool反编译 && notepad++查看）

1. 将要反编译的apk文件放到apktool文件夹，打开cmd，执行命令：apktool d test.apk
2. 在当前目录下会生成一个test文件夹：
res文件夹下存放的是反编译出来的所有资源；
smali文件夹下存放的是反编译出来的所有代码；
AndroidManifest.xml则是经过反编译还原后的manifest文件。

----------

## 三、重新打包

1. apktool文件夹目录下打开cmd，执行命令：apktool b test -o new_test.apk
2. 在当前目录会生成一个新的new_test.apk文件：
3. 将准备好的签名文件放到apktool文件夹根目录（没有签名文件的话，通过Android Studio可以很简单的生成一个）
继续在cmd执行命令：
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore 签名文件名 -storepass 签名密码 待签名的APK文件名 签名的别名
4. 签名完成后，建议对APK文件进行一次对齐操作，这样可以使得程序在Android系统中运行得更快，对齐操作使用的是zipalign工具，该工具在<Android SDK>/build-tools/<version>目录下，需要将这个目录配置到系统环境变量当中才可以在任何位置执行此命令。继续在cmd中执行命令：
zipalign 4 new_test.apk new_test_aligned.apk
5. 执行成功后，会生成一个对齐后的new_test_aligned.apk文件
6. 最后可以通过如下命令验证apk签名是否成功：
jarsigner -verify -verbose -certs new_test_aligned.apk

----------
