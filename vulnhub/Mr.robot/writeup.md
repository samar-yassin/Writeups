# Mr.Robot Writeup

**Download [Mr.robot Machine](https://www.vulnhub.com/entry/mr-robot-1,151/)**

# Tools :
- nmap
- dirb
- hydra

## Find IP address
to find the ip address of the machine , I ran netdiscover

![alt text](https://i.imgur.com/FKeorEl.png)
`netdiscover -r 192.168.1.0/24`
IP : 192.168.53.132

## Port scan
let's run nmap to know which ports are open

![alt text](https://i.imgur.com/R2RRz6K.png)

`nmap -A 192.168.53.132`


okay now we have port 80 and 443

## website enumeration

I will run dirb in the background while browsing the website

I will run dirb in the background while browsing the website

![alt text](https://i.imgur.com/alK7sxV.png)

so I have tried this commands and nothing but videos



I think we should look at robots.txt file

![alt text](https://i.imgur.com/8Bu8b02.png)

ok ! we found the first key and another file called fsocity.dic

![alt text](https://i.imgur.com/10KCoAp.png)

after downloading the fsocity file I found it's a wordlist
let's find out its word count

![alt text](https://i.imgur.com/aVEQPG5.png)

`wc -w fsocity.dic`


858160 word ! maybe there're duplicates ?

I will sort them and remove the duplicates to find out

![alt text](https://i.imgur.com/0oJa0xI.png)

`sort fsocity.dic | uniq > wordlist.txt`


Now we are ready to bruteforce !, but first let's look at dirb result.


![alt text](https://i.imgur.com/wJZrAKT.png)

`dirb  https://192.168.53.132`

we have a wordpress running !
let's look at the login page

![alt text](https://i.imgur.com/mNozAlO.png)

after trying some credentials , I noticed it tells us whether the username is valid or not.
now I guess it's easy to bruteforce the username using hydra
but I thought about trying "elliot" the main character in the mr.robot tv show , and it's right !

![alt text](https://i.imgur.com/vYjrfck.png)

then we can bruteforce the password using hydra

![](https://i.imgur.com/fJrXxgp.png)

`hydra 192.168.53.132 -l elliot -P wordlist.txt http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=log In&testcookie=1:S=Location'`

okay we got the password **ER28-0652**

![](https://i.imgur.com/4Old6lH.png)

it's time to upload a reverse shell
I will go for php reverse shell from [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell)

**don't forget to change the ip and port in the script to your ip and the port you will listen to**

I chose to upload it in the 404.php template from appearance -> editor -> 404 template

![](https://i.imgur.com/KnlFdWD.png)

now we need to start our listener and then visit the 404.php page

![](https://i.imgur.com/a9Z0mMH.png)

`nc -lvp 1234`

now we are deamon
and I found the second flag at home/robot

![](https://i.imgur.com/Bqq02lX.png)

and there's a md5 hash !
I will use [md5 decrypt online tool](https://hashes.com/en/decrypt/hash)

![](https://i.imgur.com/8iwWeDD.png)

let's login as robot

![](https://i.imgur.com/1gQUW0q.png)

## Privilege escalation

let's find the setuid binaries

![](https://i.imgur.com/nBEqa4N.png)

`find / -user root -perm -4000 2>/dev/null`

we can run nmap !
let's find out its version

![](https://i.imgur.com/AhqMtlI.png)

`nmap --version`

with quick search we can find that this version has an interactive mode

![](https://i.imgur.com/3no9GBI.png)

`nmap --interactive`

we can spawn a shell as root !

![](https://i.imgur.com/FxLOqBX.png)

`!sh` to open a shell

finally the last flag!

![](https://i.imgur.com/uRKj91Q.png)




