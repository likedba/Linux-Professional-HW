# Systemd  
1. создаём файл с конфигурацией для сервиса
vim /etc/default/watchlog

# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log

2. создаем /var/log/watchlog.log и пишем туда строки на своё усмотрение,
плюс ключевое слово ‘ALERT’
vim /var/log/watchlog.log

ALERT
ALERT
3. Создаем скрипт который ищет ключевое слово в логе
vim /opt/watchlog.sh

#!/bin/bash

WORD=$1
LOG=$2
DATE=`date`

if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi

4. Добавим права на запуск скрипта  
chmod +x /opt/watchlog.sh


5. Создадим юнит для сервиса
vim /etc/systemd/system/watchlog.service

[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG

6. Создадим юнит для таймера  
vim /etc/systemd/system/watchlog.timer

[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target

7. Запускаем таймер
systemctl start watchlog.timer
<img width="832" height="395" alt="image" src="https://github.com/user-attachments/assets/53e6ae07-cc91-4401-af4a-60ffc0d9cec4" />



