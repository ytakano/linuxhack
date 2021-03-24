# Testing Linux Kernel in Rust for M1 Mac

## Build and Run Docker Container

build & run
```
$ cd docker
$ docker-compose build
$ docker-compose up -d
$ docker exec -it docker_linuxhack_1 /bin/zsh
```

## Compile Kernel in the Container

### Linux Kernel

```
# git clone --depth 1 git://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git
# cd linux-next
# make menuconfig
```

config

```
64-bit kernel ---> yes
General setup ---> Rust support ---> yes
General setup ---> Initial RAM filesystem and RAM disk (initramfs/initrd) support ---> yes
General setup ---> Configure standard kernel features ---> Enable support for printk ---> yes
Executable file formats / Emulations ---> Kernel support for ELF binaries ---> yes
Executable file formats / Emulations ---> Kernel support for scripts starting with #! ---> yes
Device Drivers ---> Generic Driver Options ---> Maintain a devtmpfs filesystem to mount at /dev ---> yes
Device Drivers ---> Generic Driver Options ---> Automount devtmpfs at /dev, after the kernel mounted the rootfs ---> yes
Device Drivers ---> Character devices ---> Rust Example ---> yes
Device Drivers ---> Character devices ---> Enable TTY ---> yes
Device Drivers ---> Character devices ---> Serial drivers ---> 8250/16550 and compatible serial support ---> yes
Device Drivers ---> Character devices ---> Serial drivers ---> Console on 8250/16550 and compatible serial port ---> yes
File systems ---> Pseudo filesystems ---> /proc file system support ---> yes
File systems ---> Pseudo filesystems ---> sysfs file system support ---> yes
```

compile

```
# make -j 8
```

### Busybox

```
# cd /hostdir
# git clone git://busybox.net/busybox.git
# cd busybox
# make defconfig
# make menuconfig
```

config

```
Settings ---> Build Options ---> Build static binary (no shared libs) ---> yes
```

compile

```
# make -j 8
# make install
# cd _install
# mkdir dev proc sys
```

create initial RAM disk

```
# find . | cpio --quiet -H newc -o | gzip > ../../linux-next/initrd.img
```

## Run Linux Kernel on Qemu for M1 Mac

Download ACVM https://github.com/KhaosT/ACVM/releases/

Linux on Qemu

```
$ cd linux-next
$ ~/Downloads/ACVM.app/Contents/Resources/qemu-system-aarch64 -accel hvf -cpu cortex-a57 -m 128M -M virt,highmem=off -kernel arch/arm64/boot/Image -append "console=ttyAMA0 rdinit=/bin/sh root=/dev/ram0" -initrd initrd.img -nographic
```

mount /dev and cat a device file written in Rust

```
# mount -t devtmpfs devtmpfs /dev
# cat /dev/rust_miscdev
[   57.277586] rust file was opened!
```

