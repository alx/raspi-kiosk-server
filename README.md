raspi-kiosk-server
==================

Raspberry Kiosk Server : start raspberrypi in web-browser kiosk mode, and use it as a webserver.

# Setup

## SD Card flashing

You'll need latest Raspbian image to flash your sdcard : http://www.raspberrypi.org/downloads/

* Insert sdcard in your sdcard reader
* Mount partition
* Detect the correct /dev/sdX with : ```df -h```
* flash card, with the correct path in ```if``` parameter :

```
dd bs=4M if=~/Downloads/2014-01-07-wheezy-raspbian.img of=/dev/sdc
```

For complete **sdcard flash manual**, please use eLinux documentation : http://elinux.org/RPi_Easy_SD_Card_Setup

## Boot config

The boot partition store the *cmdline.txt* that setup the fixed ip adress of the raspberrypi, and the *autostart* file that will setup the desktop display.

See network part of this manual to learn how to attribute the X ip to each raspberry.

* Mount /boot partition and edit cmdline.txt, add ```ip=10.42.0.X::10.42.0.1``` at the end of line
* Copy config/autostart on /boot partition, and edit the url on last line

## Desktop config

For the last step, you'll need to connect ot your raspberrypi using ssh.

* Put the sdcard in the raspi and start it
* Connect by ssh from remote host using : ```ssh pi@10.42.0.X``` (password: raspberry)
* launch *raspi-config* tool to force raspberrypi to start on Desktop : ```raspi-config```
* Edit */etc/lightdm/lightdm.conf* and use this setting in *[Seat]* section : ```xserver-command=X -s 0 dpms```
* Replace LXDE autostart script by the one on /boot partition :

```
cd /etc/xdg/lxsession/LXDE/
sudo rm autostart
sudo ln -sf /boot/autostart
```

Optional : when using *raspi-config*, you can choose to expand the disp-space used on the sd-card

## Server config

In this section, we'll install nginx/mysql/php5/nodejs/laravel to have a full-stack webserver ready on the raspberrypi.

Raspberrypi kiosk clients will connect to this server to have their content served.

Connect to ssh, and use these commands :

```
sudo apt-get update
sudo apt-get install vim mysql git nginx php5-fpm php5-cgi php5-cli php5-common php5-mcrypt php5-mysql redis-server
wget http://node-arm.herokuapp.com/node_latest_armhf.deb
sudo dpkg -i node_latest_armhf.deb
rm node_latest_armhf.deb
curl https://www.npmjs.org/install.sh | sudo sh
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

Now we'll setup the raspberrypi to served the content of *www* folder in the repository to its web clients.

Still in ssh :

```
sudo mkdir /var/www
sudo chmod 775 /var/www -R
sudo chown www-data:www-data /var/www
sudo su www-data
git clone https://github.com/alx/raspi-kiosk-server.git
```

Exit from **www-data** user (or disconnect/reconnect from ssh), and use these commands :

```
sudo cp /var/www/raspi-kiosk-server/config/nginx_default.conf /etc/nginx/sites-enabled/default
sudo /etc/init.d/nginx reload
```

If you connect to http://10.42.0.X, you should view an **Hello World** page.

Other raspberrypi client configured in kiosk mode (with the right url) should automaticly display this page.

## Using PHP to connect different screens

Mini-laravel app that use routes to serve different displays to clients.

## Using nodejs and socket.io to refresh screens from server

http://www.volkomenjuist.nl/blog/2013/10/20/laravel-4-and-nodejsredis-pubsub-realtime-notifications/

# Network setup

Describe how the allocation of ip adresses is working.
