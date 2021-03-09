# Hack The Box - Academy

## This is my writeup and walkthrough for Academy machine  from Hack The Box.

![academy](https://user-images.githubusercontent.com/36403473/99427934-af2a5480-290e-11eb-8cb5-e6a888fc9174.png)

## `Enumeration`
   
#### 1-Nmap 
```
nmap -sC -sV  10.10.10.215
Starting Nmap 7.80 ( https://nmap.org ) at 2020-11-17 19:55 EET
Nmap scan report for academy.htb (10.10.10.215)
Host is up (0.37s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Hack The Box Academy
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
#### checking http service

```adding 10.10.10.194 to our /etc/hosts file``` 
first i tried to exploit login page may i find sql injection but i failed ,second i used dirsearch tool to brute force directories.


![Screenshot from 2020-11-17 20-09-37](https://user-images.githubusercontent.com/36403473/99430365-e8b08f00-2911-11eb-883f-21a6fbac1018.png)

I opened the `admin.php` page and found it was a login page
So I created an account and tried to login from `admin.php` but failed.
 this is the request format during registration
![Screenshot from 2020-11-15 22-54-59](https://user-images.githubusercontent.com/36403473/99431286-2eba2280-2913-11eb-956d-d380814c0deb.png)

i tried to change the value of the `roleid` to 1 and login again this time i can login successfully
![Screenshot from 2020-11-14 23-18-50](https://user-images.githubusercontent.com/36403473/99432884-75a91780-2915-11eb-8ea7-f87b4a6ab249.png)
i added this subdomain to ```/etc/hosts``` and opened it  

![Screenshot from 2020-11-17 21-07-39](https://user-images.githubusercontent.com/36403473/99436772-25808400-291a-11eb-870c-d45949af3d30.png)

I noticed that site based on ```laravel``` framework and i have `app_key` so i searched exploits and i found that ```PHP Laravel Framework 5.5.40 / 5.6.x < 5.6.30 - token Unserialize Remote Command```
this module available in `metasploit`

okey lets exploit 
![Screenshot from 2020-11-15 00-23-00](https://user-images.githubusercontent.com/36403473/99437337-fc142800-291a-11eb-8701-3b0b22e9695c.png)
we need to set
``` 
1-app_key "dBLUaMuZz7Iq06XtL/Xnz/90Ejq+DEEynggqubHWFj0="
2-lhost
3-rhost 
4-vhost (virtual host) for subdomain  
5-local machine port 
```
#### 2- user access 
In fact, it was not difficult to obtain the powers of the user.
I just reviewed the env file and found file with data base  and the `cry0l1t3` password.
Just need some time to search.

![Screenshot from 2020-11-15 00-31-20](https://user-images.githubusercontent.com/36403473/99439108-70e86180-291d-11eb-82cc-8fdc6e26fb57.png)
![Screenshot from 2020-11-15 00-32-36](https://user-images.githubusercontent.com/36403473/99439150-7e055080-291d-11eb-9e5a-1ae41c53137d.png)
![Screenshot from 2020-11-15 00-32-47](https://user-images.githubusercontent.com/36403473/99441864-2d8ff200-2921-11eb-9cc1-b50ee0252cc7.png)

#### 3-root access
now i can login with `ssh` 
![Screenshot from 2020-11-15 23-05-46](https://user-images.githubusercontent.com/36403473/99442687-641a3c80-2922-11eb-8cd2-6c8293249d3a.png) 
i upgraded to tty shell `python3 -c 'import pty; pty.spawn("/bin/bash")'` or to upgrade to full tty shell use this commands
```
   /usr/bin/script -qc /bin/bash /dev/null
    Ctrl-Z
    stty raw -echo
    fg
    Ctrl-Z
```
i found that `adm` group,
 To be honest, I did not know at first what it was, but I searched for it and knew the difference between it and the` admin `group


![croped](https://user-images.githubusercontent.com/36403473/99443600-b740bf00-2923-11eb-8951-bf2d80895feb.png)
by adm previlige i could check process happen in the server is checked all logs i had access on them,

so i checked `audit` logs 
its very big logs so i should find way to search logs easly 
``` cat audit.log.3 |grep "uid=1002" ```
![Screenshot from 2020-11-17 22-27-59](https://user-images.githubusercontent.com/36403473/99444798-5d40f900-2925-11eb-911d-064af480324d.png)

this process for server user tried to  use `su` command this data after hexadecimal decode `mrb3n_Ac@d3my!`
I think it's clear it's to  mrb3n 
i could take `mrb3n` previlige 
![Screenshot from 2020-11-16 10-28-45](https://user-images.githubusercontent.com/36403473/99446299-7fd41180-2927-11eb-94dc-4e687ba72ebf.png)
by lookng at user privileges `sudo -l`
![Screenshot from 2020-11-16 10-36-22](https://user-images.githubusercontent.com/36403473/99449819-8e6ef880-2928-11eb-8170-ea9b2448a536.png)
this user can run coomand from composer
## composer 
```
Composer is a tool for dependency management in PHP.
It allows you to declare the libraries your project depends on and it will manage (install/update) them for you.
```
[Composer documentation ](https://getcomposer.org/)


