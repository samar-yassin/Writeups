## raven writeup

this writeup explains how to **gain root** on raven machine.

I followed the five phases of penetration testing .

This machine is about enumerating the wordpress , and then we will find two usernames this will let us connect via ssh,
then we will find credintials for mysql database . we will find senstive password hashed on wp-users table on mysql that can be break.
after that we can find that we can run python , that will let us gain root.


**you can download it from here https://www.vulnhub.com/entry/raven-1,256/**
