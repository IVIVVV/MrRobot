# MrRobot
This is my first "project", which include the ways that i made to do a simple pentest to a webserver from "vulnhub.com".
If anyone have a better way of doing things please contact me thanks in advance

Link to that machine: https://www.vulnhub.com/entry/mr-robot-1,151/


- Let's start by running an nmap using the command: nmap -sS -T4 <IP/url range>. In my case the ip range is 10.1.1.10-100 and the target's ip is 10.1.1.11
Hmmm port 23,80,443 are open.... looks like we got ourself a website, lets check it using a webroswer and... nothing seems to be helpful

- We now know that its a website lets continue by doing a fuzz in my case im going to use nikto: nikto -h <10.1.1.11> or you can just use nmap -sV --script=http-enum <10.1.1.11>. 
+ The result returned some links to wordpress login pages, we will check that out later. BUT we can now have some hope for reverse shell!
+ robots.txt looks juicy lets go there first! and as expected we got our first flag
not only that we also got our hands on the "fsocity.dic" access and download it, seems to be some kind of username/password list AND YOU KNOW WHAT THAT MEANS!
thats right brute force time
Before trying brute force let's make the job simpler for our computer shall we? Lets run the cat sorting code: cat <fsociety.dic> | uniq >> <your new file> 
basically what this does is to remove all repeated lines

- Now since we dont know what the username is, we cant just brute force both username and password at the same time cause we dont have all days and we are all busy people
+ There are lots of tool out there to find the username but im just going to use hydra cause im a noob:
+ hydra -vV -L <username list> -p <random password> <ip> http-post-form '/<path to login>:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username'
+ username list: the list that you filtered earlier 
+ random password: lets just put something like "ihatemylife" since we dont need the real password just yet
+ path to login: we all know what this is
+ log=^USER^&pwd=^PASS^&wp-submit=Log+In : ^USER^ and ^PASS^ are placeholders that wiil be replaced with the actual values.
+ That was me being extra, since we know this is a wordpress login page we can just use the wpscan: wpscan --url <path> -enumerate u 
+ And with that we got 2 username and lets try our luck by brute forcing 1 by 1 
+ After i leave the computer running by itself i now have the account Elliot:ER28-0652 now all i have to do is just to login and see something i can mess with

- By "something i can mess with" i meant REVERSE SHELL 
+ You can just go to the revershell code i put in my project and change some stuff like ip and port, pop that in something thing like the 404.php save it and we are done for now
+ Go ahead and type in 10.1.1.11/ihatemyself which is some random path and return the 404.php oh i forgot to mention, you should do: "nc -lvnp port" before searching for the
path, this will setup a listener on that port
+ type in id and we seem to be login as daemon lets check for some directories. Start off with home list that we,look at what we have here, robot. Cd into that and bingo we go the 2nd key
but i got access denied hmmm... oh wait the password is right there in the password.raw-md5 cat into that we got the username and password but that password looks weird
and the file name is md5 so lets just try and do a hashcat and the password is: abcdefghijklmnopqrstuvwxyz 
+ Change directory to the user daemon:
python -c 'import pty;pty.spawn("/bin/bash")'
and access that user "robot": su robot. open the txt folder and we got ourself the 2nd key

- After finding the second key i got stuck and have no idea where else to look for, after some research(days after) i learned that i can execute shell from nmap
+ from there i remembered that there was a directory with nmap so i did what i have to do: nmap --interactive and that actually works following the guides in this site
https://gtfobins.github.io/gtfobins/nmap/
+ the command: 
nmap --interactive
nmap> !sh
and now we are root :D 

- List everything within root directory and we see the last key and GG! I CAN FINALLY SLEEP NOW  
