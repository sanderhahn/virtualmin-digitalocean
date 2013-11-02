# Setup Virtualmin on Digital Ocean using Ubuntu 12.04

Virtualmin is an open source admin panel that
makes managing your webserver easier. Digital Ocean
provides a clean interface to quickly create new
VPS servers. Ubuntu 12.04 is a reliable linux
distribution that is a LTS release (long time support).

Other nice VPS hosting providers are:
- [Linode](https://www.linode.com/) International
- [DirectVPS](https://www.directvps.nl/) The Netherlands

## Create a Droplet

Sign up at [Digital Ocean](https://www.digitalocean.com/) and
add your SSH key.

```
Name: virtualmin
Size: 512MB should work, you can resize later
Region: Amsterdam 1
Image: Ubuntu "Ubuntu 12.04 x64"
SSH Key: Select your SSH key
```

Click "Create Droplet"

## Initial Setup

Digital Ocean shows the IP address of your new VPS
and that can be used to connect using SSH:

```bash
ssh root@<ip-address>
```

Replace `<ip-address>` part with the ip address of your
VPS.

Once inside your VPS we will update and upgrade the
packages, setup the timezone and locale.

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

## Setup Swap

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

Download the installer script and start using the shell. The full
instructions are available at:
[Virtualmin Installation Quickstart](http://www.virtualmin.com/documentation/installation/automated).

```bash
wget http://software.virtualmin.com/gpl/scripts/install.sh
sh install.sh
```

Answer yes and provide a full qualified domain name like:
`virtualmin.yourdomain.com`. Domain providers like
[TransIP](https://www.transip.nl/)
or [Gandi.net](https://www.gandi.net/) over easy panels to setup your DNS records.

Generate a random password that you can use for the root user:

```bash
PASSWD=`pwgen 12 1`
/usr/share/webmin/changepass.pl /etc/webmin root $PASSWD
echo Password is $PASSWD
```

## Reboot and Finish Installation

After the installation is finished you can access the control panel at:
`https://<ip-address>:10000` or `https://virtualmin.yourdomain.com:10000`.
Your browser will warn you because the certificate is self signed
and not provided by an offical certificate provider. 

Use default values in the "Post-Installation Wizard" steps. You can reuse the
generated root password for the MySQL database. Select "Only store hashed
passwords" in the password storage mode step.

The "Re-Check and Refresh Configuration" doesn't immediately work on
my system, so lets first reboot the system using:

```
Webmin > System > Bootup and Shutdown > Click "Reboot System" (down the page)
```

After a minute or so the system is back online and clicking on "Re-Check
Configuration" will display that everything is okay.

You can start configuring virtual hosts using "Create Virtual
Server" and it will setup hosting, database and email for you.
The configuration that can be done using Virtualmin is extensive but the interface is
much easier than doing manual configuration using the commandline.
Take a dive into the [Step by step tutorials](http://www.virtualmin.com/documentation/tutorial)
to learn more.

Congratulations you are now running your own Linux server :)
