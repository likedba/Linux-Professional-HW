# Raid

1. посмотрим, какие блочные устройства у нас есть (добавлено 8 дисков через ui vmware)

`lsblk`

![image](https://github.com/user-attachments/assets/913e93e8-b328-4cbf-9378-67e679773ceb)

2. Создаем raid10

```bash
sudo mdadm --create /dev/md0 --level=10 --raid-devices=8 /dev/sd{b..i}
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

3. Проверим raid

`cat /proc/mdstat`

![image](https://github.com/user-attachments/assets/6f74c8a3-cd99-4caa-88b5-8b23da3ef84c)

`sudo mdadm -D /dev/md0`

![image](https://github.com/user-attachments/assets/ab22587a-2798-42a7-b44c-b961ec0f4549)

## Сломаем и починим RAID

4. Фэйлим диск sdf

```bash
sudo mdadm /dev/md0 --fail /dev/sdf
mdadm: set /dev/sdf faulty in /dev/md0
```

5. Смотрим проблему рэйда

```bash
cat /proc/mdstat
Personalities : [raid0] [raid1] [raid4] [raid5] [raid6] [raid10] [linear]
md0 : active raid10 sdi[7] sdh[6] sdg[5] sdf[4](F) sde[3] sdd[2] sdc[1] sdb[0]
      4186112 blocks super 1.2 512K chunks 2 near-copies [8/7] [UUUU_UUU]
```

`sudo mdadm -D /dev/md0`

![image](https://github.com/user-attachments/assets/28943bb3-f305-4048-91b3-f47d6a65ef84)

6. Дропнем сломанный диск из массива

```bash
mdadm /dev/md0 --remove /dev/sdf
sudo mdadm /dev/md0 --remove /dev/sdf
mdadm: hot removed /dev/sdf from /dev/md0
```

7. Добавим новый диск в массив

```bash
sudo mdadm /dev/md0 --add /dev/sdf
mdadm: added /dev/sdf
```

8. Смотрим состояние массива (восстановлен)

`cat /proc/mdstat`

![image](https://github.com/user-attachments/assets/404379ae-6b18-4473-93ab-ed9b7e71126a)

## Создать GPT таблицу, пять разделов и смонтировать их в системе

9. Создаем таблицу GPT

`sudo parted -s /dev/md0 mklabel gpt`

![image](https://github.com/user-attachments/assets/f503dd46-c071-48d1-b031-42e602cc5b69)

11. Создаем 5 партиций

```bash
sudo parted /dev/md0 mkpart primary ext4 0% 20%
sudo parted /dev/md0 mkpart primary ext4 20% 40%
sudo parted /dev/md0 mkpart primary ext4 40% 60%
sudo parted /dev/md0 mkpart primary ext4 60% 80%
sudo parted /dev/md0 mkpart primary ext4 80% 100%
```

![image](https://github.com/user-attachments/assets/744afe15-f5a3-4841-afa0-f86fdb3a71c2)

12. Создаем ФС на разделах

```bash
sudo mkfs.ext4 /dev/md0p1;
sudo mkfs.ext4 /dev/md0p2;
sudo mkfs.ext4 /dev/md0p3;
sudo mkfs.ext4 /dev/md0p4;
sudo mkfs.ext4 /dev/md0p5;
```

либо

`for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i;`

14. Создаем каталоги для монтирования

`sudo mkdir -p /raid/part{1,2,3,4,5}`

16. Монтируем ФС в созданные каталоги

```bash
sudo mount /dev/md0p1 /raid/part1
sudo mount /dev/md0p2 /raid/part2
sudo mount /dev/md0p3 /raid/part3
sudo mount /dev/md0p4 /raid/part4
sudo mount /dev/md0p5 /raid/part5
```

либо

`for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done`

![image](https://github.com/user-attachments/assets/f1683e2d-5f58-4d21-85da-a5480b69a3da)

## Настроим автомонтирование

14. Получим UUID RAID

```bash
sudo blkid | grep md0
/dev/md0p4: UUID="1b6f1e0a-7cd2-4258-93a7-a21a4bc93881" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="primary" PARTUUID="6bf7895d-1d6b-4dd8-9bf2-ed8b3f0e4ac7"
/dev/md0p2: UUID="4e546a4c-ead9-412a-bf54-f1e6b9098812" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="primary" PARTUUID="ebb40e89-6ca3-4cf7-8719-6e191e649beb"
/dev/md0p5: UUID="6f484527-0922-4b67-aa70-00b5dc338420" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="primary" PARTUUID="1affab16-e9f5-484a-9454-962f233271bb"
/dev/md0p3: UUID="2c711bae-3d31-452c-a3c6-4d77d5acd20d" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="primary" PARTUUID="56d7c8d9-e1d1-47dc-bebd-2365d12069f7"
/dev/md0p1: UUID="e1d96cfd-d12a-497d-9be2-a0ce542c85ff" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="primary" PARTUUID="746399d8-8d21-47ed-a125-3ae991e29f4b"
```

15. Добавим запись в /etc/fstab

`sudo nano /etc/fstab`

![image](https://github.com/user-attachments/assets/bd156cbd-7ff1-43d5-b8ab-08d9fdf4b98b)

17. Проверяем монтирование

`sudo mount -a`

19. Сохраняем конфигурацию RAID

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

18. Рестартуем машину

`sudo reboot`

20. Проверяем массив и монтирование

```bash
mount | grep raid
cat /proc/mdstat
```

![image](https://github.com/user-attachments/assets/44a5b30f-ec44-4d8f-a190-1b8a49ff0670)

