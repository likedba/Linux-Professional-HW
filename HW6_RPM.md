# Размещаем свой RPM в своем репозитории  
`sudo apt install -y wget devscripts build-essential debhelper dpkg-dev cmake git nano nginx libpcre3-dev zlib1g-dev libssl-dev`
2. Получение исходников NGINX  
```bash
mkdir -p ~/nginx-build && cd ~/nginx-build
apt source nginx
```  
3. Установим зависимости для сборки  
`sudo apt build-dep nginx`  
4. Получение модуля ngx_brotli  
```bash
cd ~
git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
cd ngx_brotli/deps/brotli
mkdir out && cd out
```  
5. Сборка модуля brotli  
```bash
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF \
    -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto" \
    -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto" \
    -DCMAKE_INSTALL_PREFIX=./installed ..
cmake --build . --config Release -j $(nproc) --target brotlienc
cd ../../..
```  
<img width="557" height="344" alt="image" src="https://github.com/user-attachments/assets/b5f5c9f5-d11f-439e-a8b0-29fa52292886" />  
<img width="584" height="140" alt="image" src="https://github.com/user-attachments/assets/b34e6460-d107-4921-9578-738c05c989e5" />  

6. Сборка NGINX  
```bash
cd ~/nginx-build/nginx-*
dpkg-buildpackage -b -uc -us
cd ~/nginx-build
ls *.deb
```  
<img width="651" height="77" alt="image" src="https://github.com/user-attachments/assets/5dc939bc-6c64-42e5-a213-4845fec0a12b" />  
<img width="813" height="137" alt="image" src="https://github.com/user-attachments/assets/d817ccac-47c4-464c-a355-f682c95abb45" />  

7. Установка собранного пакета  
```bash
sudo apt install ./nginx_*.deb
sudo systemctl start nginx
sudo systemctl status nginx
```  
<img width="953" height="261" alt="image" src="https://github.com/user-attachments/assets/cc462c07-92b4-4d8f-bb95-27b69fd15633" />  

## Создание собственного репозитория  

8. Настройка директории репозитория  
```bash
sudo mkdir -p /var/www/html/repo
sudo cp ~/nginx-build/*.deb /var/www/html/repo/
```  

9. Создание репозитория  
```bash
sudo apt install -y reprepro
cd /var/www/html/repo
mkdir conf
vim conf/distributions
```  
```bash
Origin: My Repository
Label: My Local Repo
Codename: noble
Architectures: amd64 arm64
Components: main
Description: My custom package repository
SignWith: yes
```  

10. Добавление пакетов в репозиторий  
`reprepro includedeb noble /var/www/html/repo/*.deb`  

На этом моменте происходит ошибка  
<img width="640" height="153" alt="image" src="https://github.com/user-attachments/assets/bf7e741e-6649-4bf7-87ab-b8883ca9979c" />  
reprepro пытается подписать пакеты GPG-ключом, но не находит подходящего  

Создание GPG-ключа для репозитория  
```bash
sudo apt install -y gnupg
gpg --gen-key
```  
Экспорт открытого ключа, настройка reprepro  
```bash
gpg --armor --export alextrask@inbox.ru | tee /var/www/html/repo/repo-key.gpg
vim /var/www/html/repo/conf/distributions
```  
```bash
Origin: My Repository
Label: My Local Repo
Codename: noble
Architectures: amd64 arm64
Components: main
Description: My custom package repository
SignWith: 09D757F88E92FEF74B77D421FF871B0AAA173CA2
```  

11. Повторное добавление пакетов в репозиторий, обновление репозитория  
```bash
cd /var/www/html/repo
reprepro includedeb noble /var/www/html/repo/*.deb
reprepro export noble
```  

12. Настройка вм vmubuntu-2 для использования репозитория  
```bash
wget -qO - http://192.168.121.141/repo/repo-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/my-repo.gpg
echo "deb [signed-by=/usr/share/keyrings/my-repo.gpg] http://192.168.121.141/repo noble main" | sudo tee /etc/apt/sources.list.d/my-repo.list
sudo apt update
```  

13. Установка пакетов из своего репозитория  
`sudo apt install nginx`  
<img width="782" height="185" alt="image" src="https://github.com/user-attachments/assets/5aa7bbf2-4d4d-4f11-a220-d3becbffbda5" />  
``

14. Проверка  
`nginx -V 2>&1 | grep brotli`  
проверка показывает, что nginx был собран без модуля brotli
вероятно, проблема в том, что система устанавливает стандартный пакет NGINX, а не из своего репозитория, собранный с модулем Brotli.  

После сборки с измененным именем nginx все равно ставится без модуля brotli  
