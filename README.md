#gsv-6ToWAMP
## GSV-6CPU Modul to WAMP
This shows how to hook up a Raspberry Pi2 to a WAMP router and display real-time GSV-6CPU readings in a browser, as well as the configuration of the GSV-6CPU from the browser.

## What is an GSV6/GSV6-CPU Modul
[GSV6/GSV6-CPU](http://www.me-systeme.de/docs/de/flyer/flyer_gsv6.pdf) is a measurement amplifier from [ME-Messsysteme GmbH](http://www.me-systeme.de/)

## project status
released

## Dependencies
All Dependencies are included.

Dependencie | License
--- | ---
[Autobahn](http://autobahn.ws/) | MIT License
[Bootstrap](http://getbootstrap.com/) | MIT License
[jQuery](https://jquery.com/) | MIT License
[Smoothie](http://smoothiecharts.org/) | MIT License
[Moment.js](http://momentjs.com/) | MIT License
[Highstock](http://www.highcharts.com/products/highstock) | Do you want to use Highcharts/Highstock for a personal or non-profit project? Then you can use Highchcarts/Hightock for free under the  Creative Commons Attribution-NonCommercial 3.0 License

## How it works

The `serial2ws` program will open a serial port connection with your RPi or PC. It will communicate over a specific protocol with the GSV-6CPU-module.

### Control

The `serial2ws` program receives and calls procedures via WAMP.

### Sense

The GSV-6CPU-module sends sensor values by sending its protocol-data via serial. The data can contain a measure-frame, an answer-frame or a request-frame.
The `serial2ws` receives those frames, parses each frame and then receives or publishes WAMP events to the different WAMP-topics with the payload consisting of the frames.


## How to run

You will need to have the following installed on the RPi to run the project: 

* Python or PyPy (used in this installation tutorial)
* Twisted
* AutobahnPython
* PySerial

### Install
All downloads a dropped to ~/downloads and all installs dropped to ~/install

	mkdir ~/downloads
	mkdir ~/install
#### PyPy
PyPy comes pre-installed on a Raspberry Pi with the raspian-image 2015-05-05. It comes with version 2.2.1 but we have to use version 4.0.0.
For the update we have to download the latest version of PyPy, extract files and update paths.
	
	cd ~/downloads
	wget https://bitbucket.org/pypy/pypy/downloads/pypy-4.0.1-linux-armhf-raspbian.tar.bz2
	cd ~/install
	tar xvjf ../downloads/pypy-4.0.1-linux-armhf-raspbian.tar.bz2
	echo "export PATH=\${HOME}/install/pypy-4.0.1-linux-armhf-raspbian/bin:\${PATH}" >> ~/.profile
	source ~/.profile

#### pip
	cd ~/downloads
	wget https://bootstrap.pypa.io/get-pip.py
	pypy get-pip.py
	
#### twisted
	cd ~/downloads
	wget https://pypi.python.org/packages/source/T/Twisted/Twisted-15.4.0.tar.bz2
	cd ~/install
	tar xvjf ../downloads/Twisted-15.4.0.tar.bz2

##### Now we have to comment one line in the twisted-source-code
	cd Twisted-15.4.0
	nano setup.py
	line 63 comment with a # -> -> #conditionalExtensions=getExtensions(),
	Ctrl + O
	Ctrl + X
	pypy setup.py install

#### PySerial
	pip install pyserial
	
#### Autobahn Framework
	pip install autobahn
	
#### numpy
for normal python use

	pip install numpy

we use pypy and under pypy we can't use the normal install method. [Source](http://pypy.org/download.html#installing-numpy)
use the follwing line to isntall numpy with pypy

	pypy -m pip install git+https://bitbucket.org/pypy/numpy.git

#### Install usbmount, for automount usb-store
	sudo apt-get install usbmount
Change usbmount config

	sudo nano /etc/usbmount/usbmount.conf
Goto FS_MOUNTOPTIONS="" and change it to

	FS_MOUNTOPTIONS="-fstype=vfat,gid=users,dmask=0007,fmask=0117"
	Ctrl + O
	Ctrl + X
	sudo reboot
	
#### Crosbar.io (WAMP-Router)
	sudo apt-get install build-essential libssl-dev libffi-dev python-dev
	pip install crossbar
	
### Checkout from github
	cd ~/
	git clone https://github.com/flashbac/gsv-6ToWAMP.git
	
### Set timezone
	cd ~/
	echo "TZ='Europe/Berlin';" >> ~/.profile
	echo "export TZ" >> ~/.profile
	source ~/.profile
	
### Create folder for csv-files
	mkdir messungen
	
### Run crossbar server
	cd <projectname>
	crossbar start &
	check with with the browser http://<ip>:8080 -> some information have to appear there
	
### Run the serial2ws.py script
	pypy serial2ws.py --baud=115200 --port=/dev/ttyAMA0
	goto http://<ip>:8000

## Start crossbar and serial2ws at systemstart
Copy the crossbar-(start)-script and the serial2ws-(start)-script from scripts-folder to /etc/init.d/

	cd ~/gsv-6ToWAMP/scripts
	sudo cp crossbar /etc/init.d/
	sudo cp serial2ws /etc/init.d/
	
Make the script runnable and add crossbar to rc.d

	sudo chmod +x /etc/init.d/crossbar
	sudo chmod +x /etc/init.d/serial2ws
	
from now on, you can start and stop crossbar and serial2ws via the deamon

	sudo /etc/init.d/crossbar start
	sudo /etc/init.d/crossbar stop
	
	sudo /etc/init.d/serial2ws start
	sudo /etc/init.d/serail2ws stop
	
autostart for crossbar and serial2ws
I use the rc.local for them. open /etc/rc.local

	sudo nano /etc/rc.local
	
edit like this

	service networking restart

	# Print the IP address
	_IP=$(hostname -I) || true
	if [ "$_IP" ]; then
	  printf "My IP address is %s\n" "$_IP"
	fi

	/etc/init.d/crossbar start
	/etc/init.d/serial2ws start
	
## Ehternet configuration (OPTIONAL)
if you have no connection (cabel) at eth0, it is better to disable dhcp on eth0. It will be speedup your systemstart und avoid some network glitches.
set up your desired network options in  /etc/network/interfaces
	
	iface eth0 inet static
      address 192.168.2.10
      netmask 255.255.255.0
      
optinal and a gateway

      gateway 192.168.2.1
      
or disable eth0 at all with follwing line in /etc/network/interfaces

	iface eth0 inet manual
	
## Establish the RPi as a Wifi Accespoint with hostapd [Source](http://elinux.org/RPI-Wireless-Hotspot)
Verify that your wifi-adapter is on the [compatible list](http://elinux.org/RPI-Wireless-Hotspot)
and make sure, that you are connected via eth0 (by cable)

	sudo apt-get install firmware-ralink hostapd wireless-tools dnsmasq iw

But be careful that your adapter is compatible with hostapd. Previously I use an EW-7811Un with Realtek RTL8188CUS Chipset and this one will not work out of the box with hostapd.
For the Raspberry Pi you can use a pachted binary from the binary folder or build a [patched Version](https://github.com/lostincynicism/hostapd-rtl8188)
Now I using an Adapter with Ralink RT5370 chipset. This Adapter is compatible to hostapd and works with the default hostapd driver (nl80211). If you have also an compatible adapter too, you can skip the next steps and go further to the dns configuration.

### Use pre-compiled patched hostapd from Binary-Folder

Copy hostapd-binary from git-binary Folder

	cd /usr/sbin
	sudo mv /usr/sbin/hostapd /usr/sbin/hostapd.bak
	sudo cp gsv-6ToWAMP/binary/hostapd hostapd
	sudo chown root:root hostapd
	sudo chmod 755 hostapd


### Build hostapd-rtl8188 (patched Version)
First of all you have to clone the repo

	cd ~
	git clone https://github.com/lostincynicism/hostapd-rtl8188
then install dependencies

	sudo apt-get install libnl-3-dev libnl-genl-3-dev

build hostapd-rtl8188
	
	cd hostapd-rtl8188/hostapd
	make
	
copy hostapd-binary

	cd /usr/sbin
	sudo mv /usr/sbin/hostapd /usr/sbin/hostapd.bak
	sudo cp hostapd-rtl8188/hostapd/hostapd hostapd
	sudo chown root:root hostapd
	sudo chmod 755 hostapd


### Configuration of the Accespoint
#### Configure DHCP-Server for wireless-interface
create a backup from the orginal dnsmasq-config-file

	sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
	
Open dnsmasq-config-File with the following command
	
	sudo nano /etc/dnsmasq.conf
	
Configure DHCP. Edit the file /etc/dnsmasq.conf and configure it like this
	

	interface=wlan0
	no-dhcp-interface=eth0
	dhcp-range=interface:wlan0,192.168.9.2,192.168.9.30,infinite
		
save dnsmasq.conf changes and exit nano with
	
	Ctrl + O
	Ctrl + X
	
#### Configure wireless-interface
You will need to give the Pi a static IP address on the wireless interface with the following command

	sudo ifconfig wlan0 192.168.9.1
	
To automatically set this up on boot, edit the file /etc/network/interfaces, open it by typing the following command

	sudo nano /etc/network/interfaces
	
and replace the line "iface wlan0 inet dhcp" to
(If the line "iface wlan0 inet dhcp" is not present, add the above lines to the bottom of the file.)

	iface wlan0 inet static
	  address 192.168.9.1
	  netmask 255.255.255.0
	  broadcast 192.168.9.255

Change the lines (they probably will not all be next to each other)

	allow-hotplug wlan0
	wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
	iface wlan0 inet dhcp

to

	#allow-hotplug wlan0
	#wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
	#iface wlan0 inet manual

and add one the file end follwing lines:

	# restart hostapd and dnsmasq
		up service hostapd restart
		up service dnsmasq restart
		
#### Configure hostapd
Configure HostAPD. Create a WPA-secured network. To create a WPA-secured network, open the file hostapd.conf

	sudo nano /etc/hostapd/hostapd.conf
	
and add the following lines and change the ssid-, channel- and wpa_passphrase-line to values of your choice.
It seems to be necessary that the passphrase starts with a capital letter.

	interface=wlan0
	driver=nl80211
	ssid=ME_AP
	hw_mode=g
	channel=6
	macaddr_acl=0
	auth_algs=1
	ignore_broadcast_ssid=0
	wpa=2
	wpa_passphrase=My_Passphrase
	wpa_key_mgmt=WPA-PSK
	wpa_pairwise=TKIP
	rsn_pairwise=CCMP

open  /etc/default/hostapd

	sudo nano /etc/default/hostapd

and change

	#DAEMON_CONF=""
 
to

	DAEMON_CONF="/etc/hostapd/hostapd.conf"

exit with

	Ctrl + O
	Ctrl + X

Now run the following commands to start the access point

	service networking restart
	
Your Pi should now be hosting a wireless hotspot. Test it.

last step reboot

	sudo reboot

sometimes after reboot (and hostapd start) the pi doesnt assigne a IP  address to wlan0. You can solve it by chage file /etc/default/ifplugd and change it like this [Source](http://rpi.vypni.net/wifi-ap-rt5370-on-raspberry-pi/):

	sudo nano /etc/default/ifplugd
	
and change to
	
	INTERFACES="eth0"
	HOTPLUG_INTERFACES="eth0"
	ARGS="-q -f -u0 -d10 -w -I"
	SUSPEND_ACTION="stop"
