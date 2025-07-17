# LVM-1
1. Смотрим какие диски присутствуют в системе, после их добавления в ui vmware

`lsblk`

<img width="509" height="181" alt="image" src="https://github.com/user-attachments/assets/cd27b565-5ffa-4c9b-85ca-1b11b510f0b1" />

`lvmdiskscan`
<img width="404" height="164" alt="image" src="https://github.com/user-attachments/assets/78c75b22-15ed-4800-a5a6-34f73999b921" />
3. Разметим диск для будущего использования LVM - создадим PV
`pvcreate /dev/sdb`
4. Затем можно создавать первый уровень абстракции - VG
`vgcreate otus /dev/sdb`

5. Создадим Logical Volume
`lvcreate -l+80%FREE -n test otus`
6. Посмотрим информацию о только что созданном Volume Group
`vgdisplay otus`
<img width="500" height="321" alt="image" src="https://github.com/user-attachments/assets/d75883b7-c074-4a49-b4e4-5759618b3da8" />
7. Посмотрим информацию о том, какие диски входит в VG:
`vgdisplay -v otus | grep 'PV Name'`
<img width="427" height="29" alt="image" src="https://github.com/user-attachments/assets/6036bc44-0b18-4235-b74c-1eecc502f2af" />
8. Получим детальную информацию о LV 
`lvdisplay /dev/otus/test`
<img width="518" height="263" alt="image" src="https://github.com/user-attachments/assets/c9ab692c-34c7-43ad-9b77-20acf2cc73e8" />
<img width="741" height="121" alt="image" src="https://github.com/user-attachments/assets/ef5ace28-0974-42ad-b0ca-ccce7e531560" />

9. Создадим еще один LV из свободного места, но в мегабайтах:
`lvcreate -L100M -n small otus`
<img width="316" height="52" alt="image" src="https://github.com/user-attachments/assets/475eb6ad-66b3-4cca-bba6-75ae2b3c1d4d" />
10. Создадим на LV файловую систему и смонтируем его
`mkfs.ext4 /dev/otus/test`
<img width="564" height="153" alt="image" src="https://github.com/user-attachments/assets/7d763e64-e090-48b4-87d5-a065db0cf1a1" />

# Расширение LVM

11. Создадим PV
`pvcreate /dev/sdc`
12. Расширим VG, добавив в него диск
```bash
vgextend otus /dev/sdc
vgdisplay -v otus | grep 'PV Name'
```
<img width="424" height="46" alt="image" src="https://github.com/user-attachments/assets/aeeefabf-5014-40ae-b813-b57e327b725b" />
<img width="367" height="58" alt="image" src="https://github.com/user-attachments/assets/99434c01-0157-4155-bede-931dd36240d2" />
```bash
mkdir /data
mount /dev/otus/test /data/
mount | grep /data
```
13. Сымитируем занятое место

`dd if=/dev/zero of=/data/test.log bs=1M count=8000 status=progress`
<img width="714" height="167" alt="image" src="https://github.com/user-attachments/assets/76aeda2a-9a01-4c13-bac1-cf923735de9f" />
<img width="490" height="38" alt="image" src="https://github.com/user-attachments/assets/d10c5d6a-20e4-46d2-823e-bbf1f080d146" />
14. Увеличиваем LV за счет появившегося свободного места
`lvextend -l+80%FREE /dev/otus/test`
<img width="801" height="33" alt="image" src="https://github.com/user-attachments/assets/68b86544-6579-43af-b31d-7f7b6ca4127d" />
`lvs /dev/otus/test`
<img width="654" height="35" alt="image" src="https://github.com/user-attachments/assets/32c86947-f4cb-4cd8-924b-19a67572df18" />
15. Ресайзим ФС
`resize2fs /dev/otus/test`
`df -Th /data`
<img width="496" height="36" alt="image" src="https://github.com/user-attachments/assets/1f963e4c-db7e-4a99-ad50-d20da9ef7745" />
16. Уменьшаем существующий LV
```bash
umount /data/
e2fsck -fy /dev/otus/test
```
<img width="642" height="111" alt="image" src="https://github.com/user-attachments/assets/6de23d2e-c287-4f9e-9fc3-0b3e68675bf7" />
```bash
resize2fs /dev/otus/test 10G
lvreduce /dev/otus/test -L 10G
mount /dev/otus/test /data/
df -Th /data/
```
<img width="492" height="37" alt="image" src="https://github.com/user-attachments/assets/f41bb721-1c6b-4886-ac82-15d9003c4465" />
`lvs /dev/otus/test`

# Работа со снэпшотами

17. Создаем снэпшот
```bash
lvcreate -L 500M -s -n test-snap /dev/otus/test
vgs -o +lv_size,lv_name | grep test
```
<img width="496" height="34" alt="image" src="https://github.com/user-attachments/assets/b4eb15c1-211c-4157-8227-6de088b92614" />
`lsblk`
<img width="507" height="311" alt="image" src="https://github.com/user-attachments/assets/d360379b-7b51-46cd-b9de-c5b6366e4d51" />
18. Смонтируем снэпшот
```bash
mkdir /data-snap
mount /dev/otus/test-snap /data-snap/
ll /data-snap/
```
<img width="477" height="77" alt="image" src="https://github.com/user-attachments/assets/50cc39c2-3c49-4ae1-82ce-18963799af1e" />
`umount /data-snap`

12. Откат на снэпшот
`rm /data/test.log`
<img width="449" height="48" alt="image" src="https://github.com/user-attachments/assets/68c7787f-90df-40f1-9e76-8f3ecaace4ec" />
```bash
umount /data
lvconvert --merge /dev/otus/test-snap
```
<img width="328" height="30" alt="image" src="https://github.com/user-attachments/assets/1c95f44f-9a26-4116-8058-951a33cce841" />
`mount /dev/otus/test /data`
<img width="480" height="92" alt="image" src="https://github.com/user-attachments/assets/1e893aa5-a85f-4c44-a023-6b2f1f3910db" />


## HW
1. Очистим стенд от изменений из предыдущих действий
```bash
umount /data
lvremove /dev/ubuntu-vg/otus-test
lvremove /dev/ubuntu-vg/otus-small
vgremove otus
pvremove /dev/sdb
pvremove /dev/sdc
```
<img width="507" height="182" alt="image" src="https://github.com/user-attachments/assets/f872623c-4af5-49b7-af01-224e54e26adc" />

2. создадим PV VG LV FS
```bash
pvcreate /dev/sdb
vgcreate vg_root /dev/sdb
lvcreate -n lv_root -l +100%FREE /dev/vg_root
mkfs.ext4 /dev/vg_root/lv_root
mount /dev/vg_root/lv_root /mnt
```
3.  копируем все данные с / раздела в /mnt
```bash
rsync -avxHAX --progress / /mnt/
ls /mnt
```
<img width="914" height="61" alt="image" src="https://github.com/user-attachments/assets/655edd1e-1eee-49fd-8897-15c9b6105a3d" />

4. редактируем grub для запуска из нового /
```bash
for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
chroot /mnt/
grub-mkconfig -o /boot/grub/grub.cfg
```
<img width="627" height="200" alt="image" src="https://github.com/user-attachments/assets/58ececca-acbf-4735-b359-58bc3a59c140" />
```bashupdate-initramfs -u
lsblk
```
<img width="509" height="197" alt="image" src="https://github.com/user-attachments/assets/5aca6ccb-42f8-419d-909a-4f1d518bd8e0" />

5. Изменим размер старой VG и вернем на него рут. Для этого удаляем старый LV размером в 40G и создаём новый на 8G
```bash
lvremove /dev/ubuntu-vg/ubuntu-lv
lvcreate -n ubuntu-vg/ubuntu-lv -L 8G /dev/ubuntu-vg
mkfs.ext4 /dev/ubuntu-vg/ubuntu-lv
mount /dev/ubuntu-vg/ubuntu-lv /mnt
rsync -avxHAX --progress / /mnt/
for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done
chroot /mnt/
grub-mkconfig -o /boot/grub/grub.cfg
update-initramfs -u
```
<img width="614" height="211" alt="image" src="https://github.com/user-attachments/assets/a6b40cb0-52d6-4ab3-b9ef-4303966d8baa" />

6. Выделим том под /var в зеркало
```bash
pvcreate /dev/sdc /dev/sdd
vgcreate vg_var /dev/sdc /dev/sdd
lvcreate -L 950M -m1 -n lv_var vg_var
mkfs.ext4 /dev/vg_var/lv_var
```
<img width="525" height="174" alt="image" src="https://github.com/user-attachments/assets/eda6f02a-ad83-4bc4-8de0-fa160e211213" />
```bash
mount /dev/vg_var/lv_var /mnt
cp -aR /var/* /mnt/
mkdir /tmp/oldvar && mv /var/* /tmp/oldvar
umount /mnt
mount /dev/vg_var/lv_var /var
echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab
lsblk
```
<img width="505" height="318" alt="image" src="https://github.com/user-attachments/assets/18f10784-a478-4243-a6c5-f3eda701d3fd" />
`reboot`
7. Удалим временную Volume Group, lv, PV 
```bash
lvremove /dev/vg_root/lv_root
vgremove /dev/vg_root
pvremove /dev/sdb
```
<img width="508" height="302" alt="image" src="https://github.com/user-attachments/assets/a0ed6928-232d-4278-917b-52c7958ed8df" />

8. Выделим том под /home аналогично /var
```bash
lvcreate -n LogVol_Home -L 2G /dev/ubuntu-vg
mkfs.ext4 /dev/ubuntu-vg/LogVol_Home
mount /dev/ubuntu-vg/LogVol_Home /mnt/
cp -aR /home/* /mnt/
rm -rf /home/*
umount /mnt
mount /dev/ubuntu-vg/LogVol_Home /home/
echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab
lsblk
```
<img width="506" height="315" alt="image" src="https://github.com/user-attachments/assets/80c21ee6-a6d4-40b3-bca4-56ffd93adb3a" />

# Работа со снапшотами

1. Генерируем файлы в /home
`touch /home/file{1..20}`
2. Создаем снэпшот
`lvcreate -L 100MB -s -n home_snap /dev/ubuntu-vg/LogVol_Home`
3. Дропаем несколько файлов
```bash
rm -f /home/file{11..20}
ls -la /home
```
<img width="473" height="230" alt="image" src="https://github.com/user-attachments/assets/25681855-be94-4ac8-bd7c-69bd264abef5" />
4.  Восстанавливаем
```bash
umount /home
lvconvert --merge /dev/ubuntu-vg/home_snap
```
<img width="389" height="35" alt="image" src="https://github.com/user-attachments/assets/4b533c53-95d4-4203-9b86-f59b8c8bc086" />
`mount /dev/mapper/ubuntu--vg-LogVol_Home /home`
5. проверяем
`ls -al /home`
<img width="479" height="381" alt="image" src="https://github.com/user-attachments/assets/a38175a1-8075-4e75-8a0c-a3433ca1fd1b" />

Восстановление успешно
