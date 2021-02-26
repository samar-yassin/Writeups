# Help! Where is Santa?

after deploying the machine, I got the IP address of the machine is 10.10.83.213


**What is the port number for the web server?**

to answer this question we can do nmap scan to find out

![](https://i.imgur.com/dMal2F6.png?1)

`nmap 10.10.83.213`

so the web server is running on port 80

Answer --> 80

**Without using enumerations tools such as Dirbuster, what is the directory for the API?  (without the API key)**

we can't use enumeration tools , so let's look at the website maybe we can find something

let's check the source code

![](https://i.imgur.com/mUvt2k7.png?1)

we can see here at line 88 , the link for the api

Answer --> /api/



**Find out the correct API key. Remember, this is an odd number between 0-100. After too many attempts, Santa's Sled will block you. 
To unblock yourself, simply terminate and re-deploy the target instance (10.10.83.213)**

this is the last question but I will answer it first as we don't have a clue for the third one.

so again can't do a bruteForce .. but we have an information says that the api-key is between 1-100 and it's odd number .
so we can build a customized tool to find out the api-key

```
#!/usr/bin/env python


#!/usr/bin/env python


import requests


for i in range(1,100,2) :
	url = "http://10.10.44.126/api/%s" %i
	response = requests.get(url)
	content = response.text
	print("key = ",i,"  content ->",url)




```

here I started with a loop that iterates from  0 to 100 with step=2 , so it count like 1,3,5,..
then inside the loop I assigned the url to bruteforce and %i to change the value of the api-key every iteration
then request the page and print out the respone
![](https://i.imgur.com/XpyJZvI.png?1)

api-key=57 is the right api-key and we can see in its response there's an address , we can use it as santa's address

Answer --> 57

**Where is Santa right now?**

Answer --> Winter Wonderland, Hyde Park, London
