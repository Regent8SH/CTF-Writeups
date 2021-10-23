Hello,

My name is Regent8SH, and this is my first CTF writeup. Currently I am Security+ and A+ certified. I have completed the INE Student training in preparation of the eJPT, and am looking forward to completing the eJPT exam so I can begin working on the OSCP. I hope to continue documenting these as a way of showcasing my Cyber Security journey.

I absolutely love the TryHackMe platform, and I highly recommend you try it out at https://tryhackme.com

First things first, I always start by adding the IP to my /etc/hosts file (I will be using ignite.thm), and follow it up with an initial nMap scan.

nMap found the following result on port 80:

![image](https://user-images.githubusercontent.com/92694455/138529055-393227d7-6dfb-41b5-b90b-0667cd39b2f3.png)

Navigating to http://ignite.thm in our browser, we find a FUEL CMS Welcome page:

![image](https://user-images.githubusercontent.com/92694455/138529164-593867bb-8e56-4830-ab48-2a6ab6beca24.png)

It says Version 1.4, and a quick google search finds that there is probably a remote code execution vulnerability:

![image](https://user-images.githubusercontent.com/92694455/138529863-5493527b-d5cb-47cb-a92d-06f17bfd5a55.png)

I'm going to start a gobuster in the background to be safe. (I always like to add php and txt file extensions in CTFs just in case):

![image](https://user-images.githubusercontent.com/92694455/138529333-d2b17141-3b79-46bd-b32b-91c67df400f8.png)

Now, I looked over both of the RCE exploit scripts on exploit-db and I'm going to start with the python script because I have more experience with Python.

I download the file to "fuelexploit.py", change the IP in the code to "ignite.thm", and run:

![image](https://user-images.githubusercontent.com/92694455/138531570-63f23637-5c83-4d85-9f17-2fec262ef1fd.png)

Immediately I notice the "burp0_url" and "proxy".. Oops... turns out Burpsuite needs to be on.. lol. Let's boot that up and try again:

![image](https://user-images.githubusercontent.com/92694455/138531619-85486b76-6c90-413e-bfb2-45e677de45ea.png)

Okay.. I cropped the response because it appears to print the full source of the reponse page, but the first line of the response tells me that the RCE is working. Lets try to get a reverse shell.

I pop a listener on my favorite port 8123, copy my favorite "mkfifo" reverse shell from the PenTestMonkey reverse shell cheat sheet, and pop it into our exploit (cropped to hide my tun IP):

![image](https://user-images.githubusercontent.com/92694455/138531819-ab681792-2e25-4cf6-b744-2120205c6ea5.png)

Let's check out our listener:

![image](https://user-images.githubusercontent.com/92694455/138531851-c49467db-1bb0-4544-b9db-bda470d67836.png)

Neat! We've got our reverse shell. I will check which python version (both), grab my shell stabilization notes, and get a better shell:

![image](https://user-images.githubusercontent.com/92694455/138533739-1086ca17-a62d-4ff8-bc16-309cd3b0709e.png)

![image](https://user-images.githubusercontent.com/92694455/138532045-14166d6e-32d7-4bd3-a9c3-c6cfce1eba4a.png)

Alright, we're good to go. Let's navigate to the /home/ directory to check the users and grab our first flag:

![image](https://user-images.githubusercontent.com/92694455/138532120-2b567a90-2cf5-4bf4-a5b8-27e1291b1d10.png)

I won't show the flag, because you should definitely try for it yourself! But let's figure out this Privelege Escalation.

We cd over to the /tmp/ directory, then create a python web server on our /opt/ directory so we can wget our tool linpeas.sh.

Let's start that up now!:

![image](https://user-images.githubusercontent.com/92694455/138532294-ce660daf-22b0-4ebb-bbf1-7f17869fd22b.png)

TBH I'm not seeing a lot. I notice a pkexec vulnerability suggestion, and we appear to be in the right Linux Kernel Version range, so I download the exploit and give it a go.. Nope it doesn't work, but it says it's being reported to the administrator. Lol.

Wait! Linpeas noticed this in it's enumeration:

![image](https://user-images.githubusercontent.com/92694455/138534302-da10c11b-7ab2-444b-baf8-cf502c7d2e47.png)

Looks like there may be some creds in these PHP files. Let's go through them.
Lucky for us, we check the very first file listed here and there is a DB creds list that has root and password:

![image](https://user-images.githubusercontent.com/92694455/138533345-a60b8e25-dfc0-4bd7-93ff-21e7af620939.png)

![image](https://user-images.githubusercontent.com/92694455/138534355-5647d4ca-1898-4f3a-aa2d-ee3a57153802.png)

So our creds look like root:m*****
Let's try to su to root?

![image](https://user-images.githubusercontent.com/92694455/138533399-8239b877-0d2f-49ba-b462-0a9041505c2a.png)

Wow that was faster than I thought it would be. It did show that you should read over all of the results of your Linpeas, not jsut the guaranteed privilege escalations. I'm gonna cat the flag real quick to finish the room out:

![image](https://user-images.githubusercontent.com/92694455/138533473-b224e60e-ab22-4cc8-b01e-b154645d2802.png)

Pretty fun box! Very basic, but it did a good job of making me actually read the exploit script being used and making me dig deeper into my PrivEsc enumeration script. 

Note: I had to go back a recapture a screenshot of the linpeas results, and when I did I notice that the password was caught as well. The simplicity of the format made me not notice, but I should have caught this as well! Here's a POC (as usual, edited to remove the root password):

![image](https://user-images.githubusercontent.com/92694455/138534185-997422c2-d8b0-4bb6-9963-cfa95db36462.png)

Again, I highly recommend visiting https://tryhackme.com 
