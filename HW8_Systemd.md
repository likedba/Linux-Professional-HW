# Systemd  
1. создаём файл с конфигурацией для сервиса  
`vim /etc/default/watchlog`  

```bash
# Configuration file for my watchlog service
# Place it to /etc/default

# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
```  

2. создаем /var/log/watchlog.log и пишем туда строки на своё усмотрение,
плюс ключевое слово ‘ALERT’  
`vim /var/log/watchlog.log`  
```bash
ALERT
ALERT
```  

3. Создаем скрипт который ищет ключевое слово в логе  
`vim /opt/watchlog.sh`  
```bash
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
```  

5. Создадим юнит для сервиса  
`vim /etc/systemd/system/watchlog.service`  

```bash
[Unit]
Description=My watchlog service

[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG
```  

6. Создадим юнит для таймера  
`vim /etc/systemd/system/watchlog.timer`

```bash
[Unit]
Description=Run watchlog script every 30 second

[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service

[Install]
WantedBy=multi-user.target
```  

7. Запускаем таймер  
`systemctl start watchlog.timer`  
<img width="832" height="395" alt="image" src="https://github.com/user-attachments/assets/53e6ae07-cc91-4401-af4a-60ffc0d9cec4" />  

# Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта  

8. Устанавливаем spawn-fcgi и необходимые для него пакеты  
`apt install spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid -y`  

9. создать файл с настройками для будущего сервиса в файле /etc/spawn-fcgi/fcgi.conf  
`vim /etc/spawn-fcgi/fcgi.conf`  

```bash
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
```  

10. Создать юнит  
`vim /etc/systemd/system/spawn-fcgi.service`  

```bash
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/spawn-fcgi/fcgi.conf
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
```

11. запустить сервис  
`systemctl start spawn-fcgi`  
<img width="711" height="240" alt="image" src="https://github.com/user-attachments/assets/d20945ac-9a1e-4f11-a376-02accc0a8bf4" />  

# Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно  

12. Установим Nginx из стандартного репозитория  
`apt install nginx -y`

13. Для запуска нескольких экземпляров сервиса модифицируем исходный service для использования различной конфигурации, а также PID-файлов. Для этого создадим новый Unit для работы с шаблонами  
`vim /etc/systemd/system/nginx@.service`  

```bash
# Stop dance for nginx
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the nginx process.
# If, after 5s (--retry QUIT/5) nginx is still running, systemd takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if nginx is alive, systemd sends
# SIGKILL to all the remaining processes in the process group (KillMode=mixed).
#
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx-%I.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%I.conf -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx-%I.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```  

14. создать два файла конфигурации  
`vim /etc/nginx/nginx-first.conf`  
<img width="351" height="204" alt="image" src="https://github.com/user-attachments/assets/55371211-a00f-4ff6-b31c-79c4577ddecd" />  

`vim /etc/nginx/nginx-second.conf`  
<img width="350" height="203" alt="image" src="https://github.com/user-attachments/assets/351f1044-485c-4a88-99c4-530bc1d07e0b" />  

15. Проверим рабооту сервисов
```bash
systemctl start nginx@first
systemctl start nginx@second
systemctl status nginx@second
```
<img width="1208" height="255" alt="image" src="https://github.com/user-attachments/assets/82c3fe5a-5b7e-4d0a-a599-b518753cbe48" />

`ss -tnulp | grep nginx`
<img width="1244" height="70" alt="image" src="https://github.com/user-attachments/assets/0d6d8b33-4374-44ff-926a-e3a0db4300c7" />

`ps afx | grep nginx`
<img width="1045" height="123" alt="image" src="https://github.com/user-attachments/assets/e82762ac-c51c-469c-a528-fb32b63b104b" />
