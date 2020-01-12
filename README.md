# PAM

[Разбор методички с практикой](https://github.com/Edo1993/otus_14/blob/master/pam_practice.md) 

Домашнее задание:

1. Запретить всем пользователям, кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников
* дать конкретному пользователю права работать с докером и возможность рестартить докер сервис

[Vagrantfile + скрипт](https://github.com/Edo1993/otus_14/tree/master/homework) 

Проверить выполнение дз:
- в выходные должно пускать под bun, под cookie не пустит.

- Пользователь dockerUser может выполнять команду ```systemctl restart docker```, проверить, что докер перезапущен ``` systemctl status docker```



Описание работы.

*Запретить всем пользователям, кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников*

Реализовано через ```pam_script```.

Установлены epel-release + pam_script
```
yum install epel-release -y
yum install pam_script -y
```
Добавляем в ```/etc/pam.d/sshd``` соответствующую запись ```auth  required  pam_script.so```.
Сам скрипт скопировали из текущей директории на vm и выдали права на выполнение
```
chmod +x /vagrant/pam_script
cp /vagrant/pam_script /etc/
```
Создаём группу *admin*, создаём булочку и сразу добавляем её в админы при создании, задаём пароль.
```
groupadd admin
useradd -G admin bun
echo "bun:Otus2019" | chpasswd
```
Создаём печеньку, задаём пароль.
```
useradd cookie
echo "cookie:Otus2019" | chpasswd
```
Чтобы быть уверенными, что на стенде разрешен вход через ssh по паролю выполним:
```
bash -c "sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config"
```
Перезапускаем сервис ```systemctl restart sshd``` - можно проверять


____________________________________
*дать конкретному пользователю права работать с докером и возможность рестартить докер сервис*

Создаём пользователя, задаём пароль. Группа ```docker``` создаётся в процессе установки docker'а, поэтому её отдельно не задаю.
```
useradd dockerUser
echo "dockerUser:Otus2019" | chpasswd
```
Манипуляции для установки docker'а:
```
yum check-update
curl -fsSL https://get.docker.com/ | sh
systemctl start docker
```
Пользователя dockerUser добавляем в группу docker'а + даём ему админские права для возможности рестарта сервиса.
```
usermod -aG docker dockerUser
usermod -G wheel dockerUser
```

Перезапускаем docker ```systemctl restart docker```.
