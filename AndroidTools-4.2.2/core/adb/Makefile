# Makefile for adb;

TOOLCHAIN_DIR=/Users/darren/Library/Android/sdk/ndk/26.1.10909125/toolchains/llvm/prebuilt/darwin-x86_64/bin
CC=$(TOOLCHAIN_DIR)/aarch64-linux-android33-clang
VPATH+= ../adb
SRCS+= adb.c
SRCS+= adb_client.c
SRCS+= adb_auth_host.c
SRCS+= commandline.c
SRCS+= console.c
SRCS+= file_sync_client.c
SRCS+= fdevent.c
SRCS+= get_my_path_linux.c
SRCS+= services.c
SRCS+= sockets.c
SRCS+= transport.c
SRCS+= transport_local.c
SRCS+= transport_usb.c
SRCS+= usb_linux.c
SRCS+= usb_vendors.c
SRCS+= utils.c

VPATH+= ../libcutils
SRCS+= abort_socket.c
SRCS+= socket_inaddr_any_server.c
SRCS+= socket_local_client.c
SRCS+= socket_local_server.c
SRCS+= socket_loopback_client.c
SRCS+= socket_loopback_server.c
SRCS+= socket_network_client.c
SRCS+= list.c
SRCS+= load_file.c

VPATH+= ../libzipfile
SRCS+= centraldir.c
SRCS+= zipfile.c


CPPFLAGS+= -DADB_HOST=1
CPPFLAGS+= -DHAVE_FORKEXEC=1
CPPFLAGS+= -DHAVE_SYMLINKS
CPPFLAGS+= -DHAVE_TERMIO_H
CPPFLAGS+= -I../adb
CPPFLAGS+= -I../include

LIBS+= -L. -lc -lcrypto -lz

OBJS= $(SRCS:.c=.o)

all: adb

adb: $(OBJS)
	$(CC) -o $@ $(LDFLAGS) $(OBJS) $(LIBS)

clean:
	rm -rf $(OBJS) adb
