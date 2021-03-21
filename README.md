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

Now that we're in, let us figure out what to do from here. A good place to start is finding out where we're able to execute commands to the system. Here's a view of the dashboard after logging in:

![image](https://user-images.githubusercontent.com/53369798/111735795-00156f80-8853-11eb-840b-5fb38a7a827b.png)

Let's go ahead and see what's up wtih the job called "project". After looking around for a bit, I did come across this console output from build #1:

![image](https://user-images.githubusercontent.com/53369798/111917293-9913d880-8a55-11eb-8cbe-13a100f7783c.png)

This may be useful, as in the moment of typing this, I'm not all too certain. Part of doing Pentesting is noting down anything that **can** be of value. Anyways, by going to "Configure", I was able to discover how the build was able to execute that command:

![image](https://user-images.githubusercontent.com/53369798/111917412-425ace80-8a56-11eb-8379-dca53a45d79a.png)

As this appears to be a Windows enviornment, I could utilize Powershell in order to gain inital access into the system via reverse shell. To do this, I will be using this reverse shell from Nishang:
>This is just a snippet of the full code

![image](https://user-images.githubusercontent.com/53369798/111918009-5bb14a00-8a59-11eb-99bf-ead955fa520e.png)

In order for us to get this script onto the Jenkins server, we'll need to download it from our attacking machine. WE can do this by starting up a web server using Python:

![image](https://user-images.githubusercontent.com/53369798/111918397-45a48900-8a5b-11eb-8108-a034e58ad716.png)

![image](https://user-images.githubusercontent.com/53369798/111918404-505f1e00-8a5b-11eb-89cc-c3eb850ad1ce.png)

With our web server up and running, all we need to do is to have Jenkins download the script and execute it:

![image](https://user-images.githubusercontent.com/53369798/111918464-8bf9e800-8a5b-11eb-9208-a7372b1878cc.png)

Now that's set up, we need to make sure we have a listener up and running as well:

![image](https://user-images.githubusercontent.com/53369798/111918494-adf36a80-8a5b-11eb-9992-fdea76e60d8d.png)

And now for the moment of truth. To get the command running, we need to start a new build. A few unsuccessful attempts and a little bit of testing later, we've managed to get everything working in our favor:

![image](https://user-images.githubusercontent.com/53369798/111918533-e1ce9000-8a5b-11eb-91dc-73fc56d788b3.png)
>The red bar indicates that the build is still in the process of execute the full command, however, just as it is normally with reverse shell connections, the command will fully executes when the reverse shell closes

![image](https://user-images.githubusercontent.com/53369798/111918585-222e0e00-8a5c-11eb-9c30-c516b8ac389e.png)

And now we have access into the system! By doing a bit of navigating through the system, I was able to discover the "user.txt" file on Bruce's desktop:

![image](https://user-images.githubusercontent.com/53369798/111918654-83ee7800-8a5c-11eb-8f41-4fd9b92b59ae.png)

To make this easier on escalating our privileges, we can actually switch from using our first reverse shell into that of a meterpreter shell. Let's start off by first generating the payload that we'll be using:

![image](https://user-images.githubusercontent.com/53369798/111919169-f6f8ee00-8a5e-11eb-81ef-1de34cf7f75a.png)

And now, time to download our payload onto the Jenkins system:

![image](https://user-images.githubusercontent.com/53369798/111919453-6c18f300-8a60-11eb-84f8-a9bd8fa573cb.png)

Here's the console log from Jenkins confirming that the payload has been downloaded successfully:

![image](https://user-images.githubusercontent.com/53369798/111919480-97034700-8a60-11eb-8332-27dc958312be.png)

Now, we'll start a listener in Metasploit to await for an incoming connection request:

![image](https://user-images.githubusercontent.com/53369798/111919546-fcefce80-8a60-11eb-9a79-cdb1c4aa3976.png)

And now to start the payload:

![image](https://user-images.githubusercontent.com/53369798/111921716-7640ee80-8a6c-11eb-9757-c9b1a877df0b.png)
>The reason as to why I'm no longer in Bruce's directoy is simple: my previous session timed out and I just started up a new session

And now we have a meterpreter session!:

![image](https://user-images.githubusercontent.com/53369798/111921752-b86a3000-8a6c-11eb-87cf-73678d9a3126.png)
>I had some diffculty getting a stable session, but eventually it worked out. I think there was something wrong with the machine itself, but that's ok

Now that we're in, lets check out what privileges we have:

![image](https://user-images.githubusercontent.com/53369798/111921807-febf8f00-8a6c-11eb-8abf-ad97331b836d.png)
>"SeDebugPrivilege" and "SeImpersonatePrivilege" are both enabled, those will prove to be useful

As we can see, we've got some privileges that are beneficial to us in order to impersonate our authentication tokens. To further exploit these, we can utilize the "incognito" module in Metasploit as follows:

![image](https://user-images.githubusercontent.com/53369798/111921925-96bd7880-8a6d-11eb-8bea-b38155b52fce.png)

Now that we have an admin token impersonated, we can use this to leverage our way into a root process (such as _services.exe_). To do this, we can use the "migrate" command in meterpreter to do just that:

![image](https://user-images.githubusercontent.com/53369798/111921997-f74cb580-8a6d-11eb-8b1f-ee615d0a0f79.png)
>In this system, the PID (Process ID) for "_services.exe_" is 668

Now that we have privileges equivalent to that of a root user, we can now obtain and read the root.txt file waiting for us:

![image](https://user-images.githubusercontent.com/53369798/111922113-7e9a2900-8a6e-11eb-8bfb-d32fa61cebb0.png)

And that's the end of this room! I really did enjoy this CTF!
