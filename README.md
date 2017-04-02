# Experiments and Notes for Onion Omega2+

This repo will contain experiments I do with my Omega2+ as well as some notes for doing various things.

I exclusively use `vi` in this documentation, you may substitute `nano` after installing it.

My Omega's IP address is shown in examples here as `10.10.10.250`, you'll need to substitute your own IP address.

My client machine is OSX or Linux, I don't really know how to do anything on Windows any more, I have not really used MS products since WinXP.  These instructions are geared toward using the shell in those operating systems, you can install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) and Linux in your Windows OS and play along if you'd like.

## Experiments

I've just been messing with the OS, there's nothing here yet.

Eventually you'll be able to clone this to your Omega and mess around with it.

## Wifi Setup from Command Line

### Connect to Wifi Network

Plug your Omega in and from your computer connect to the wifi network it created.  It will usually be *Omega-XXXX*, where XXXX is the last four letters of your MAC address as shown on top of the device.  The password for this network is `12345678`.

### SSH Into Device

Then SSH into it.  The standard IP address is always `192.168.3.1`.

    ssh root@192.168.3.1

#### SSH Host Identification Error

If you have shelled into that IP before you will get the following error.

	@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
	@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
	@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
	IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
	Someone could be eavesdropping on you right now (man-in-the-middle attack)!
	It is also possible that a host key has just been changed.
	The fingerprint for the RSA key sent by the remote host is
	SHA256:LJ/ODrlh30b/qpEMFFbuI1Foz2gWIJu/zsx5w9L+xBY.
	Please contact your system administrator.
	Add correct host key in /Users/pjobson/.ssh/known_hosts to get rid of this message.
	Offending RSA key in /Users/pjobson/.ssh/known_hosts:10
	RSA host key for 192.168.3.1 has changed and you have requested strict checking.
	Host key verification failed.

To remove your key, either edit your known_hosts file or run:

	ssh-keygen -R 192.168.3.1

### Wifi Setup

You can use the `wifisetup` command to configure your wifi without the web interface.

    root@Omega-ADBD:~# wifisetup
    Onion Omega Wifi Setup

    Select from the following:
    1) Scan for Wifi networks
    2) Type network info
    q) Exit

    Selection:

If you are not hiding your SSID, select `1`, otherwise you can manually enter it with `2`.

    Selection: 1
    Scanning for wifi networks...

    Select Wifi network:
    1) Neighbor's Network 1
    2) Neighbor's Network 2
    3) Neighbor's Network 3
    4) Neighbor's Network 4
    5) Whatever You Named Your Network
    6) Neighbor's Network 5
    7) Neighbor's Network 6
    8) Neighbor's Network 7

Select your network.

    Selection: 5
    Network: Whatever You Named Your Network
    Authentication type: WPA2PSK
    Enter password: Your_Password_Here

    > Restarting wifimanager for changes to take effect

This sometimes takes a bit to get an IP address, after it is done just reboot and exit to your shell.  You can use the `&&` to run a second command after the first is complete.

    reboot && exit

This should bring the device back up on your regular network.

## Update Firmware from Command Line

**As of version 0.1.9-b158 updating your firmware will wipe your device.**

The update images for the device are hosted on [onion.io](http://repo.onion.io/omega2/images/).  You can use `oupgrade` to automagically update your firmware.

    oupgrade --help
    
    Functionality:
      Check if new Onion firmware is available and perform upgrade

    Usage: /usr/bin/oupgrade

    Arguments:
     -h, --help        Print this usage prompt
     -v, --version     Just print the current firmware version
     -l, --latest      Use latest repo version (instead of stable version)
     -f, --force       Force the upgrade, regardless of versions
     -c, --check       Only compare versions, do not actually update
     -u, --ubus        Script outputs only json

You can force upgrade to a specific version with `sysupgrade`.  At this writing `v0.1.9-b157` was the latest version.

    wget -P /tmp http://repo.onion.io/omega2/images/omega2p-v0.1.9-b157.bin 
    sysupgrade -n /tmp/omega2p-v0.1.9-b157.bin

This will output something like this and eventually your connection will be broken.

    killall: watchdog: no process killed
    Sending TERM to remaining processes ... uhttpd device-client avahi-daemon onion-helper udhcpc udhcpc packet_write_wait: 
    Connection to 10.10.10.250 port 22: Broken pipe
    
Don't turn the device off while it is updating or you could brick it.  There is an article on possibly unbricking your device on the [community page](https://community.onion.io/topic/1154/omega-2-usb-firmware-install-after-brick-resolved/4).

After you update you will need to delete the device from your local `~/.ssh/known_hosts` file.

## Forcing an IP Address in an OpenWRT Router

If you use an OpenWRT router and probably other routers you can setup a Static Lease to bind the Omega's MAC address to a specific IP.  This makes it easy to find your device on the network, if you happen to forget the IP address.

Login to your router's web interface and navigate to **Network** ⟶ **DHCP and DNS**.  Here at the bottom you should see *Static Leases* and you can add your Omega's specific MAC address and an IP address on your network.  Here is mine for example, you can see I have a server and a printer listed in there as well.

![Static Leases](./images/screenshot_49.png)

After you bind the MAC to the IP address, hit Save & Apply then reboot or power cycle your Omega.

## Install Some Packages

### Update the Package Manager

    opkg update

### Install GIT

    opkg install git git-http ca-bundle

### Install a Better Text Editor

    opkg install vim
    ### --- OR --- ###
    opkg install nano

### Install BASH

    opkg install bash
    vi /etc/passwd

Change this:

    root:x:0:0:root:/root:/bin/ash

To this:

    root:x:0:0:root:/root:/bin/bash

Log out of the Omega's shell and log back in for this to take effect.  You may test with:

    echo $SHELL

## Create Your `.profile`

Create your local bin.

    mkdir ~/bin

Touch your profile.

    touch ~/.profile

Edit it.

    vi ~/.profile

Add some stuff to it.

    alias l='ls -lah'
    export PATH=~/bin:$PATH
    
Source it, this applies the changes you made.

    source ~/.profile

## SSH

### Default Login / Password

If your device is going to be in the wild, be sure to change these.

    Username: root
    Password: onioneer
    

### Generate SSH Keys

    mkdir ~/.ssh
    dropbearkey -t rsa -f ~/.ssh/id_rsa
    dropbearkey -y -f ~/.ssh/id_rsa | sed -n 2p > ~/.ssh/id_rsa.pub

### SSH to Omega without Password

On your client machine, if you have not, generate a public and private key with `ssh-keygen`.  You may set a password if you'd like.

    ssh-keygen

Now cat the client's `id_rsa.pub` into your Omega, you'll need to change my IP to yours here.

    cat ~/.ssh/id_rsa.pub | ssh root@10.10.10.250 'cat >> /etc/dropbear/authorized_keys'

This will prompt for your Omega's password then throw an error, you can ignore the error.

    root@10.10.10.250's password:
    shell-init: error retrieving current directory: getcwd: cannot access parent directories: Not a tty

Now when you ssh from the client to the Omega it should not prompt you.

    root@10.10.10.250

## Setting Up Git

Full readme here: [docs/git_setup.md](docs/git_setup.md)

## Setting Up SDCARD for `/root` and SWAP

Full readme here: [docs/setting_up_sdcard_for_root_and_swap.md](docs/setting_up_sdcard_for_root_and_swap.md)

## Node.js

Install Node and NPM

    opkg install nodejs
    opkg install npm

I like to store my global node modules in my root directory, so as to not take up too much room in the root file system, this along with my instructions above for setting up SDCARD for `/root` gives me plenty of extra space.

    mkdir -p /root/node/bin
    npm config set prefix /root/node

Add the node bin to your `.profile`, either edit it or echo to it.

    echo "export PATH=~/node/bin:\$PATH" >> ~/.profile
    source ~/.profile

Test it out.

    npm install -g lorem-ipsum
    lorem-ipsum
    # should output some lorem ipsum text
    npm remove -g lorem-ipsum
    
## File Transfer

To send/receive files to/from the device I recommend using `scp`, you'll want to substitute your device's IP for the one I have listed.

    # Receive a single file from your etc to your local machine
    scp root@10.10.10.250:/etc/openwrt_release .
    
    # Receive a full directory from /www to your local machine
    scp -r root@10.10.10.250:/www .
    
    # Send a single file to your root directory
    scp localfile.txt root@10.10.10.250:~/
    
    # Send a full directory to your root directory
    scp -r /some_path root@10.10.10.250:~/
    

## External Antenna

You can add an external antenna to the device with the IPX jack on the main unit.

![IPX Jack](./images/omega2-IPX.jpg)

You can convert it to a standard wifi antenna with an **IPX to RP-SMA adapter**.  You can get these on [Amazon - ipx to rp-sma](https://goo.gl/yXCly5) or [eBay - ipx to rp-sma](https://goo.gl/BwT5Vr) or various other places.  

![IPX to RP-SMA](./images/ipx-to-rp-sma.jpg)

Otherwise you can use an IPX antenna, I used this one which I had laying around and afixed the antenna to the bottom of the unit.  You can get IPX antennas also from again [Amazon - IPX antenna](https://goo.gl/pWwB2Q) or [eBay - IPX antenna](https://goo.gl/uabwr6) or various other places.

![IPX Modification](./images/ipx-mod.jpg)

I got a slight speed increase using [speedtest-cli](https://github.com/sivel/speedtest-cli) after adding the external antenna.

Before external antenna.

	root@Omega-ADBD:/tmp# ./speedtest-cli
	Retrieving speedtest.net configuration...
	Testing from Verizon Fios (xxx.xxx.xxx.xxx)...
	Retrieving speedtest.net server list...
	Selecting best server based on ping...
	Hosted by Shentel Service Company (Ashburn, VA) [7.69 km]: 21.32 ms
	Testing download speed................................................................................
	Download: 10.16 Mbit/s
	Testing upload speed....................................................................................................
	Upload: 11.48 Mbit/s


After external antenna.

	root@Omega-ADBD:/tmp# ./speedtest-cli
	Retrieving speedtest.net configuration...
	Testing from Verizon Fios (xxx.xxx.xxx.xxx)...
	Retrieving speedtest.net server list...
	Selecting best server based on ping...
	Hosted by GigeNET (Ashburn, VA) [7.69 km]: 24.893 ms
	Testing download speed................................................................................
	Download: 16.73 Mbit/s
	Testing upload speed....................................................................................................
	Upload: 10.14 Mbit/s
