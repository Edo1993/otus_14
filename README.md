# PAM

[Разбор методички с практикой](https://github.com/Edo1993/otus_14/blob/master/pam_practice.md) 

Домашнее задание:

1. Запретить всем пользователям, кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников
* дать конкретному пользователю права работать с докером и возможность рестартить докер сервис

[Vagrantfile + скрипт](https://github.com/Edo1993/otus_14/tree/master/homework) 

Проверить выполнение дз:
- в выходные должно пускать под булочкой (bun), под печенькой (cookie) не пустит.

- Пользователь dockerUser может выполнять команду ```systemctl restart docker```, проверить, что докер перезапущен ``` systemctl status docker```

На стендовой виртуальной машине создадим 2х пользователей:
```
sudo useradd bun && \
sudo useradd cookie
```
Назначим им пароли:
```
echo "Otus2019"|sudo passwd --stdin bun &&\
echo "Otus2019" | sudo passwd --stdin cookie
```
Чтобы быть уверенными, что на стенде разрешен вход через ssh по паролю выполним:
```
sudo bash -c "sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config && systemctl restart sshd.service"
```
Добавим группу admin
```
sudo groupadd admin
```
Добавим пользователя bun в группу admin 
```
sudo usermod -G admin bun
```

# Модуль pam_time

Модуль pam_time позволяет достаточно гибко настроить доступ пользователя с учетом времени. Настройки данного модуля хранятся в файле /etc/security/time.conf.
```
cd /etc/security/
sudo vi time.conf
```
Добавим в конец файла строку
```
*;*;!%admin;!SaSu
```
Теперь настроим PAM, так как по-умолчанию данный модуль не подключен.
Для этого приведем файл ```/etc/pam.d/sshd``` к виду:
```
...
account required pam_nologin.so
account required pam_time.so
...
```
