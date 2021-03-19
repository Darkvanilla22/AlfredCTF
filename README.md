# AlfredCTF
**TryHackMe writeup of the Alfred CTF room** -- _Now featuring better formatting!_

In this room, we will be exploiting a common misconfiguration with Jenkins, a automation tool used for CI/CD (continuous integration/development). We will also be escalating our privileges in order to gain full system access.

To start, let's do a Nmap scan of the machine to probe for any open ports:

![image](https://user-images.githubusercontent.com/53369798/111731050-0d2d6100-8849-11eb-88fd-6f5bb9a2012c.png)

Here's the results to that scan:

![image](https://user-images.githubusercontent.com/53369798/111731172-45cd3a80-8849-11eb-835d-0f8e98974e9d.png)

As you can see, we have three open ports: **80, 3389, and 8080**.

Let's check out the website on port 80:

![image](https://user-images.githubusercontent.com/53369798/111731633-41ede800-884a-11eb-89a0-df254f94291c.png)

At first glance, it doesn't appear to be anything useful here. Let's check the source code next:

![image](https://user-images.githubusercontent.com/53369798/111731731-7366b380-884a-11eb-9ca6-e1071dd7077e.png)

Again, it doesn't look like there is anything hiding from us in plain sight. I'll go ahead and check out the other open web port (8080):

![image](https://user-images.githubusercontent.com/53369798/111732777-b45fc780-884c-11eb-95ac-c4a25dbbdd3f.png)

Looks like a Jenkins login page. Through the power of a quick guess, I tried out credentials "**admin:admin**" to see if their was any default login credentials that have been set. Lo and behold, that was the case!
>Make it a habit to always check for default credentials went probing for initial access! It will save you lots of time and frustration! "admin:password" is also a possible combination to try out too!

Now that we're in, let us figure out what to do from here. A good place to start is finding out where we're able to execute commands to system. Here's a view of the dashboard after logging in:

![image](https://user-images.githubusercontent.com/53369798/111735795-00156f80-8853-11eb-840b-5fb38a7a827b.png)

