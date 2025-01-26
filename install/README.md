# Tutorial Instalasi Arch Linux

Kunjungi tutorial resmi [di sini](https://wiki.archlinux.org/title/Installation_guide) 

Sebelum mengikuti tutorial ini, pastikan anda telah melakukan hal berikut
- Download the official [image](https://archlinux.org/download/) file.
- Membuat bootable 
- Boot ke bootable

Jika anda melihat ini di komputer anda :
```
[ root@archiso `]$ 
```

Maka anda siap mengikuti tutorial ini.

## Bagian 1: Pre-Installation 

### Tahap 1: Konfigurasi virtual console
* Gunakan `loadkeys` untuk mengatur [keymap](https://wiki.archlinux.org/title/Linux_console/Keyboard_configuration), contoh untuk *United State English*
```
# loadkeys us
```
Untuk melihat keymap yang tersedia, gunakan `localectl list-keymaps`

### Tahap 2: Menghubungkan ke internet
Pilihan untuk menghubungkan ke internet :
- Ethernet : plug in cable
- Wi-Fi : use `iwctl`
- Mobile broadband modem : use `nmcli`

Verifikasi internet `ping 8.8.8.8`, jika output nya ada yang seperti ini : `Name or service not known` maka anda belum terhubung ke internet.

Cek waktu sistem `timedatectl`

### Tahap 3: Mempartisi disk dan mouting
Identifikasi [block device](https://wiki.archlinux.org/title/Device_file#Block_devices) menggunakan `lsblk`
Gunakan `cfdisk` untuk mengatur partisi.
Buatlah partisi sesuai kebutuhan. Misal
1. Root 
2. Home 
3. Boot 
4. Swap 

Partisi *root* :
```
$ mkfs.ext4 /dev/root_partition
$ mount /dev/root_partition /mnt
```
Partisi *home* : 
```
$ mkfs.ext4 /dev/home_partition
$ mkdir -p /mnt/home
$ mount /dev/home_partition /mnt/home
```
Untuk partisi *swap*
```
$ mkswap /dev/swap_partition
$ swapon /de/swap_partition
```
#### Boot Partition
Bagi pengguna dengan sistem *EFI* format dengan :
```
$ mkdir /mnt/boot/efi
$ mkfs.fat -F 32 /dev/efi_partition
$ mount /dev/efi_partition /mnt/boot/efi
```
Bagi pengguna dengan sistem *MBR* format dengan 
```
$ mkfs.ext4 /dev/boot_partition
$ mkdir -p /mnt/Boot
$ mount /dev/boot_partition /mnt/boot
```
verifikasi boot flag, untuk mengatur boot flag gunakan :
```
$ parted /dev/sda
$ set 1 boot on 
```

## Bagian kedua: Installation

Atur [mirro](https://wiki.archlinux.org/title/Mirrors) sesuai kebutuhan : `/etc/pacman.d/mirrorlist`

### Tahap 1: Install package 
Package wajib di install : `base`, `linux`, `linux-firmware`, opsional: `linux-lts`
Package lain yang juga penting
- hardare bug and security fixes : `amd-ucode`, `intel-ucode`
- [Boot loader](https://wiki.archlinux.org/title/Arch_boot_process) : `grub` (untuk *UEFI* sistem), `syslinux` (untuk *BIOS* sistem)
- Alat untuk membuild packages linux : `base-devel`
- Network manager : `networkmanager`
- text editor untuk console : `vim` or `nano`
Gunakan `pacstrap` untuk mengunduh packages, contoh :
```
$ pacstrap -K /mnt base
```
## Bagian ketiga: Mengkonfigurasi sistem
* membuat [fstab](https://wiki.archlinux.org/title/Fstab) file : 
```
& genfstab -U /mnt >> /mnt/etc/fstab
```
cek lagi dengan : `# cat /mnt/etc/fstab`
opsional : edit *UUID* pada fstab file sesuai partisi
### Konfigurasi root
Ganti ke root :
```
$ arch-chroot /mnt
```
Hal yang akan dilakukan
1. Mengatur time zone 
2. Mengatur pengaturan bahasa (locale)
3. Menambahkan pengguna / user
4. Mengatur bootloader
```
$ arch-chroot /mnt
```
* Mengatur [time zone](https://wiki.archlinux.org/title/System_time#Time_zone)
```
$ ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
& hwclock --systohc
```
* Mengatur locale, edit `/etc/locale.gen` dan hapus komentar untuk `en_US.UTF-8 UTF-8`, kemudian jalankan :
```
$ locale-gen
```
Buat `/etc/locale.conf`, dan tulis :
```
LANG=en_US.UTF-8
```
opsional : Mengatur keyboard layout, edit `/etc/vconsole.conf` dan tulis
```
KEYMAP=us
```
cek lagi dengan `# mkinitcpio -p linux-lts`

### Menambahkan [pengguna](https://wiki.archlinux.org/title/Users_and_groups#User_management)
Membuat nama host, edit `/etc/hostname` kemudian tulis nama host nya, setelah itu atur password dengan `passwd`
Menambahkan user, contoh :
```
useradd -m -G wheel -s /bin/bash dapz
```
edit `VISUDO`

#### Mengatur bootloader
Untuk *UEFI* sistem, gunakan `grub` 
```
# grub-install 
# grub-mkconfig -o /boot/grub/grub.cfg
```
Untuk *BIOS*, gunakan `syslinux`
```
# pacman -S syslinux
# syslinux-install_update -i -a -m
```
Jika sudah aman, keluar dari root dengan `$ exit`
## Bagian akhir: Reboot
Sebelum rebot, aktifkan service penting.

### Unmount
```
$ umount -R /mnt
```
or
```
$ umount all
```

reboot dengan `reboot`

[Tutorial lainnya]()

Reference
[LEGACY SETUP](https://gist.github.com/xbns/cb8d0f9734a99c19c2503d8439f79e71#file-arch-linux-installation-on-mbr-system-md)
[UEFI SETUP](https://gist.github.com/xbns/3516ee4582f74fc3c41fee3541369fd5#file-arch-linux-installation-on-uefi-gpt-system-md)