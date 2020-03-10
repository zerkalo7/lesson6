## OTUS-Linux-Lesson 6  
# Задания основные (без *)  

### Задание 1. Попасть в систему без пароля несколькими способами

Логичнее, пожалуй, не переписывать методичку, а говорить о том, что не получилось и как это решилось.  

В качестве образа был взят centos/7.  

При реализации первого способа (через init=/bin/sh) возникло две проблемы:  
1) Система игнорировала добавление в конец строки с linux16, как init, так и rd.break. И это не смотря на отсутствие rhgb и quiet.  
Т.е. система просто загружалась до аутентификации в обычном режиме.  
Это вылечилось обрезанием строки по самый ro. Ниже изначальное состояние строки:  
```
linux16 /boot/vmlinuz-3.10.0-957.12.2.el7.x86_64 root=UUID=8ac075e3-1124-4bb6-bef7-a6811bf8b870 ro \
no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop \
crashkernel=auto LANG=en_US.UTF-8
``` 
Похоже, что дело в записях console, но на 100% не уверен.

2) Второй сложностью стало то, что после смены пароля и переазгруки, система перестала пускать любого пользователя.  
Оказалось, что дело во включенном SELinux. При отключении всё становилось ок, но, кто сказал, что на всех серверах он будет отключен?  
Хэши паролей хранятся в /etc/shadow. Если файл меняется при отключенном SELinux, то он обижается.  
Соответетственно, надо инициализировать SELinux перед манипуляциями. 
```
/usr/sbin/load_policy -i
mount -o remount,rw /
passwd root
mount -o remount,ro /
exec /sbin/init 6
```
Иии... Работает. :)

Похоже, что в этом случае перерывание загрузки происходит еще до момента принятия решения о том, какие политики должны быть загружены.  
Насколько я понимаю, что в этом случае /.autorelabel тоже должен помочь. 

Во втором и третьем способах всё прошло по методичке.

### Задание 2. Переименовать VG  

В качество образа используем стенд с 3-его занятия.
https://gitlab.com/otus_linux/stands-03-lvm  

Переименовываем VG. 
```
vgrename VolGroup00 OtusRoot 
```

Правим /etc/fstab, /etc/default/grub и /boot/grub2/grub.cfg (заменяем старое название на новое).
Пересоздаем intird c новым названием.
```
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```
  
После перезагрузки выполняем 
**vgs**  

И убеждаемся, что все нормально.

> VG       #PV #LV #SN Attr   VSize   VFree  
> OtusRoot   1   2   0 wz--n- <38.97g    0
  
 
### Задание 3. Добавить модуль в initrd

Переходим в папку со скриптами модулей и создаем там папку
```
cd /usr/lib/dracut/modules.d/
mkdir 01test
``` 

Скачиваем два скрипта из задания.
```
yum install -y wget
cd 01test
wget https://gist.githubusercontent.com/lalbrekht/e51b2580b47bb5a150bd1a002f16ae85/raw/80060b7b300e193c187bbcda4d8fdf0e1c066af9/gistfile1.txt
mv gistfile1.txt module-setup.sh
wget https://gist.githubusercontent.com/lalbrekht/ac45d7a6c6856baea348e64fac43faf0/raw/69598efd5c603df310097b52019dc979e2cb342d/gistfile1.txt
mv gistfile1.txt test.sh
```

Пересобираем initrd
```
dracut -f -v
```

Идём в /boot/grub2/grub.cfg и удаляем из строки "linux16" rghb и quiet. Перезагружаемся и наблюдаем пингвина. 
