# Santa's watching 

IP address of the machine is 10.10.169.169

**Given the URL "http://shibes.xyz/api.php", what would the entire wfuzz command look like to query the "breed" parameter using the wordlist "big.txt" (assume that "big.txt" is in your current directory)**

Answer --> wfuzz -c -z file,big.txt http://shibes.xyz/api.php?breed=FUZZ

**Use GoBuster (against the target you deployed -- not the shibes.xyz domain) to find the API directory. What file is there?**
![](https://i.imgur.com/zpcfEr3.png?1)

`dirb http://10.10.169.169`

after visiting this directory we can find a file named site-log.php
![](https://i.imgur.com/s3t5Dr0.png?1)

Answer --> site-log.php

**Fuzz the date parameter on the file you found in the API directory. What is the flag displayed in the correct post?**

![](https://i.imgur.com/JmEQbAj.png?1)

`wfuzz -c -z file,wordlist -u http://10.10.169.169/api/site-log.php?date=FUZZ`

we can see there's one result that has a characters in it.
after opening this file we can find the flag "THM{D4t3_AP1}"

Answer --> THM{D4t3_AP1}
