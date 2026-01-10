Building and Installing a Custom Linux Kernel on Void Linux (Safe Method)

This guide explains how to build, install, and boot a custom kernel on Void Linux without breaking EXT4, initramfs, or Docker networking.

📌 Important Void Linux Rules

Void appends a local version suffix (_1) to kernels.

All components must use the exact same version string:

Kernel image

/lib/modules

initramfs

GRUB entry

Never remove _1.

Example kernel version:

6.18.4_1

1️⃣ Install Required Build Dependencies
sudo xbps-install -S \
  base-devel \
  bc \
  bison \
  flex \
  elfutils \
  libelf-devel \
  ncurses-devel \
  openssl-devel \
  dracut \
  grub-x86_64-efi

2️⃣ Download and Extract the Kernel
mkdir -p ~/kernel-build
cd ~/kernel-build

wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.18.4.tar.xz
tar xf linux-6.18.4.tar.xz
cd linux-6.18.4

3️⃣ Configure the Kernel

Start from your current working kernel config:

zcat /proc/config.gz > .config
make olddefconfig


Optionally review settings:

make menuconfig

Required Options (verify)
File systems  --->
  <*> EXT4 filesystem support

Networking support  --->
  Networking options  --->
    <*> Network packet filtering framework (Netfilter)

    IP: Netfilter Configuration  --->
      <*> IPv4 NAT
      <*> MASQUERADE target support

Device Drivers  --->
  Network device support  --->
    <*> Bridge support

4️⃣ Build the Kernel
make -j$(nproc)

5️⃣ Install Kernel Modules
sudo make modules_install


Verify:

ls /lib/modules | grep 6.18.4


Expected:

6.18.4_1

6️⃣ Install Kernel Files to /boot
sudo cp arch/x86/boot/bzImage /boot/vmlinuz-6.18.4_1
sudo cp System.map /boot/System.map-6.18.4_1
sudo cp .config /boot/config-6.18.4_1

7️⃣ Generate initramfs (IMPORTANT)

Do not manually add ext4 or other modules.

sudo dracut --kver 6.18.4_1 --force /boot/initramfs-6.18.4_1.img


Verify:

ls -lh /boot/initramfs-6.18.4_1.img

8️⃣ Update GRUB
sudo grub-mkconfig -o /boot/grub/grub.cfg


Verify entry:

grep -A5 "6.18.4" /boot/grub/grub.cfg


Expected:

linux /boot/vmlinuz-6.18.4_1
initrd /boot/initramfs-6.18.4_1.img

9️⃣ Reboot
sudo reboot
