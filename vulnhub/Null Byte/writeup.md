# NULL BYTE

# TOOLS
- nmap
- hydra
- sqlmap
- exiftool
- md5 decryption
- base64 decoder


# Find IP Address
I will run netdiscover.

![](https://i.imgur.com/vnJ6so6.png)

`netdiscover -r 192.168.1.0/24`

# Port scan
running nmap

![](https://i.imgur.com/pT5BAsY.png)

`nmap -A 192.168.53.135`
so here we have web server on port 80 and ssh service running on port 777

let's look at the website

![](https://i.imgur.com/805Mu33.png)

okay there's nothing here and no robots.txt
let's look at dirb

![](https://i.imgur.com/cEerdlP.png)

`dirb 192.168.53.135`
directory uploads ! ,but nothing interesing in it ..
and phpmyadmin looks interesing . tried to log at it as root root and was wrong so let's dig more

so nothing here but this photo? , let's examine it!
downloaded it and I will run exiftool on it

![](https://i.imgur.com/WMhF0kp.png)

`exiftool main.gif`
and there's comment there! .. it doesn't look like a username , I think i will use it as a directory

![](https://i.imgur.com/U1lkogZ.png)

and yes it is!, so it tells us it needs key. let's look at the source code

![](https://i.imgur.com/UYmtsrl.png)

so maybe we can bruteforce it using hydra.

![](https://i.imgur.com/RBrAce4.png)

and we got a hit

entering it will get us to another page for searching usernames

![](https://i.imgur.com/cX9CbJO.png)

I typed nothing and get us all the result

![](https://i.imgur.com/zi7W9Kz.png)

and I tried injecting a double quote , it tells us it's mysql

![](https://i.imgur.com/cGoH9Xk.png)

let's use sqlmap !

![](https://i.imgur.com/cuTJj3U.png)

`sqlmap http://192.168.53.135/kzMb5nVYJw/420search.php?usrtosearch= --dbs`

and here's the result

![](https://i.imgur.com/EEshmrO.png)

okay let's enumerate mysql tables

![](https://i.imgur.com/LsTjb0w.png)

`sqlmap http://192.168.53.135/kzMb5nVYJw/420search.php?usrtosearch= -D mysql -tables`

let's dump table user

![](https://i.imgur.com/KZMd4Em.png)

`sqlmap http://192.168.53.135/kzMb5nVYJw/420search.php?usrtosearch= -D mysql -T user -dump`

so i think the root username and sunnyvale password is for phpmyadmin so let's try
and yes now we have acsess to phpmyadmin !
we could just dumb the tables for phpmyadmin from sqlmap but since we got here let's continue...

looking at the tables I found this..

![](https://i.imgur.com/ISMMGuR.png)

so after trying to crack it with john and couldn't crack it .. i tried to decode it as base64 using [base64 decoder](https://www.base64decode.org/)

![](https://i.imgur.com/Ce42jhJ.png)

so it gave us another hash ... it looks like md5 so let's try it [md5 decryption](https://www.md5online.org/md5-decrypt.html)

![](https://i.imgur.com/zEj8oLA.png)

okay we have another password , sure thing it's for ssh , DON'T FORGET it's running on port 777.

![](https://i.imgur.com/rN2bH1x.png)

`ssh ramses@192.168.53.135 -p 777`

# Privilege escalation

after looking around I found a procwatch file and readme file at /var/www/backup

looking at readme

![](https://i.imgur.com/WKLyJCV.png)

let's find out what is this procwatch file

![](https://i.imgur.com/qeq0dNI.png)

okay that means we are on the right path , let's run the executable file

![](https://i.imgur.com/uup29RS.png)

here it just run ps !
so if we manage to make this file thinks that "ps" command is "sh" , running procwatch will give us root shell

looking at those articles [path variable](http://www.linfo.org/path_env_var.html#:~:text=PATH%20is%20an%20environmental%20variable,commands%20issued%20by%20a%20user.)  ,  [exploit suid](https://www.pentestpartners.com/security-blog/exploiting-suid-executables/) will help alot.

so we basiclly will copy "/bin/sh" to somewhere and then append it to the path variable.
so when we run the file first thing it will run ps and $PATH will search for ps at /tmp/ where we coppied "sh" as ps there

let's do it and get a copy of sh command

![](https://i.imgur.com/tVkWfqT.png)

printing out PATH variable so we can know what to modify and then modifying it

![](https://i.imgur.com/q5YeqBi.png)

so here we ran procwatch and we're root now!

![](https://i.imgur.com/KfWTJPk.png)

let's get our proof that we were here

![](https://i.imgur.com/wnFv1eF.png)


# References

- [path variable](http://www.linfo.org/path_env_var.html#:~:text=PATH%20is%20an%20environmental%20variable,commands%20issued%20by%20a%20user.)

- [exploit suid](https://www.pentestpartners.com/security-blog/exploiting-suid-executables/)


