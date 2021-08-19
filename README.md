# custom-rpi-initramfs
scripts to handle RPi initramfs generation - originally made by @Ingo at https://raspberrypi.stackexchange.com/a/92558/16278 

# BTRFS on RPi
https://raspberrypi.stackexchange.com/questions/8265/btrfs-root-filesystem-on-raspbian

- Install the required packages - the kernel module, and the tools to update initramfs with it: `sudo apt install btrfs-tools initramfs-tools`
- Tell initramfs to load the btrfs module (should be happening automagically, for some reason, didn't work on my RPi1) - append a line with "btrfs" to the list of required modules: `echo 'btrfs' | sudo tee -a /etc/initramfs-tools/modules`
- Create an initramfs hook (for building the image) and script (for booting) for btrfs - defaults are provided, but in my testing, they were not used automagically, had to copy them into /etc. `sudo mkdir -p /etc/initramfs-tools/hooks ; sudo mkdir -p /etc/initramfs-tools/scripts/local-premount ; sudo cp /usr/share/initramfs-tools/hooks/btrfs /etc/initramfs-tools/hooks ; sudo cp /usr/share/initramfs-tools/scripts/local-premount/btrfs /etc/initramfs-tools/scripts/local-premount; sudo chmod +x /etc/initramfs-tools/hooks/btrfs /etc/initramfs-tools/scripts/local-premount/btrfs`

*upgrading* the kernel would break the system. To fix this, I used his kernel-install hook [scripts](https://raspberrypi.stackexchange.com/a/92558/16278):
  - Edit /etc/default/raspberrypi-kernel and uncomment `INITRD=Yes`
  - delete `/etc/kernel/postinst.d/initramfs-tools`
  - add [rpi-initramfs-tools](https://github.com/Piskvor/custom-rpi-initramfs/blob/master/etc/kernel/postinst.d/rpi-initramfs-tools) to /etc/kernel/postinst.d/ and `chmod +x` it
  - optionally, download update-rpi-initramfs for simpler manual updates of the initramfs. 

apt update && apt upgrade (when new kernel install will get initrd and also update config.txt)

edit cmdline.txt
console=serial0,115200 console=tty1 root=UUID="03371803-2005-4f70-ad7b-686341f32924" rootflags=subvol=/subvolumes/@root rootfstype=btrfs apparmor=1 security=apparmor elevator=deadline fsck.repair=yes rootwait

use UUID not PARTUUID as this won't work with a initrd
