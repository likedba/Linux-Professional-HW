# HW1 Обновление ядра Linux
```bash
# Используя среду виртуализации VMware Workstation развернута ВМ с Ubuntu
aleksei@vmubuntu-1:~$ uname -r
6.8.0-62-generic
aleksei@vmubuntu-1:~$ uname -p
x86_64
# На момент выполнения дз на https://kernel.ubuntu.com/mainline последняя версия - v6.16-rc3
mkdir kernel && cd kernel
wget https://kernel.ubuntu.com/mainline/v6.16-rc3/amd64/linux-headers-6.16.0-061600rc3-generic_6.16.0-061600rc3.202506222136_amd64.deb
wget https://kernel.ubuntu.com/mainline/v6.16-rc3/amd64/linux-headers-6.16.0-061600rc3_6.16.0-061600rc3.202506222136_all.deb
wget https://kernel.ubuntu.com/mainline/v6.16-rc3/amd64/linux-image-unsigned-6.16.0-061600rc3-generic_6.16.0-061600rc3.202506222136_amd64.deb
wget https://kernel.ubuntu.com/mainline/v6.16-rc3/amd64/linux-modules-6.16.0-061600rc3-generic_6.16.0-061600rc3.202506222136_amd64.deb
sudo dpkg -i *.deb
aleksei@vmubuntu-1:~/kernel$ ls -al /boot
total 211208
drwxr-xr-x  4 root root     4096 Jun 27 16:11 .
drwxr-xr-x 23 root root     4096 Jun 27 05:31 ..
-rw-r--r--  1 root root   299511 Jun 22 21:37 config-6.16.0-061600rc3-generic
-rw-r--r--  1 root root   287598 May 19 10:55 config-6.8.0-62-generic
drwxr-xr-x  5 root root     4096 Jun 27 16:12 grub
lrwxrwxrwx  1 root root       35 Jun 27 16:11 initrd.img -> initrd.img-6.16.0-061600rc3-generic
-rw-r--r--  1 root root 92972327 Jun 27 16:11 initrd.img-6.16.0-061600rc3-generic
-rw-r--r--  1 root root 72393693 Jun 27 05:33 initrd.img-6.8.0-62-generic
lrwxrwxrwx  1 root root       27 Jun 27 05:31 initrd.img.old -> initrd.img-6.8.0-62-generic
drwx------  2 root root    16384 Jun 27 05:28 lost+found
-rw-------  1 root root 10126738 Jun 22 21:37 System.map-6.16.0-061600rc3-generic
-rw-------  1 root root  9109506 May 19 10:55 System.map-6.8.0-62-generic
lrwxrwxrwx  1 root root       32 Jun 27 16:11 vmlinuz -> vmlinuz-6.16.0-061600rc3-generic
-rw-------  1 root root 16028160 Jun 22 21:37 vmlinuz-6.16.0-061600rc3-generic
-rw-------  1 root root 15006088 May 19 15:52 vmlinuz-6.8.0-62-generic
lrwxrwxrwx  1 root root       24 Jun 27 05:31 vmlinuz.old -> vmlinuz-6.8.0-62-generic

aleksei@vmubuntu-1:~/kernel$ sudo update-grub
aleksei@vmubuntu-1:~/kernel$ sudo grub-set-default 0
aleksei@vmubuntu-1:~/kernel$ sudo reboot
aleksei@vmubuntu-1:~$ uname -r
6.16.0-061600rc3-generic
# ядро успешно обновлено
```
![image](https://github.com/user-attachments/assets/4751de50-f009-4b3f-b7a1-c209ebdc5971)
