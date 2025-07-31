# Написать скрипт для CRON, который раз в час будет формировать письмо и отправлять на заданную почту  

Необходимая информация в письме:  
  
Список IP адресов (с наибольшим кол-вом запросов) с указанием кол-ва запросов c момента последнего запуска скрипта;  
Список запрашиваемых URL (с наибольшим кол-вом запросов) с указанием кол-ва запросов c момента последнего запуска скрипта;  
Ошибки веб-сервера/приложения c момента последнего запуска;  
Список всех кодов HTTP ответа с указанием их кол-ва с момента последнего запуска скрипта.  
Скрипт должен предотвращать одновременный запуск нескольких копий, до его завершения.  

1. Создадим лог сервера и логу ошибок  
```bash
vim /var/log/nginx/access.log
vim /var/log/nginx/error.log
```  

2. Создадим скрипт по ТЗ  
`vim /usr/local/bin/nginx_monitor.sh`

```bash
#!/bin/bash

# Блокировка от множественного запуска
LOCKFILE=/tmp/monitor_script.lock
if [ -e ${LOCKFILE} ] && kill -0 `cat ${LOCKFILE}`; then
    echo "Script is already running"
    exit
fi

trap "rm -f ${LOCKFILE}; exit" INT TERM EXIT
echo $$ > ${LOCKFILE}

# Конфигурация
LOG_FILE="/var/log/nginx/access.log"  # Укажите путь к вашему лог-файлу
ERROR_LOG_FILE="/var/log/nginx/error.log"  # Укажите путь к логу ошибок
TMP_FILE="/tmp/access_log_processed.tmp"
EMAIL="a.salugin@gmail.com" 
LAST_RUN_FILE="/tmp/monitor_script_last_run"

# Определяем временной диапазон
NOW=$(date +%Y-%m-%d\ %H:%M:%S)
if [ -f "$LAST_RUN_FILE" ]; then
    LAST_RUN=$(cat "$LAST_RUN_FILE")
else
    LAST_RUN=$(date -d "1 hour ago" +%Y-%m-%d\ %H:%M:%S)
fi

# Сохраняем время последнего запуска
echo "$NOW" > "$LAST_RUN_FILE"

# Подготавливаем письмо
MAIL_FILE="/tmp/nginx_monitor_mail.txt"
echo "Отчет за период с $LAST_RUN по $NOW" > $MAIL_FILE
echo "" >> $MAIL_FILE

# 1. Список IP с наибольшим количеством запросов
echo "Топ IP адресов по количеству запросов:" >> $MAIL_FILE
awk -vDate="$LAST_RUN" -vDate2="$NOW" '$4" "$5 >= "["Date && $4" "$5 < "["Date2 {print $1}' "$LOG_FILE" | sort | uniq -c | sort -nr | head -n 10 >> $MAIL_FILE
echo "" >> $MAIL_FILE

# 2. Список запрашиваемых URL с наибольшим количеством запросов
echo "Топ URL по количеству запросов:" >> $MAIL_FILE
awk -vDate="$LAST_RUN" -vDate2="$NOW" '$4" "$5 >= "["Date && $4" "$5 < "["Date2 {print $7}' "$LOG_FILE" | sort | uniq -c | sort -nr | head -n 10 >> $MAIL_FILE
echo "" >> $MAIL_FILE

# 3. Ошибки веб-сервера/приложения
echo "Ошибки веб-сервера/приложения:" >> $MAIL_FILE
awk -vDate="$LAST_RUN" -vDate2="$NOW" '$1" "$2 >= Date && $1" "$2 < Date2' "$ERROR_LOG_FILE" >> $MAIL_FILE
echo "" >> $MAIL_FILE

# 4. Список всех кодов HTTP ответа
echo "Коды HTTP ответа:" >> $MAIL_FILE
awk -vDate="$LAST_RUN" -vDate2="$NOW" '$4" "$5 >= "["Date && $4" "$5 < "["Date2 {print $9}' "$LOG_FILE" | sort | uniq -c | sort -nr >> $MAIL_FILE

# Отправка письма
mail -s "NGINX Monitor Report ($LAST_RUN - $NOW)" "$EMAIL" < $MAIL_FILE

# Очистка
rm -f ${LOCKFILE}
rm -f $MAIL_FILE
```  

3. Добавим выполнение скрипту  
`chmod +x /usr/local/bin/nginx_monitor.sh`  

4. Установим мэйл сервис  
`apt install mailutils`  

5. Добавим скрипт в крон  
`crontab -e`  
`0 * * * * /usr/local/bin/nginx_monitor.sh`
