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

