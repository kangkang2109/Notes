adb root
进入手机内核shell
adb shell 
adb remount 重新挂载，使系统有写的权限(手机刷机后第一次开机有保护措施，只有读权限，只有先获取root权限再remount，可以改成读写权限)
adb install /name.apk 安装