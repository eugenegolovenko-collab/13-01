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


