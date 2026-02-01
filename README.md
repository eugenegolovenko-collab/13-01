# Домашнее задание к занятию "`Уязвимости и атаки на информационные системы`" - `Евгений Головенко`
---
### Задание 1

Скачайте и установите виртуальную машину Metasploitable: https://sourceforge.net/projects/metasploitable/.

Это типовая ОС для экспериментов в области информационной безопасности, с которой следует начать при анализе уязвимостей.

Просканируйте эту виртуальную машину, используя **nmap**.

Попробуйте найти уязвимости, которым подвержена эта виртуальная машина.

Сами уязвимости можно поискать на сайте https://www.exploit-db.com/.

Для этого нужно в поиске ввести название сетевой службы, обнаруженной на атакуемой машине, и выбрать подходящие по версии уязвимости.

Ответьте на следующие вопросы:

- Какие сетевые службы в ней разрешены?
- Какие уязвимости были вами обнаружены? (список со ссылками: достаточно трёх уязвимостей)
  
*Приведите ответ в свободной форме.*  

### Решение 1

Подготовка стенда:
- VirtualBox
- Kali Linux (атакующая машина) 
- Metasploitable 2 (целевая машина)

Сеть:
- Kali: 2 интерфейса:
   - eth0 — host-only 192.168.56.101/24
   - eth1 — NAT 10.0.3.15/24 (для связи с внешним миром, для обновлений, например)

- Metasploitable:
   - 192.168.56.102/24 — host-only

Вся учебно-познавательная деятельность осуществляется только внутри host-only сети из следующих соображений:
- Kali видит другие виртуалки через eth0, изолированный от внешней среды;
- Metasploitable изолирован внутри виртуальной сети и основной компьютер в безопасности;
- никто не пострадает.

Сканирование Metasploitable с использованием nmap:
```cmd
nmap -sS -sV -O -p- 192.168.56.102
```
-sS — SYN-сканирование;

-sV — определение версий сервисов;

-O — определение ОС;

-p- — сканирование всех TCP-портов.

В результате сканирования были обнаружены следующие открытые сетевые службы, получена информация об операционной системе:

```cmd
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.3.4
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
53/tcp    open  domain      ISC BIND 9.4.2
80/tcp    open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
512/tcp   open  exec        netkit-rsh rexecd
513/tcp   open  login       OpenBSD or Solaris rlogind
514/tcp   open  shell       Netkit rshd
1099/tcp  open  java-rmi    GNU Classpath grmiregistry
1524/tcp  open  bindshell   Metasploitable root shell
2049/tcp  open  nfs         2-4 (RPC #100003)
2121/tcp  open  ftp         ProFTPD 1.3.1
3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
3632/tcp  open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
5432/tcp  open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp  open  vnc         VNC (protocol 3.3)                                                
6000/tcp  open  X11         (access denied)                                                   
6667/tcp  open  irc         UnrealIRCd                                                        
6697/tcp  open  irc         UnrealIRCd                                                        
8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)                                                   
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1                                            
8787/tcp  open  drb         Ruby DRb RMI (Ruby 1.8; path /usr/lib/ruby/1.8/drb)
41164/tcp open  mountd      1-3 (RPC #100005)
45023/tcp open  java-rmi    GNU Classpath grmiregistry
48536/tcp open  nlockmgr    1-4 (RPC #100021)
60435/tcp open  status      1 (RPC #100024)
MAC Address: 08:00:27:5B:E5:B6 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: Hosts:  metasploitable.localdomain, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

В целом, видно что на хосте запущено большое количество сетевых сервисов, многие из которых устарели и потенциально уязвимы, например:

- vsftpd 2.3.4. Backdoor Command Execution. Сервис: FTP. Порт: 21. Уязвимость позволяет получить удалённый `shell` при подключении к FTP-серверу. Exploit-DB: https://www.exploit-db.com/exploits/17491 Тип: Remote Code Execution. Критичность: высокая.
- Samba (CVE-2007-2447). Remote Command Execution. Сервис: Samba (SMB). Порты: 139 / 445. Версия: 3.X
Уязвимость позволяет удалённо выполнять команды через некорректную обработку `username mapping`.
Exploit-DB: https://www.exploit-db.com/exploits/16320 Тип уязвимости: Remote Code Execution. Критичность: критическая.

- UnrealIRCd. Backdoored Server. Сервис: IRC. Порты: 6667, 6697. Версия: UnrealIRCd. Сервер содержит встроенный `backdoor`, позволяющий выполнять произвольные команды. Exploit-DB: https://www.exploit-db.com/exploits/16922 Тип уязвимости: Remote Code Execution. Критичность: критическая.

Еще дополнительно:
- distccd — служба доступна без аутентификации, позволяет удалённо выполнить произвольную команду.
- ProFTPD 1.3.1 — множественный Remote Code Execution.
- Apache Tomcat (8180) — включён Tomcat Manager. Тип уязвимости: Weak credentials. Remote Code Execution через Manager.
- nfs — доступ к файловой системе.
- rsh / rlogin — небезопасные устаревшие протоколы (в rexecd пароли передаются в открытом виде без шифрования).
- Bind shell на 1524 — прямой root-доступ.

---

### Задание 2

Проведите сканирование Metasploitable в режимах SYN, FIN, Xmas, UDP.

Запишите сеансы сканирования в Wireshark.

Ответьте на следующие вопросы:

- Чем отличаются эти режимы сканирования с точки зрения сетевого трафика?
- Как отвечает сервер?

*Приведите ответ в свободной форме.* 

### Решение 2

- SYN scan
```cmd
sudo nmap -sS -p 1-1000 192.168.56.102
```

Отправляются TCP-пакеты с флагом `SYN` в `well-known ports` (1-1000). Ответы: `SYN, ACK` -> open; `RST, ACK` -> closed.

Wireshark фильтр: `tcp.flags.syn == 1`

Пример:

Вывод `nmap`:

```cmd
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sS -p 1-1000 192.168.56.102   
[sudo] password for kali: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-31 13:06 -0500
Nmap scan report for 192.168.56.102
Host is up (0.0010s latency).
Not shown: 988 closed tcp ports (reset)
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
23/tcp  open  telnet
25/tcp  open  smtp
53/tcp  open  domain
80/tcp  open  http
111/tcp open  rpcbind
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
512/tcp open  exec
513/tcp open  login
514/tcp open  shell
MAC Address: 08:00:27:5B:E5:B6 (Oracle VirtualBox virtual NIC)
Nmap done: 1 IP address (1 host up) scanned in 0.71 seconds
```
<img width="1141" height="576" alt="Capture1" src="https://github.com/user-attachments/assets/29320e78-1fb5-4c78-8ee6-82ca0e0e0aa2" />
<img width="884" height="346" alt="Capture2" src="https://github.com/user-attachments/assets/00a2a459-13ed-4302-8939-8417a2c011db" />

на скрине видны порты, которые на `SYN` радостно отвечают `SYN-ACK` и готовы продолжить общение.

- FIN scan

По своей сути это отправка некорректного пакета `FIN` вне сессии.
```cmd
sudo nmap -sF -p 1-1000 192.168.56.102
```
Ожидаемые ответы: `RST, ACK` -> closed; не отвечает -> либо открыт, либо ответы фильтруются.

```cmd
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sF -p 1-1000 192.168.56.102
[sudo] password for kali: 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-01-31 14:09 -0500
Nmap scan report for 192.168.56.102
Host is up (0.00046s latency).
Not shown: 988 closed tcp ports (reset)
PORT    STATE         SERVICE
21/tcp  open|filtered ftp
22/tcp  open|filtered ssh
23/tcp  open|filtered telnet
25/tcp  open|filtered smtp
53/tcp  open|filtered domain
80/tcp  open|filtered http
111/tcp open|filtered rpcbind
139/tcp open|filtered netbios-ssn
445/tcp open|filtered microsoft-ds
512/tcp open|filtered exec
513/tcp open|filtered login
514/tcp open|filtered shell
MAC Address: 08:00:27:5B:E5:B6 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 2.33 seconds
```

Wireshark фильтр: `tcp.flags.fin == 1 || tcp.flags.reset == 1`

<img width="1180" height="914" alt="Capture3" src="https://github.com/user-attachments/assets/f436b6cc-aa64-49d8-8b50-23499662365f" />

На скрине это видно на примере обращения к портам 23 и 587.

Порт 23 на одиночный запрос с флагом `FIN` решил не отвечать, потому что он хотя открыт, ждет соединений, но полученный `FIN` не укладывается в логику протокола TCP, нет активного соединения и финализировать нечего. Поэтому игнор. Но это и есть признак открытости порта.

Порт же 587 получив неожиданный сегмент, когда нет соединения и вообще он закрыт, сбрасывает «несуществующее» соединение, но будучи вежливым портом, подтвердждает получение пакета (в ответе видны флаги `RST`, `ACK`).

В целом, можно сказать, что FIN-scan менее шумный, чем SYN-scan, так как открытые порты ничего не отвечают. Но данный способ однозначно не может сказать о статусе сканируемых портов, так как TCP-пакеты с флагом FIN могут быть отсеяны фаерволами и порты будут выглядеть как открытые.

- Xmas scan

```cmd
sudo nmap -sX -p 1-1000 192.168.56.102
```

Отправляется TCP-пакет с флагами: `FIN`, `PSH`, `URG`. Ответ: `RST` -> closed, нет ответа -> open / filtered.

```cmd
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sX -p 1-1000 192.168.56.102
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-01 05:17 -0500
Nmap scan report for 192.168.56.102
Host is up (0.00043s latency).
Not shown: 988 closed tcp ports (reset)
PORT    STATE         SERVICE
21/tcp  open|filtered ftp
22/tcp  open|filtered ssh
23/tcp  open|filtered telnet
25/tcp  open|filtered smtp
53/tcp  open|filtered domain
80/tcp  open|filtered http
111/tcp open|filtered rpcbind
139/tcp open|filtered netbios-ssn
445/tcp open|filtered microsoft-ds
512/tcp open|filtered exec
513/tcp open|filtered login
514/tcp open|filtered shell
MAC Address: 08:00:27:5B:E5:B6 (Oracle VirtualBox virtual NIC)
Nmap done: 1 IP address (1 host up) scanned in 1.98 seconds
```

<img width="1176" height="890" alt="Capture4" src="https://github.com/user-attachments/assets/7c346d7b-3313-429f-9cea-5fa6986dd43d" />

Скрин, на примере порта 135, показывает, что в ответ на абсурдный по логике TCP запрос хост отвечает, что порт закрыт (`RST`, `ACK`).

`Xmas scan` позволяет выявить закрытые порты, так же как и предыдущий `FIN scan`. Под это сканирование есть стандартные сигнатуры IDS, собственно как и для `FIN scan`, которые позволяют сразу выявлять его. *Перед проведением подобных видов сканирований настоятельно рекомендуется согласовать это с владельцем системы и командой безопасности.*

- UDP scan

```cmd
sudo nmap -sU -p 1-1000 192.168.56.102
```

Вся логика сканирования строится на косвенных признаках. Исходящий пакет, это пустая UDP-датаграмма на конкретный порт.

Если порт закрыт, то в ответ отправляется ICMP-пакет, содержащий:

```
ICMP Destination Unreachable
Type 3 Code 3 (Port unreachable)
```

Если порт открыт, то ответа может не быть, либо это будет UDP-пакет.

Для примера можно понаблюдать за поведением порта 547:

<img width="1174" height="892" alt="Capture5" src="https://github.com/user-attachments/assets/8cd8e531-e2d6-4653-92f4-d65ef1b3f5f9" />

Все закрытые порты пожно посмотреть фильтром `icmp && icmp.type == 3`

Можно отметить, что `UDP scan` достаточно неспешный, потому что приходитьтся повторять пакеты в силу природы протокола, выжидать таймауты.

Наиболее интересные для `UDP scan` следующие порты и сервисы, который на них живут:

- 53, DNS
- 67/68, DHCP
- 69, TFTP
- 123, NTP
- 161, SNMP

