# Размещаем свой RPM в своем репозитории
sudo apt update && sudo apt install -y wget devscripts debhelper build-essential cmake gcc git nano reprepro apt-utils
mkdir rpm && cd rpm
sudo nano /etc/apt/sources.list
apt source nginx
cd nginx-1.24.0
Установим зависимости для сборки
sudo apt build-dep nginx
