# GRUB  

1. Включить отображение меню Grub  
`vim /etc/default/grub`  
```bash
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=10
```  
```bash
update-grub
reboot
```  
<img width="914" height="683" alt="image" src="https://github.com/user-attachments/assets/f165026c-47f9-4079-9412-8f517f306f0f" />  

2. Попасть в систему без пароля несколькими способами. Способ 1
<img width="640" height="377" alt="image" src="https://github.com/user-attachments/assets/ce4f7549-0b2b-42aa-ac16-a7b58f0976f6" />  
изменена строка, начинающаяся с linux, добавлено в конец init=/bin/bash и сtrl-x с загрузкой в систему
<img width="453" height="80" alt="image" src="https://github.com/user-attachments/assets/31d5040d-22cc-4197-92c6-0bd0f44b8d7f" />  
```bash
mount -o remount,rw /
touch /newfile
ls /new*
```  
<img width="406" height="100" alt="image" src="https://github.com/user-attachments/assets/6521eb40-29c7-4735-9eaa-bade99548579" />  

3. Попасть в систему без пароля несколькими способами. Способ 2  
 
<img width="726" height="404" alt="image" src="https://github.com/user-attachments/assets/a9dab434-915f-4c85-9760-c21b7379e838" />  
<img width="710" height="390" alt="image" src="https://github.com/user-attachments/assets/21e83c09-335d-4b81-98df-84b009de7b1c" />  
<img width="713" height="409" alt="image" src="https://github.com/user-attachments/assets/24ac222b-6c77-4da9-a1ea-fb1ce1755c48" />  
`vgs`  
<img width="400" height="50" alt="image" src="https://github.com/user-attachments/assets/4239a910-ec91-4bba-a145-6e6b0f09c814" />  
`vgrename ubuntu-vg ubuntu-otus`  
`vim /boot/grub/grub.cfg`  
Меняем везде название vg на новое  
`reboot`  
<img width="524" height="182" alt="image" src="https://github.com/user-attachments/assets/58e58de0-8a65-4706-ad9d-9e959bb91ce1" />  



