新版本方案：
adb root
adb remount
adb reboot
adb root
adb remount
adb push adb /system/bin/
adb shell
adb devices         // 看看是否有A123456的设备
adb pull /data/log /data/subscreen-log
exit
adb pull /data/subscreen-log subscreen-log



命令行升级流程：(假设OTA包名为NP511-NP512_firmware_1.0.9.zip)
adb mkdir /data/ota
adb push NP511-NP512_firmware_1.0.9.zip /data/ota/update.zip
adb shell ota_test
adb reboot