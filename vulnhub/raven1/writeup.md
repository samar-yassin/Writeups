# raven writeup


This machine is about enumerating the wordpress , and then we will find two usernames this will let us connect via ssh,
then we will find credintials for mysql database . we will find senstive password hashed on wp-users table on mysql that can be break.
after that we can find that we can run python , that will let us gain root.


**you can download it from here https://www.vulnhub.com/entry/raven-1,256/**

## Find  IP adress
first step is to get the IP of the machine , so we can run netdiscover or nmap.
here I ran netdicover.

![alt text](https://i.imgur.com/AK3QbZL.png)
`command : netdiscover -r 192.168.1.0/24`

know we know our target IP is 192.168.53.130


## Port scan
now we need to do nmap scan to know what services are running.
![alt text](https://i.imgur.com/qrTz6fr.png)
great ! , here we have port 22 open and that means if we can find a username we can bruteforce the password.

we also have port 80 open, we need to have look at it.
but before that I will run dirb on the background to find any interesting paths.
![alt text](https://i.imgur.com/qynGcaJ.png)



I ran burpsuite while browsing the website.
![alt text](https://i.imgur.com/EQFc0Aa.png)



after analyzing response pages , I found first flag on service.html pages.
flag1{b9bbcb33e11b80be759c4e844862482d}

here's dirb result .

![alt text](https://i.imgur.com/18kPvHg.png)

`command : dirb http://192.168.53.130`


and there's something interesting in dirb and burpsuite result , we have a wordpress running!
let's run wpscan , so we can obtain usernames or find vulnerable plugins.

![alt text](https://i.imgur.com/OFb6qOg.png)
![alt text](https://i.imgur.com/0aOtUh8.png)

Good! , we found two usernames !
now we need to use hydra to do the bruteforce.
I will use rockyou wordlist .

![alt text](https://i.imgur.com/6TCVKN5.png)

`command : hydra -l michael -P rockyou.txt ssh://192.168.53.130`

now let's login via ssh .
![alt text](https://i.imgur.com/xeoa8xv.png)


after navigating through folders I found the second flag on /var/www
flag2{fc3fd58dcdad9ab23faca6e9a36e581c}                                                                                                           

let's take a look on html directory

![alt text](https://i.imgur.com/2gdsflE.png)

here we found some html and php files and wordpress folder , let's take a look.
let's look at the configuration file(wp-config.php) if there's sensitve information.

![alt text](https://i.imgur.com/TOmkHYA.png)

okay ! those are user and password for mysql database named wordpress!
let's look at them.

![alt text](https://i.imgur.com/J4syuEy.png)


here we found the password hashes for the two users , let's crack them using john the ripper.
before that i found two flags on wp_posts!

![alt text](https://i.imgur.com/DxDsWOc.png)

flag3{afc01ab56b50591e7dccf93122770cd2}
flag4{715dea6c055b9fe3337544932f2941ce}

now back to john ! , we will save the hash in a file i named it hashsteven.

![alt text](https://i.imgur.com/9b4NhST.png)

##  Privilege Escalation
now we have steven password let's login vis ssh.

![alt text](https://i.imgur.com/Hedd09j.png)


here I need to know which command steven can run in sudo .

![alt text](https://i.imgur.com/oQFTNyK.png)

`command : sudo -l`

ok! we can use python.

![alt text](https://i.imgur.com/fPM2Eeu.png)

`command :  sudo /usr/bin/python -c "import pty;pty.spawn('/bin/bash');"`

now we are root, and we got the fourth flag !

flag4{715dea6c055b9fe3337544932f2941ce}
