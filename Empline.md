This is my second CTF write-up on Github. I am now eJPT, Security+, and IBM Cybersecurity Analyst certified. I have been working as an Offensive Security Consultant for about 7 months. I am planning to take the OSCP exam by the end of September, which is why I am practicing this documentation.

Just like before, I love the TryHackMe platform, and I highly recommend you try it out at https://tryhackme.com

As usual, I start with adding the IP to my /etc/hosts file (Let's use empline.thm), and then kick off our nmap scan.

![image](https://user-images.githubusercontent.com/92694455/178883091-d3b5815b-c17a-4c60-a05d-cefad9ded81b.png)

We begin to browse the webserver on port 80 while we kick off a gobuster to look for additional subdomains, almost imediately finding "job.empline.thm". Let's add that to our /etc/hosts and browse to the page.

![image](https://user-images.githubusercontent.com/92694455/178885563-46136d66-b9fa-4005-b983-b0b069bc57eb.png)

We notice on the opencats job page that it says that it is running "Version 0.9.4 Countach". A quick google of exploits for that version find an exploit script on Github. We can go ahead and download that script.

![image](https://user-images.githubusercontent.com/92694455/178885874-132107e5-bcae-44d3-8a52-eee61613eb10.png)

Running this script on the vulnerable Opencats immediately pops a basic Linux shell prompt. We pop open a local listener and attempt to run a reverse shell, however it seems unsuccessful from this weakened shell. Let's write a script to copy over and execute a reverse shell command locally.

![image](https://user-images.githubusercontent.com/92694455/178886618-0989fdab-d532-42ec-852a-e9a57088aff1.png)

Now we can copy this script over to the vulnerable machine using a simple Python HTTP Server.

![image](https://user-images.githubusercontent.com/92694455/178886929-95373ba6-661a-4dff-a7e3-6d925fde129d.png)

Executing the command, our listener finally catches a shell. We stabilize the session using Python3.

![image](https://user-images.githubusercontent.com/92694455/178887523-8bbe4bbc-c648-4d55-988d-c58c901e4563.png)

My favorite thing to do in Linux when geting a foothold on a webserver with the www-data user is to check the webroot folders in the /var/www/ directories. In this case, after viewing config PHP files we find some DB credentials. Here I display with a grep for "DATABASE" to make the screenshot easier to read in this write-up.

![image](https://user-images.githubusercontent.com/92694455/178889884-d5c7f60b-5c43-4c7e-991d-8ef40a348fdf.png)

We log into the local MySQL database with the found credentials, and begin enumerating tables from the database. 

![image](https://user-images.githubusercontent.com/92694455/178890271-0e4b3cff-7657-4801-9c1f-ac5d3be40fa0.png)

Looking into the "user" table, we find the MD5 hashes of all of the users' passwords. The "george" user is notable, as this user was confirmed to exist in /etc/passwd

![image](https://user-images.githubusercontent.com/92694455/178890656-595ed063-de1c-4338-9582-8bf297a96c0f.png)

Hashcat quickly cracks the hash for the george user, which I deleted from this screenshot because I guess you should at least have to do something for yourself.

![image](https://user-images.githubusercontent.com/92694455/178891406-cec4c254-a1bc-4b10-b68a-e7b490938712.png)

Using the cracked password, we log into using SSH to the box with the "george" user.

![image](https://user-images.githubusercontent.com/92694455/178891526-980ab472-afb0-4065-a0f8-5a3420e73a9c.png)

Running through our immediate checklist when lookiing for Privilege Escalation, we check things like Sudo, SUID, and Capabilities. Immediately, a capability stands out. Ruby appears to have chown privileges.

![image](https://user-images.githubusercontent.com/92694455/178891904-9d74ec06-8cb5-4c7b-9fbd-1cdd7b026b71.png)

Checking HackTricks, we find a guide explaining ways of exploiting misconfiguration of Capabilities. There even appears to be an example showcasing Ruby.

![image](https://user-images.githubusercontent.com/92694455/178892218-7e42156d-f350-474c-b525-a392a5b6c967.png)

Viewing /etc/passwd, we can see the available users and their user IDs. We can abuse the payload found on HackTricks to change the ownership of the /etc/shadow and /etc/passwd files to our "george" user.

![image](https://user-images.githubusercontent.com/92694455/178894182-92170ee4-eefe-43ad-b747-1205ee9cf1f7.png)

Owning the /etc/shadow and /etc/passwd files, I copy the hashed password value of "george" (since I already have the plaintext equivilant) and create a new user name "Regent8SH" with root privileges. Switching users to this newly created account immediately gives me root, completing the room!

![image](https://user-images.githubusercontent.com/92694455/178893694-962e2e00-0470-4d89-9f91-d51dbda182d6.png)
