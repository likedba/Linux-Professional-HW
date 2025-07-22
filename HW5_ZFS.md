# ZFS
## Определение алгоритма с наилучшим сжатием

1. Смотрим список всех дисков, которые есть в виртуальной машине
<img width="507" height="246" alt="image" src="https://github.com/user-attachments/assets/63f123dc-18ec-4fd9-bfbd-0423c1c1c4d9" />

2. Установим ZFS

`sudo apt install zfsutils-linux`

4. Создадим несколько пулов из raid1
```bash
zpool create otus1 mirror /dev/sdb /dev/sdc
zpool create otus2 mirror /dev/sdd /dev/sde
zpool create otus3 mirror /dev/sdf /dev/sdg
zpool create otus4 mirror /dev/sdh /dev/sdi
zpool list
```

<img width="686" height="80" alt="image" src="https://github.com/user-attachments/assets/a5188843-e8fb-4bd9-8864-fae1cd9654f7" />

4. Добавим разные алгоритмы сжатия в каждую файловую систему
```bash
zfs set compression=lzjb otus1
zfs set compression=lz4 otus2
zfs set compression=gzip-9 otus3
zfs set compression=zstd otus4
zfs get all | grep compression
```

<img width="481" height="62" alt="image" src="https://github.com/user-attachments/assets/2bcc89bb-f84b-48cf-8cbd-7bbb2eb13faa" />

6. Скачаем один и тот же файл во все пулы

`for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done`

`ls -l /otus*`

<img width="533" height="229" alt="image" src="https://github.com/user-attachments/assets/bba9d6bc-f309-4a9a-a43b-749543f35c24" />

`zfs list`

<img width="312" height="78" alt="image" src="https://github.com/user-attachments/assets/121f35ce-364d-4b4d-b16c-c263c7f36fa1" />

Таким образом получается, что gzip-9 самый эффективный, но zstd почти на одном уровне.

## Определение настроек пула

1. Скачиваем архив в домашний каталог

```bash
wget -O archive.tar.gz --no-check-certificate 
 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
tar -xzvf archive.tar.gz
```

3. Проверим, возможно ли импортировать данный каталог в пул

`zpool import -d zpoolexport/`

<img width="645" height="212" alt="image" src="https://github.com/user-attachments/assets/207fcb11-0801-4b48-8caa-21542c650434" />

5. Импорт пула в ОС
```bash
zpool import -d zpoolexport/ otus
zpool status
zpool get all otus
zfs get available otus
```

<img width="250" height="31" alt="image" src="https://github.com/user-attachments/assets/8657780d-22df-452c-a90e-e05a57b3d8b4" />

`zfs get readonly otus`

<img width="261" height="31" alt="image" src="https://github.com/user-attachments/assets/480b976d-7b98-4bcd-b503-8f0a676b593f" />

`zfs get recordsize otus`

<img width="299" height="33" alt="image" src="https://github.com/user-attachments/assets/4e80ce32-c6a9-4c25-b766-3566c05c932e" />

`zfs get compression otus`

<img width="338" height="33" alt="image" src="https://github.com/user-attachments/assets/7c0b921a-e50b-4551-b1d3-cf17989d6728" />

`zfs get checksum otus`

<img width="268" height="33" alt="image" src="https://github.com/user-attachments/assets/d530d00d-9b09-458b-b511-c5397fdff3d4" />

## Работа со снапшотом, поиск сообщения от преподавателя

1. Скачать файл

`wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download`

3. Восстановим файловую систему из снапшота

`zfs receive otus/test@today < otus_task2.file`

5. Поиск сообщения

`find /otus/test -name "secret_message"`

`cat /otus/test/task1/file_mess/secret_message`

<img width="521" height="37" alt="image" src="https://github.com/user-attachments/assets/a499ec3f-48df-4899-92ad-57a00256c90f" />

