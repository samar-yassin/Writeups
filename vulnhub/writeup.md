Find the IP adress
first step is to get the IP of the machine , so we can run netdiscover or nmap.
here I ran netdicover.


command : netdiscover -r 192.168.1.0/24

know we know our target IP is 192.168.53.130

Port scan
now we need to do nmap scan to know what services are running.

great ! , here we have port 22 open and that means if we can find a username we can bruteforce the password.

we also have port 80 open, we need to have look at it.
but before that I will run dirb on the background to find any interesting paths.



I ran burbsuite while browsing the website.



after analyzing response pages , I found first flag on service.html pages.
flag1{b9bbcb33e11b80be759c4e844862482d}

here's dirb result .

command : dirb http://192.168.53.130


and there's something interesting in dirb and burbsuite result , we have a wordpress running!
let's run wpscan , so we can obtain usernames or find vulnerable plugins.
   
   Good! , we found two usernames !
   now we need to use hydra to do the bruteforce.
   I will use rockyou wordlist .
   command : hydra -l michael -P rockyou.txt ssh://192.168.53.130
   
   
   now let's login via ssh .
   
   
   after navigating through folders I found the second flag on var/www
   flag2{fc3fd58dcdad9ab23faca6e9a36e581c}                                                                                                           
   
   let's take a look on html folder
   
   here we foung some html and php files and wordpress folder , let's take a look.
   let's look at the configuration file if there's sensitve information.
   
   
   
   okay ! those are user and password for mysql database named wordpress!
   let's look at them.
   
   
   
   here we found the password hashes for the two users , let's crack them using john the ripper.
   before that i found two flags on wp_posts!
   
   
   flag3{afc01ab56b50591e7dccf93122770cd2}
   flag4{715dea6c055b9fe3337544932f2941ce}
   
   now back to john ! , we will save the hash in a file i named it hashsteven.
   
   
   
   now we have steven password let's login vis ssh.
   
   Privilege Escalation
   
   
   
   here I need to know which command steven can run in sudo .
   command : sudo -l
   ok! we can use python.
   
   command :  sudo /usr/bin/python -c "import pty;pty.spawn('/bin/bash');"
   
   now we are root, and we got the fourth flag !
   
   flag4{715dea6c055b9fe3337544932f2941ce}
