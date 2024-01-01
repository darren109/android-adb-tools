将adb push到system/bin/
将libcrypto.so.3 libz.so.1 push到system/lib64

1. adb root
   adb remount
   adb push adb /system/bin
   adb push libcrypto.so.3 /system/lib64/
   adb push libz.so.1 /system/lib64/
2. adb shell进入android执行以下命令
   export HOME=/data/
   mkdir /data/.android
   echo 7531 > /data/.android/adb_usb.ini
   adb devices         // 看看是否有A123456的设备
   adb pull /data/log /data/subscreen-log


