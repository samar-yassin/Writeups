# raven writeup


This machine is about enumerating the wordpress , and then we will find two usernames this will let us connect via ssh,
then we will find credintials for mysql database . we will find senstive password hashed on wp-users table on mysql that can be break.
after that we can find that we can run python , that will let us gain root.


**you can download it from here https://www.vulnhub.com/entry/raven-1,256/**

## Find  IP adress
first step is to get the IP of the machine , so we can run netdiscover or nmap.
here I ran netdicover.

![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/netdiscover.png?raw=true)
```
command : netdiscover -r 192.168.1.0/24
```
know we know our target IP is 192.168.53.130


## Port scan
now we need to do nmap scan to know what services are running.
![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/nmap-A.png?raw=true)
great ! , here we have port 22 open and that means if we can find a username we can bruteforce the password.

we also have port 80 open, we need to have look at it.
but before that I will run dirb on the background to find any interesting paths.
![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/website.png?raw=true)



I ran burbsuite while browsing the website.
![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/burbsuite.png?raw=true)



after analyzing response pages , I found first flag on service.html pages.
flag1{b9bbcb33e11b80be759c4e844862482d}

here's dirb result .

![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/dirb.png?raw=true)

```
command : dirb http://192.168.53.130
```


and there's something interesting in dirb and burbsuite result , we have a wordpress running!
let's run wpscan , so we can obtain usernames or find vulnerable plugins.

![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/wpscan.png?raw=true)
![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/wpscan2.png?raw=true)

Good! , we found two usernames !
now we need to use hydra to do the bruteforce.
I will use rockyou wordlist .

![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/hydra.png?raw=true)
```
command : hydra -l michael -P rockyou.txt ssh://192.168.53.130
```

now let's login via ssh .
![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/michaelssh.png?raw=true)


after navigating through folders I found the second flag on /var/www
flag2{fc3fd58dcdad9ab23faca6e9a36e581c}                                                                                                           

let's take a look on html directory

![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/config.png?raw=true)

here we found some html and php files and wordpress folder , let's take a look.
let's look at the configuration file(wp-config.php) if there's sensitve information.

![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/db.png?raw=true)


okay ! those are user and password for mysql database named wordpress!
let's look at them.

![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/wpusers.png?raw=true)


here we found the password hashes for the two users , let's crack them using john the ripper.
before that i found two flags on wp_posts!

![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/flag3&4.png?raw=true)

flag3{afc01ab56b50591e7dccf93122770cd2}
flag4{715dea6c055b9fe3337544932f2941ce}

now back to john ! , we will save the hash in a file i named it hashsteven.

![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/john.png?raw=true)

##  Privilege Escalation
now we have steven password let's login vis ssh.

![alt text](https://github.com/samar-yassin/writeups/blob/main/vulnhub/raven1/photos/stevenssh.png?raw=true)


here I need to know which command steven can run in sudo .
```
command : sudo -l
```

ok! we can use python.
```
command :  sudo /usr/bin/python -c "import pty;pty.spawn('/bin/bash');"
```

now we are root, and we got the fourth flag !

flag4{715dea6c055b9fe3337544932f2941ce}
