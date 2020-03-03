## OTUS-Linux-Lesson 6  
# Задания основные (без *)  

Задание 1. Попасть в систему без пароля несколькими способами

Логичнее, пожалуй, не переписывать методичку, а говорить о том, что не получилось и как это решилось.  

В качестве образа был взять centos/7.  

При реализации первого способа (через init=/bin/sh) возникло две проблемы:  
1) Система игнорировала добавление в конец строки с linux16, как init, так и rd.break. И это не смотря на отсутствие rhgb и quiet.  
Т.е. система просто загружалась до аутентификации в обычном режиме.  
Это вылечилось обрезанием строки по самый ro. Ниже изначальное состояние строки:  
```
linux16 /boot/vmlinuz-3.10.0-957.12.2.el7.x86_64 root=UUID=8ac075e3-1124-4bb6-bef7-a6811bf8b870 ro /
no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop /
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

Подтверждение выполнения второго и третьего задания - в графических файлах в репозитории. По методичке и без сложностей.
