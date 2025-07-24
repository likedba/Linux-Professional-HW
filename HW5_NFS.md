# NFS

Созданы 2 ВМ:
vmubuntu-1 192.168.121.141
vmubuntu-2 192.168.121.142


1. Настраиваем сервер NFS

```bash
apt install nfs-kernel-server
ss -tnplu
```

<img width="568" height="215" alt="image" src="https://github.com/user-attachments/assets/ff271df7-4c53-4853-9681-1e61e78085e5" />

2. создаем каталог для экспорта

```bash
mkdir -p /srv/share/upload
chown -R nobody:nogroup /srv/share
chmod 0777 /srv/share/upload
```

4. Редактируем exports

```bash
cat << EOF > /etc/exports 
/srv/share 192.168.121.142/32(rw,sync,root_squash)
EOF
exportfs -r
exportfs -s
```

<img width="874" height="31" alt="image" src="https://github.com/user-attachments/assets/8601203c-a7ee-4b50-88b6-7ef037782f6d" />

6. Настраиваем клиент

`apt install nfs-common`

8. Редактируем /etc/fstab

```bash
echo "192.168.121.141:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0" >> /etc/fstab
systemctl daemon-reload
systemctl restart remote-fs.target
ls /mnt
mount | grep mnt
```

<img width="989" height="75" alt="image" src="https://github.com/user-attachments/assets/19d16aed-7f77-4134-81b4-35b0aea570ca" />

10. Проверка работоспособности

на сервере

```bash
cd /srv/share/upload/
touch check_file
```

смотрим на клиенте

`ls -la /mnt/upload`

<img width="460" height="81" alt="image" src="https://github.com/user-attachments/assets/2753152b-136d-4744-a15e-683aedc4bb3d" />

`touch /mnt/upload/client_file`

<img width="472" height="93" alt="image" src="https://github.com/user-attachments/assets/eefec975-a019-4181-bad4-e7475539cf32" />

