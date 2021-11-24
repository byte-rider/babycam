# Baby camera and stream to the grandparents using a Raspberry Pi

## Buy the hardware
* [Raspberry Pi Zero W](https://core-electronics.com.au/raspberry-pi-zero-w-wireless.html)
* [Night Vision Camera](https://www.ebay.com.au/itm/114513687410?hash=item1aa98c3372:g:n2QAAOSwuihfrKj5)
* [Articulating mobile phone holder](https://www.amazon.com.au/Phone-Holder-Bed-Gooseneck-Mount/dp/B095Z14NPQ/ref=sr_1_25_sspa?keywords=Gooseneck+Phone+holder&qid=1637710958&s=electronics&sr=1-25-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUExRDNQRVRPUTNLRDQ4JmVuY3J5cHRlZElkPUEwNjIwODI0MkxLRTVDU1VENTc3TiZlbmNyeXB0ZWRBZElkPUEzU05HUEE2V1JDQUREJndpZGdldE5hbWU9c3BfYnRmJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==)

## Install Raspberry Pi OS (Buster)
Download the Rapsberry Pi OS 'Buster' 32-bit version and install it to an SD card.
Why not 'Bullseye'? At the time of writing Bullseye changed to a different camera stack which didn't work with the infrared camera I had purchased. I suspect bugs will be ironed out eventually.
* [2021-05-07-raspios-buster-armhf-lite.zip](https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2021-05-28/)

You can install it to an SD using the Official [Raspberry Pi Imager](https://www.raspberrypi.com/software/)

## Configure the software
Boot up, log in, then update everything
```
sudo apt update && sudo apt upgrade -y
```

Turn on the Camera module via raspi-config
```
# Turn on camera via Interface options
sudo raspi-config
```

Download this repo by cloning it
```
mkdir -p ~/babycam
git clone https://github.com/george-edwards-code/babycam.git ~/babycam/
```

Run the installer
```
~/babycam/install.sh
```

Setup cron jobs
```
crontab -e
```

Replace crontab with the following
```
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)

SHELL=/bin/bash

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

# turn the LED OFF at 8pm each night
  * 20  *  *  * sudo systemctl start babycam-led-disable.service

# turn the LED ON at 6am each day
  *  6  *  *  * sudo systemctl stop babycam-led-disable.service

# reboot the system at 2am each day
  *  2  *  *  * sudo reboot now
```

Reboot machine
```
sudo reboot now
```

## Accessing the admin interface and the stream
Admin interface: 
```
http://<ip-address-of-pi>:8765
```
Stream: 
```
http://<ip-address-of-pi>:8071
```

I suggest setting a DHCP reservation so your raspberry pi's local IP address doesn't change.

# Create a website to allow grandparents access
Register a domain from [Google Domains](https://domains.google.com/registrar/search)
Setup a Dynamic DNS 'A' record for your domain. Current instructions can be found [here](https://support.google.com/domains/answer/6147083?hl=en)
Google will issue a random username and password which will be used to update the DNS using client software.

Setup the DNS updating client software on the Raspberry Pi
```
sudo apt install ddclient -y
# installation comes with a config wizard. Satisfy the wizard with any settings you want, we will overwrite them later.
```

Configure the client appropriately.
```
sudo nano /etc/ddclient.conf
```

Delete everything in that file and replace it with the following, being sure to add your username and password(s) issued by Google
```
# Configuration file for ddclient generated by debconf
#
# /etc/ddclient.conf

protocol=googledomains
use=web
server=domains.google.com
daemon=100

# yourdomain.com
login=
password=''
yourdomain.com

# www.yourdomain.com
login=
password=''
www.yourdomain.com
```

Now restart the service
```
sudo systemctl restart ddclient
```

And you can test your config using the following:
```
sudo ddclient -query
```

And finally, setup a port forward on your home router. Forward all incoming TCP traffic from port `80` to `<ip-address-of-pi>:8071`.

