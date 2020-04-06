## How to stream to both facebook live and YT live at the same time

First, a quick note. You will need a Linux distro, or a similar OS such as macOS, to be able to do this. This has been tested many times on WSL (Windows Subsystem for Linux), and for whatever reason (as of April 2020) it will not work. 

Ok, so for the first step you need to sign up for an AWS account [here](https://portal.aws.amazon.com/billing/signup?nc2=h_ct&src=default&redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation#/start). You will need a credit card, but since this tutorial will cover using the free tier, you should not be charged for it.

Second, you need to create an *EC2* instance.
==image==
Select Ubuntu Server 18.05 LTS for your distro.
Click *Next: Configure Instance Details*. You will not need to do anyhting here, so click *Next: Add Storage*.
Again, this should not be messed with, so click *Next: Add Tags*, and click *Next: Configure Security Group*.

You will need to do some configuration on this portion of the setup. Add rules with these parameters:

| Port  | Inbound  | Service |
| ----- | -------  | ------- |
| 80    | Anywhere | HTTP    |
| 443   | Anywhere | HTTPS   |
| 22    | My IP    | SSH     |
| 1935  | My IP    | RTMP    |
| 19350 | My IP    | RTMPS   |

Note that this will allow you to access this server only from your own computer.
Click *Review and Launch* to finish. Make sure that the security group's settings are correct, then click *Launch*.
Select *Create a new key pair* from the drop-down menu. Name it something memorable, download it, and select *Launch Instances*.
Now select *View Instances*. Copy the Puplic DNS.
==image==
Fire up a terminal in the directory in which you have the key pair. Run `sudo chmod 400 <yourkeypairname>.pem`.
Then, run `sudo ssh -i '<yourkeypairname>.pem' ubuntu@<yourpublicdns>`, replacing <yourpublicdns> with the string copied earlier. If all goes well, your terminal should ask you if you want to keep connecting. Type yes and hit enter. If you do not get this message and instead get something like "the host is not available at this time", just give it a few minutes and try again.

Once you are in, you are going to need to install a few things. Run these commands one at a time:

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt install nginx
sudo apt install libnginx-mod-rtmp
sudo apt-get install stunnel4
```

Once you are done, it is time to configure everything.
