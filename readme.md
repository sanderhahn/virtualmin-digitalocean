# Setup Virtualmin on Digital Ocean using Ubuntu 12.04

Virtualmin is an open source admin panel that
makes managing your webserver easier. Digital Ocean
provides a clean interface to quickly create new
VPS servers. Ubuntu 12.04 is a reliable linux
distribution that is a LTS release (long time support).

## Create a droplet at Digital Ocean:

```
Name: virtualmin
Size: 512MB should work, you can resize later
Region: Amsterdam 1
Image: Ubuntu "Ubuntu 12.04 x64"
SSH Key: Select your SSH key
```

Click "Create Droplet"

## Inital Setup

Digital Ocean shows the IP address of your new VPS
and that can be used to connect using SSH:

```bash
ssh root@<ip-address>
```

Replace `<ip-address>` part with the ip address of your
VPS.

Once inside your VPS we will update and upgrade the
packages, setup the timezone and locale. Confirm using
yes to the questions:

```bash
apt-get update
apt-get upgrade -y
apt-get install -y ntp
```

NTP makes sure that your server time is kept up to date.
We select Europe/Amsterdam for the timezone and set the
locale to English/US.

```bash
dpkg-reconfigure tzdata
update-locale LC_ALL=en_US.UTF-8 LC_MESSAGES=POSIX
```

## Setup swap

The full instructions for swap setup are available at:
[How To Add Swap on Ubuntu 12.04](https://www.digitalocean.com/community/articles/how-to-add-swap-on-ubuntu-12-04).

```bash
dd if=/dev/zero of=/swapfile bs=1024 count=512k
mkswap /swapfile
swapon /swapfile
echo '/swapfile       none    swap    sw      0       0' >>/etc/fstab
echo 0 > /proc/sys/vm/swappiness
chown root:root /swapfile
chmod 0600 /swapfile
```

Confirm that there is a swapfile displayed when you execute `swapon -s`.

## Setup Virtualmin

Download the installer script and start using the shell, full
description available at:
[Virtualmin Installation Quickstart](http://www.virtualmin.com/documentation/installation/automated).

```bash
wget http://software.virtualmin.com/gpl/scripts/install.sh
sh install.sh
```

Answer yes and provide a full qualified domain name like:
`virtualmin.yourdomain.com`. Using a domain provider like TransIP
or Gandi.net makes it easy to setup the DNS records.

Generate a random password that you can use for the root user:

```bash
PASSWD=`pwgen 12 1`
/usr/share/webmin/changepass.pl /etc/webmin root $PASSWD
echo Password is $PASSWD
```

## Reboot and Finish Installation

After the installation is finished you can access the control panel at:
`https://<ip-address>:10000` or `https://virtualmin.yourdomain.com` if the
dns changes are propagated.

Use defaults in the "Post-Installation Wizard", you can reuse the
root password for the MySQL root password. Select "Only store hashed
passwords" in the password storage mode step.

The "Re-check and refresh configuration" doesn't immediately work on
my system, so i first reboot it using:

```
Webmin > System > Bootup and Shutdown > Click "Reboot System" (down the
page)
```

After a minute or so the system is back online and click on "Re-Check
Configuration" and everything should be okay :)

Now you can start configuring virtual hosts using "Create Virtual
Server" and it will setup hosting, database and email. The configuration
that can be done using Virtualmin is extensive but the interface is
meant to be easier than doing the configuration using the commandline.

