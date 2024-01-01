adb 移植到 Android 平台使用
-----------------------------------------------------------------------------------

### 1、工具源码

所需源码：

```
openssl:        git clone https://github.com/openssl/openssl.git
zlib-1.2.8:     git clone https://github.com/fmrico/zlib-1.2.8.git
android-tools:  git clone https://git.launchpad.net/~phablet-team/+git/android-tools

```

### 2、编译

编译我是使用的 Ubuntu16.04 android8.0 环境编译的。  
将下载的源码都放在 android 目录下

```
# souce build/envsetup.sh
# lunch xxx

```

1、编译 openssl 库

```
//cd 到openssl源码目录
# cd openssl-master/ 
# mkdir output
#./config no-asm -shared --prefix=$(pwd)/output
说明：no-asm    在交叉编译过程中不使用汇编代码代码加速编译过程。
     -shared   生成动态链接库。
     --prefix  指定安装编译生成文件的路径，如不指定则默认为当前目录。

//修改Makefile
//1、将根目录下Makefile中1723行 “CROSS_COMPILE=” 改为： “CROSS_COMPILE=aarch64-linux-android-”
//2、找到Makefile中有 “-m64” 的地方，全删了（共2处，1757/1758行）。

#make -j24

//编译过程中大概都是会报错，在报错信息中可以看到，链接交叉编译器.so系统库、.h头文件位置。源码版本位置不同，大致都是如下目录
prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/lib/gcc/aarch64-linux-android/4.9x

//将缺失的.h头文件、so库copy到上面目录，缺失的文件可以从NDK源码中找到。目录如下，如果不同可以在prebuilts目录下搜索缺失的.h文件，在arm64下都可以使用
prebuilts\ndk\r10\platforms\android-23\arch-arm64\usr\

#cp -r prebuilts/ndk/r10/platforms/android-23/arch-arm64/usr/include/* prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/lib/gcc/aarch64-linux-android/4.9.x/include/
#cp -r prebuilts/ndk/r10/platforms/android-23/arch-arm64/usr/lib/* prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/lib/gcc/aarch64-linux-android/4.9.x/

//然后再次编译
#make -j24

//编译完成后生成:libcrypto.so、libssl.so，拷贝到交叉编译器.so系统库位置,头文件拷贝到交叉编译器.h系统头文件位置
#cp libcrypto.so libssl.so ../prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/lib/gcc/aarch64-linux-android/4.9.x/
#cp -r include/* ../prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/lib/gcc/aarch64-linux-android/4.9.x/include/

```

2、编译 zlib-1.2.8 库

```
//cd 到源码目录
#cd zlib-1.2.8/

#mkdir output
#./configure --prefix=$(pwd)/output

//修改Makefile
//1、添加 TOOLSCHAIN=aarch64-linux-android-
//2、将 CC=gcc 改为 CC=$(TOOLSCHAIN)gcc
//3、将所有 gcc 改为 $(CC)  （共两处，30/31行）
//4、将 
       AR=ar
       RANLIB=ranlib
     改为：
       AR=$(TOOLSCHAIN)ar                                                                                                                                                                                                                                                                                                                                                                                                  
       RANLIB=$(TOOLSCHAIN)ranlib

//编译
#make -j24

//将so库copy到系统编译器目录
#cp libz.so ../prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/lib/gcc/aarch64-linux-android/4.9.x/

```

3、编译 adb

```
//cd 到源码
#cd android-tools/

#cp debian/makefiles/adb.mk core/adb/
#cd core/adb
#mv adb.mk Makefile

//修改Makefile
CC=aarch64-linux-android-gcc
VPATH+= ../adb
SRCS+= adb.c
SRCS+= console.c
SRCS+= transport.c
SRCS+= transport_local.c
SRCS+= transport_usb.c
SRCS+= commandline.c
SRCS+= adb_client.c
SRCS+= adb_auth_host.c
SRCS+= sockets.c
SRCS+= services.c
SRCS+= file_sync_client.c
SRCS+= get_my_path_linux.c
SRCS+= usb_linux.c
SRCS+= usb_vendors.c
SRCS+= fdevent.c

VPATH+= ../libcutils
SRCS+= load_file.c
SRCS+= socket_inaddr_any_server.c
SRCS+= socket_local_client.c
SRCS+= socket_local_server.c
SRCS+= socket_loopback_client.c
SRCS+= socket_loopback_server.c
SRCS+= socket_network_client.c

CPPFLAGS+= -DADB_HOST=1
CPPFLAGS+= -DHAVE_FORKEXEC=1
CPPFLAGS+= -DHAVE_SYMLINKS
CPPFLAGS+= -DHAVE_TERMIO_H
CPPFLAGS+= -I.
CPPFLAGS+= -I../adb
CPPFLAGS+= -I../include
CPPFLAGS+= -I../../../external/zlib

LIBS+= -fPIC -pie -fPIE -lc -lcrypto -lz -pthread

OBJS= $(SRCS:.c=.o)

#这里注意，下面缩进是tab键那种→空格，不是····的空格，不然编译会报错
all: adb
adb: $(OBJS)
		$(CC) -o $@ $(LDFLAGS) $(OBJS) $(LIBS)
clean:
		rm -rf $(OBJS) adb

//将 adb_auth_host.c 中以下代码注释掉，编译会报错
BN_copy(n, rsa->n);
pkey->exponent = BN_get_word(rsa->e);
ret = getlogin_r(username, sizeof(username));

//将 adb.h 中端口号修改，防止冲突
#if ADB_HOST_ON_TARGET
/* adb and adbd are coexisting on the target, so use 5038 for adb
 * to avoid conflicting with adbd's usage of 5037
 */
#  define DEFAULT_ADB_PORT 6038
#else
#  define DEFAULT_ADB_PORT 6037
#endif

//编译
#make -j24

```

### 3、使用

```
编译出 adb 文件push到 /system/bin
编译出 libcrypto.so.3 libz.so.1等库，把他们push到和/system/lib64即可。
注：如果运行时提示缺失so库，就在编译出来的文件中找到对应的push到 /system/lib64 下
 
这样就可以在Android目录下运行adb，与从机通信了

```

参考文章：[Android 使 adb 作为 host 运行在 arm64 平台](https://blog.csdn.net/u010164190/article/details/88975915#commentBox)

源码和编译后的 adb 放在 github 上：[https://github.com/darren109/android-adb-tools](https://github.com/darren109/android-adb-tools)