## How to stream to both Facebook live and YouTube live at the same time

Do you stream through facebook or youtube? Do you wish you could stream to both at the same time without paying hundreds of dollars for expensive software? Well, this guide will show you how!

First, a quick note. You will need a Linux distro, or a similar OS such as macOS, to be able to do this. This has been tested a few times on WSL (Windows Subsystem for Linux), and for whatever reason (as of April 2020) it will not work. 

Ok, so for the first step you need to sign up for an AWS account [here](https://portal.aws.amazon.com/billing/signup?nc2=h_ct&src=default&redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation#/start). You will need a credit card, but since this tutorial will cover using the free tier, you should not be charged for it.

Second, you need to create an *EC2* instance.

![select ec2](img/ec2.jpg.jpg)

Select Ubuntu Server 18.05 LTS for your distro.

![select ubuntu](img/ubuntu.jpg.jpg)

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

![public dns](img/publicdns_LI.jpg)

Fire up a terminal in the directory in which you have the key pair. Run `sudo chmod 400 <yourkeypairname>.pem`.
Then, run `sudo ssh -i '<yourkeypairname>.pem' ubuntu@<yourpublicdns>`, replacing <yourpublicdns> with the string copied earlier. If all goes well, your terminal should ask you if you want to keep connecting. Type yes and hit enter. If you do not get this message and instead get something like "the host is not available at this time", just give it a few minutes and try again.

Once you are in, you are going to need to install a few things. Run these commands one at a time:

```sh
sudo apt-get update
sudo apt-get upgrade
sudo apt install nginx
sudo apt install libnginx-mod-rtmp
sudo apt-get install stunnel4
```

Once you are done, it is time to configure everything. First, sign into Facebook and start a new live stream. Check *Use persistent key*, and copy the key. Now, run `sudo nano /etc/nginx/nginx.conf`. In this file, append 
```sh
rtmp {
        server {
                listen 1935;
                chunk_size 4096;

                application live {
                        live on;
                        record off;
		                    push rtmp://a.rtmp.youtube.com/live2/<your youtube stream key>;
			                  push rtmp://127.0.0.1:19350/rtmp/<your persistent stream key>;
                }
        }
}
```
to after the http arguments, pasting the facebook stream key in it's spot. In all, your .conf file should look something like this:
```sh
# Important notes

Facebook is enforcing rtmps, so additional programs are required to do this. Please see the explanation for this on the stunnel. I actually got the info from this site, but remember that sites do change. I will include a PDF file of this into our Devonthink database.

`/etc/nginx/nginx.conf`

```sh
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}

rtmp {
        server {
                listen 1935;
                chunk_size 4096;

                application live {
                        live on;
                        record off;
		                    push rtmp://a.rtmp.youtube.com/live2/<your youtube stream key>;
			                  push rtmp://127.0.0.1:19350/rtmp/<your persistent stream key>;
                }
        }
}

#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
#
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}
```
Alright, right now you would have to change this file every time you made a new stream. To change this, in YT Live select *Create New Stream Key* in the drop-down menu. 
==image==
Once you have created this key, go to the same drop-down menu and select your key name. Copy this key and paste it into the YT area of the nginx.conf file.

Third, you need to configure *stunnel*. Facebook enforces rtmp*s*, so stunnel will allow you to do both rtmp (YT), and rtmps (FB). Run `sudo nano /etc/stunnel/stunnel.conf`. You need to replace the file with this:
```sh
# /etc/default/stunnel
# Julien LEMOINE <speedblue@debian.org>
# September 2003

# Change to one to enable stunnel automatic startup
ENABLED=1
FILES="/etc/stunnel/*.conf"
OPTIONS=""

# Change to one to enable ppp restart scripts
PPP_RESTART=0

# Change to enable the setting of limits on the stunnel instances
# For example, to set a large limit on file descriptors (to enable
# more simultaneous client connections), set RLIMITS="-n 4096"
# More than one resource limit may be modified at the same time,
# e.g. RLIMITS="-n 4096 -d unlimited"
RLIMITS=""
```
Now you need to create a conf.d folder. Do this by running `sudo mkdir /etc/stunnel/conf.d` and `sudo cd /etc/stunnel/conf.d`.
Once in this folder, run `sudo nano fb.conf` and paste
```sh
[fb-live]
client = yes
accept = 127.0.0.1:19350
connect = live-api-s.facebook.com:443
verifyChain = no
```
into it.

Now you have finished configuring! Run `sudo systemctl start nginx && sudo systemctl start stunnel4`. If this is successful, you should be able to run `sudo systemctl status nginx` and `sudo systemctl status stunnel4`. If this isn't working, first make sure you followed the steps carefully, and if there is no error, Google it! Again, this will not work on WSL.

Alright. Now to get your software set up. To do this, go into the streaming settings. Select custom. For the server, paste in *rtmps://your ec2 public dns/live*. For the stream key, paste in your persistent fb stream key. Start streaming, and in a few moments you should see your video appear in both YouTube and Facebook at the same time! 
