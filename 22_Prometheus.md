1. Исходная инфраструктура на примере файла /etc/hosts на всех машинах
127.0.0.1 localhost
127.0.1.1 vmubuntu-1
192.168.121.141 vmubuntu-1
192.168.121.142 vmubuntu-2
192.168.121.143 vmubuntu-3
192.168.121.144 vmubuntu-4
192.168.121.145 vmubuntu-5
192.168.121.146 vmubuntu-6
192.168.121.147 vmubuntu-7
192.168.121.148 vmubuntu-8
192.168.121.135 vmpatronidb1-test-17
192.168.121.136 vmpatronidb2-test-17
192.168.121.137 vmpatronidb3-test-17

На vmubuntu-7 будет установлен сервер prometheus с бд postgresql
На vmubuntu-8 будет установлен сервер grafana с бд postgresql
На хостах с развернутым кластером patroni будут установлены node exporter и postgres-exporter для получения системных метрик и метрик СУБД
vmpatronidb1-test-17
vmpatronidb2-test-17
vmpatronidb3-test-17

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

2. Обновляем все машины
sudo apt update && sudo apt upgrade -y

3. Устанавливаем на всех машинах node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
tar xvf node_exporter-1.8.1.linux-amd64.tar.gz
cp node_exporter-1.8.1.linux-amd64/node_exporter /usr/local/bin/
chown root:root /usr/local/bin/node_exporter
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter

Проверяем работу экспортера
<img width="740" height="304" alt="image" src="https://github.com/user-attachments/assets/fb40c34e-d060-4363-bd0b-29f1ae033f75" />

4. Устанавливаем postgres_exporter на машины с СУБД
wget https://github.com/prometheus-community/postgres_exporter/releases/download/v0.15.0/postgres_exporter-0.15.0.linux-amd64.tar.gz
tar xvf postgres_exporter-0.15.0.linux-amd64.tar.gz
sudo cp postgres_exporter-0.15.0.linux-amd64/postgres_exporter /usr/local/bin/
sudo chown root:root /usr/local/bin/postgres_exporter
sudo groupadd --system postgres_exporter
useradd --no-create-home --shell /bin/false -g postgres_exporter postgres_exporter
Создаем юзера в бд для postgres_exporter и экстеншн pg_stat_statements
CREATE USER postgres_exporter WITH PASSWORD 'postgres_exporter' LOGIN;
GRANT pg_monitor TO postgres_exporter;
CREATE EXTENSION pg_stat_statements;
Создаем конфиг
echo "DATA_SOURCE_NAME=\"postgresql://postgres_exporter:postgres_exporter@localhost:5432/postgres?sslmode=disable\"" | sudo tee /etc/sysconfig/postgres_exporter
sudo tee /etc/systemd/system/postgres_exporter.service > /dev/null <<EOF
> [Unit]
> Description=PostgreSQL Exporter
> After=network.target
>
> [Service]
> User=postgres_exporter
> Group=postgres_exporter
> EnvironmentFile=/etc/sysconfig/postgres_exporter
> ExecStart=/usr/local/bin/postgres_exporter
> Restart=always
>
> [Install]
> WantedBy=multi-user.target
> EOF

sudo systemctl daemon-reload
sudo systemctl start postgres_exporter
sudo systemctl enable postgres_exporter

Проверяем postgres_exporter
<img width="540" height="246" alt="image" src="https://github.com/user-attachments/assets/70078c4d-10fe-4c3d-ad61-59db7ad57ed9" />

Установка сервера prometheus на vmubuntu-7
wget https://github.com/prometheus/prometheus/releases/download/v2.51.2/prometheus-2.51.2.linux-amd64.tar.gz
tar xvf prometheus-2.51.2.linux-amd64.tar.gz
sudo groupadd --system prometheus
useradd --no-create-home --shell /bin/false -g prometheus prometheus
sudo chown prometheus:prometheus /etc/prometheus /var/lib/prometheus
sudo cp prometheus-2.51.2.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.51.2.linux-amd64/promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
sudo cp -r prometheus-2.51.2.linux-amd64/consoles /etc/prometheus/
sudo cp -r prometheus-2.51.2.linux-amd64/console_libraries /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus/*

global:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
vim /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node_linux'
    static_configs:
      - targets: 
        - 'vmubuntu-1:9100'
        - 'vmubuntu-2:9100'
        - 'vmubuntu-3:9100'
        - 'vmubuntu-4:9100'
        - 'vmubuntu-5:9100'
        - 'vmubuntu-6:9100'
        labels:
          group: 'ubuntu_servers'
      - targets:
        - 'vmubuntu-7:9100'
        - 'vmubuntu-8:9100'
        labels:
          group: 'monitoring_servers'
      - targets:
        - 'vmpatronidb1-test-17:9100'
        - 'vmpatronidb2-test-17:9100'
        - 'vmpatronidb3-test-17:9100'
        labels:
          group: 'database_servers'
  - job_name: 'postgres'
    static_configs:
      - targets:
        - 'vmpatronidb1-test-17:9187'
        - 'vmpatronidb2-test-17:9187'
        - 'vmpatronidb3-test-17:9187'
        labels:
          group: 'postgres_servers'

sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml

Создаем юнит для prometheus
sudo tee /etc/systemd/system/prometheus.service > /dev/null <<EOF
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus

Проверяем работу сервера prometeus



