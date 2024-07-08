# minimal_linux

>Thanks to **Nir Lichtman** for providing instructions of this minimal linux in youtube video [Making Simple Linux Distro from Scratch
](https://www.youtube.com/watch?v=QlzoegSuIzg)



### Open new powershell terminal  
![](https://placehold.co/600x10/8FBC8B/FFF?text=------------------------------------------------------------------)

Open new powershell terminal and run below command.
```bash
# Make sure you have Docker installed on your system
docker run --privileged -itd --name minilinux -v .:/mnt/hostfs ubuntu

# I tested with Ubuntu image version 35a88802559dd2077e584394471ddaa1a2c5bfd16893b829ea57619301eb3908

# If container is not running than run below command first
docker start minilinux

docker attach minilinux


```

After previous command execution you'll land into docker instance. Run below commands with in instance.

### Update and install required packages on docker instance
![](https://placehold.co/600x10/8FBC8B/FFF?text=------------------------------------------------------------------)

```bash
# Update apt repository index
apt update

# Install required packages (Few more we'll install later)
apt install bzip2 git vim make gcc libncurses-dev flex bison bc cpio libelf-dev libssl-dev syslinux dosfstools

mkdir /boot-files
```

### Build Linux Kernel
![](https://placehold.co/600x10/8FBC8B/FFF?text=------------------------------------------------------------------)

```bash

# Clone linux kernel code 
cd /
git clone --depth 1 https://github.com/torvalds/linux.git

# I tested with below commit
# git fetch --depth 1 origin 55027e68993
# git checkout FETCH_HEAD
# OR
# git checkout 55027e68993

# Run menuconfig to generate config files (keep default)
cd linux
make menuconfig
```
>> ![Qemu driver!](/assets/images/linux_driver_qemu.png "Qemu driver") 

```bash

# Make kernel
make -j16

cp arch/x86/boot/bzImage /boot-files/
```

### Build Busybox
![](https://placehold.co/600x10/8FBC8B/FFF?text=------------------------------------------------------------------)

```bash
# Clone busybox code 
cd /
git clone --depth 1 https://git.busybox.net/busybox/

# I tested with below commit
# git fetch --depth 1 origin a6ce017a8a2db09
# git checkout FETCH_HEAD
# OR
# git checkout a6ce017a8a2db09


# Run menuconfig to generate config files (keep default)
cd busybox
make menuconfig
```

> **Resolution for busybox install error:**  
>> Run menuconfig for busybox again and deselect tc component from networing utilities option.  
>> [Got Idea from this post](https://www.reddit.com/r/linuxquestions/comments/1cizfpo/id_like_some_help_with_this_youtube_guide/)  
>> ![Networking Utilities!](/assets/images/busybox_networking.png "Networking Utilities")  
>> ![tc Component!](/assets/images/busybox_networking_tc.png "tc Component")  
>> ![Static Component!](/assets/images/busybox_static.png "Static Component")  
>> once done build busybox again and install

```bash
# Make busybox
make -j16

# Copy busybox files to initramfs folder (Refer below if you face errors related to CONFIG_TC)
mkdir /boot-files/initramfs
make CONFIG_PREFIX=/boot-files/initramfs install

```

### Create initramfs
![](https://placehold.co/600x10/8FBC8B/FFF?text=------------------------------------------------------------------)

```bash
# Create init file
cd /boot-files/initramfs
cat >> /boot-files/initramfs/init<< EOF
#!/bin/sh

/bin/sh
EOF
chmod +x init

# Remove linuxrc folder (This is not required)
rm linuxrc
```

### Create boot disk
![](https://placehold.co/600x10/8FBC8B/FFF?text=------------------------------------------------------------------)

```bash
# Create cpio file
cd /boot-files/initramfs
find . | cpio -o -H newc > ../init.cpio



cd /boot-files/
#apt install syslinux
#apt install dosfstools

# Create empty boot disk file and format it with fat filesystem
dd if=/dev/zero of=boot bs=1M count=50
mkfs -t fat boot

# Make boot file bootable
syslinux boot

# Mount boot disk in temporary location and copy kernel and CPIO file
mkdir tmpmount
mount boot tmpmount
cp bzImage init.cpio tmpmount
umount tmpmount
```

### Open another separate powershell terminal
![](https://placehold.co/600x10/8FBC8B/FFF?text=------------------------------------------------------------------)

```shell
# Copy boot file from docker instance to local 
cp boot /mnt/hostfs/
# OR
docker cp e6b3f77c4c00:/boot-files/boot .

# Make sure you have Qemu installed on your system
# Run Qemu with boot disk file
qemu-system-x86_64 boot
```

### Qemu window
![](https://placehold.co/600x10/8FBC8B/FFF?text=------------------------------------------------------------------)

```shell
# Enter below line in Qemu window when asked
/bzImage -initrd=/init.cpio

# Once booted 
echo "Tested successfully"
```
