# otus_14
# PAM
1. Запретить всем пользователям, кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников

# Подготовка стенда к работе
На стендовой виртуальной машине создадим 3х пользователей:
```
sudo useradd day && \
sudo useradd night && \
sudo useradd friday
```
Назначим им пароли:
```
echo "Otus2019"|sudo passwd --stdin day &&\
echo "Otus2019" | sudo passwd --stdin night &&\
echo "Otus2019" | sudo passwd --stdin friday
```
Чтобы быть уверенными, что на стенде разрешен вход через ssh по паролю выполним:
```
sudo bash -c "sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config && systemctl restart sshd.service"
```
# Модуль pam_time

Модуль pam_time позволяет достаточно гибко настроить доступ пользователя с учетом времени. Настройки данного модуля хранятся в файле /etc/security/time.conf.
```
cd /etc/security/
sudo vi time.conf
```
Добавим в конец файла строки
```
*;*;day;Al0800-2000
*;*;night;!Al0800-2000
*;*;friday;Fr
```
Теперь настроим PAM, так как по-умолчанию данный модуль не подключен.
Для этого приведем файл ```/etc/pam.d/sshd``` к виду:
```
...
account required pam_nologin.so
account required pam_time.so
...
```
![Image alt](https://github.com/Edo1993/otus_14/raw/master/Screenshot_76.png)

После чего в отдельном терминале можно проверить доступ к серверу по ssh для созданных пользователей.
Пользователь day

![Image alt](https://github.com/Edo1993/otus_14/raw/master/Screenshot_69.png)

Пользователь night

![Image alt](https://github.com/Edo1993/otus_14/raw/master/Screenshot_70.png)

Пользователь friday

![Image alt](https://github.com/Edo1993/otus_14/raw/master/Screenshot_71.png)

Проверяем попутки успешных входов ```last```
![Image alt](https://github.com/Edo1993/otus_14/raw/master/Screenshot_75.png)

После этого захотелось проверить, что для пользователя friday всё работает корректно. Изменила в ```/etc/security/time.conf``` для пользователя friday день на субботу Sa:

![Image alt](https://github.com/Edo1993/otus_14/raw/master/Screenshot_72.png)

Систему рестартанула - пользователю отказано в доступе (без рестратра - его спокойно пускало).

![Image alt](https://github.com/Edo1993/otus_14/raw/master/Screenshot_73.png)

Проверяем попутки Неуспешных входов ```lastb```

![Image alt](https://github.com/Edo1993/otus_14/raw/master/Screenshot_75_1.png)

# Модуль pam_exec

*Предварительную подготовку стенда также выполняем, если переходим на другую машину*

Еще один способ реализовать задачу - выполнить при подключении пользователя скрипт, в котором необходимая информация обработается самостоятельно.
Удалим из /etc/pam.d/sshd изменения из предыдущего этапа и приведем его к следующему виду:
```
...
account required pam_nologin.so
account required pam_exec.so /usr/local/bin/test_login.sh
...
```
![Image alt](https://github.com/Edo1993/otus_14/raw/master/21.png)

Добавлен модуль ```pam_exec``` и, в качестве параметра, указан скрипт, который осуществит необходимые проверки. [Скрипт](https://gist.github.com/dmitry-lyutenko/39bf8afe5d1f6fc2d48b09c325706495) взят готовый.
При запуске данного скрипта PAM-модулем будет передана переменная окружения PAM_USER, содержащая имя пользователя.

*У меня почему-то были косяки с тем, что после всех действий проверки не работали и пускало всё равно всех пользаков, поэтому скрипту я изменила права на выполнение*

```
chmod 777 test_login.sh 
```
После этого проверки на вход начали выполняться корректно.

Пользователь day

![Image alt](https://github.com/Edo1993/otus_14/raw/master/24.png)

Пользователь night

![Image alt](https://github.com/Edo1993/otus_14/raw/master/23.png)

Пользователь friday

![Image alt](https://github.com/Edo1993/otus_14/raw/master/22.png)

# Модуль pam_script

