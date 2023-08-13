Защита сети. Васёв А.В.

## Задание 1

### Проведите разведку системы и определите, какие сетевые службы запущены на защищаемой системе:

```java
sudo nmap -sA < ip-адрес >
sudo nmap -sT < ip-адрес >
sudo nmap -sS < ip-адрес >
sudo nmap -sV < ip-адрес >
```
При выполнении nmap с ключами -sA, -sT, -sS, -sV а логах suricata (/var/log/suricata/fast.log) каких либо событий не фиксируется. Для проверки работоспособности выполнена DDoS-атака на хост Suricata
```java
hping3 -S -p 80 --flood --rand-source 192.168.1.1 -I enp0s3 -c 50
```
в результате выполненя атаки с логах работы suricata зафиксированы записи об отбрасывании пакетов с атакующего хоста.

08/13/2023-09:50:20.022490  [**] [1:2400001:3707] ET DROP Spamhaus DROP Listed Traffic Inbound group 2 [**] [Classification: Misc Attack] [Priority: 2] {TCP} 42.167.144.20:10069 -> 192.168.1.1:80

![alt text](https://github.com/rus42/NetworkProtection/blob/main/Task_1.1.png)

В логах fail2ban (/var/log/auth.log) при выполнении nmap с ключами -sA, -sT, -sS, также каких либо событий не фиксируется.
При выполнении nmap с ключом -sV в логах работы fail2ban зафиксирована запись о закрытии соединения с атакующего хоста.

Aug 13 10:06:10 vm1 sshd[3502]: error: kex_exchange_identification: Connection closed by remote host
Aug 13 10:06:10 vm1 sshd[3502]: Connection closed by 192.168.1.2 port 46658

![alt text](https://github.com/rus42/NetworkProtection/blob/main/Task_1.2.png)

## Задание 2

### Проведите атаку на подбор пароля для службы SSH:

```java
hydra -L users.txt -P pass.txt < ip-адрес > ssh
```

    Настройка hydra:

    создайте два файла: users.txt и pass.txt;
    в каждой строчке первого файла должны быть имена пользователей, второго — пароли. В нашем случае это могут быть случайные строки, но ради эксперимента можете добавить имя и пароль существующего пользователя.

В логах suricata зафиксированы записи о событиях попытки подключения на хост по 22 порту

```java
08/13/2023-11:00:25.949286  [**] [1:2260002:1] SURICATA Applayer Detect protocol only one direction [**] [Classification: Generic Protocol Command Decode] [Priority: 3] {TCP} 192.168.1.2:51474 -> 192.168.1.1:22
```

В логах fail2ban зафиксированы записи о событиях попытки подключения на хост по 22 порту с использованием имен и паролей из файлов users.txt и pass.txt. Также фиксируются события о превышении допустимого количества попыток аутентификации для пользователя и для такого пользователя происходит разрыв соединения.

```java
Aug 13 11:04:56 vm1 sshd[2506]: Failed password for invalid user 123456789 from 192.168.1.2 port 42734 ssh2
Aug 13 11:04:56 vm1 sshd[2505]: Failed password for invalid user 123456789 from 192.168.1.2 port 42730 ssh2
Aug 13 11:04:56 vm1 sshd[2507]: Failed password for invalid user 123456789 from 192.168.1.2 port 42736 ssh2
Aug 13 11:04:56 vm1 sshd[2508]: Failed password for invalid user 123456789 from 192.168.1.2 port 42740 ssh2
Aug 13 11:04:56 vm1 sshd[2506]: pam_unix(sshd:auth): check pass; user unknown
Aug 13 11:04:56 vm1 sshd[2505]: pam_unix(sshd:auth): check pass; user unknown
Aug 13 11:04:56 vm1 sshd[2507]: pam_unix(sshd:auth): check pass; user unknown
Aug 13 11:04:56 vm1 sshd[2508]: pam_unix(sshd:auth): check pass; user unknown
Aug 13 11:04:58 vm1 sshd[2506]: Failed password for invalid user 123456789 from 192.168.1.2 port 42734 ssh2
Aug 13 11:04:58 vm1 sshd[2505]: Failed password for invalid user 123456789 from 192.168.1.2 port 42730 ssh2
Aug 13 11:04:58 vm1 sshd[2507]: Failed password for invalid user 123456789 from 192.168.1.2 port 42736 ssh2
Aug 13 11:04:58 vm1 sshd[2508]: Failed password for invalid user 123456789 from 192.168.1.2 port 42740 ssh2
Aug 13 11:04:58 vm1 sshd[2506]: error: maximum authentication attempts exceeded for invalid user 123456789 from 192.168.1.2 port 42734 ssh2 [preauth]
Aug 13 11:04:58 vm1 sshd[2506]: Disconnecting invalid user 123456789 192.168.1.2 port 42734: Too many authentication failures [preauth]
```

В случае если в используемых файлах users.txt и pass.txt присутствует информация о существуещем логине и пароле, то в логе fail2ban зафиксировано событие об успешном подключении

```java
Aug 13 11:13:04 vm1 sshd[2811]: Accepted password for netology from 192.168.1.2 port 53310 ssh2
Aug 13 11:13:04 vm1 sshd[2811]: pam_unix(sshd:session): session opened for user netology(uid=1000) by (uid=0)
```

    Включение защиты SSH для Fail2Ban:

    открыть файл /etc/fail2ban/jail.conf,
    найти секцию ssh,
    установить enabled в true.

После включения защиты SSH сервера от перебора паролей на атакующем хосте получено сообщение о недоступности атакуемого порта, т.е. его блокировке на атакуемом хосте.

![alt text](https://github.com/rus42/NetworkProtection/blob/main/Task_2.png)
