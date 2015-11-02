# Raspberry Pi Dashboard

## Overview

Raspberry Pi, stuck to the back of a spare 20" monitor, running **Dashing**,
**Chromium** (in kiosk mode), and a couple of other useful things.

TODO: More words here.

## Raspberry Pi

Take one RPi Model B. Put it in a Pibow case.

## Installing Raspbian

From https://www.raspberrypi.org/downloads/raspbian/ download the Raspbian
"Jessie" build.

Then, following the instructions at
https://www.raspberrypi.org/documentation/installation/installing-images/README.md,
we end up doing the following:

    cd ~/Downloads/
    unzip 2015-09-24-raspbian-jessie.zip

    # Warning: Be sure that you've got the correct output device!
    sudo dd bs=4M if=2015-09-24-raspbian-jessie.img of=/dev/sdd
    sync

## Configuring Raspbian

Raspbian "Jessie" uses a GUI, rather than the old text-based UI. To configure
it, go to Menu / Preferences / Raspberry Pi Configuration, then:

- On the "System" tab:
  - click "Expand Filesystem"
  - set the hostname.
  - Boot: To CLI (http://www.modmypi.com/blog/boot-to-command-line-raspbian-jessie)
  - Auto login: uncheck
- On the "Interfaces" tab, disable everything except SSH.
- On the "Performance" tab, set the GPU memory to "16" (GB).
- Click OK; Click "Yes" when asked if you want to reboot.

## Connecting to Wifi

- Ralink RT5370

Edit `/etc/wpa_supplicant/wpa_supplicant.conf`, as follows:

    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1

    network={
	scan_ssid=1
	proto=WPA RSN
	key_mgmt=WPA-PSK
	pairwise=CCMP TKIP
	group=CCMP TKIP
	ssid="<WIFI SSID>"
	psk="<WIFI PASSWORD>"
    }

Then restart the Wifi interface:

    sudo ifdown wlan0
    sudo ifup wlan0

If it's working, `ifconfig` should show an IP address for the `wlan0`
interface. You might need to wait for a few seconds.

## DHCP / DNS

If you're using DHCP to assign an IP address to the Raspberry Pi, don't forget
to set up a DHCP reservation and a DNS entry for it.

Note that the Raspberry Pi uses the DHCPv6 *DUID* value for DHCPv4 as well.
You can edit the `/etc/dhcpcd.conf` file if you want to change this behaviour.

## Logging in

At this point, you should be able to use SSH to log into the Raspberry Pi:

    ssh pi@raspberrypi      # assuming you've set up DNS properly

## Update packages and firmware

    sudo apt-get update
    sudo apt-get upgrade
    sudo rpi-update
    sudo shutdown -r now    # needed to activate the new firmware

## User accounts and SSH

By default, the Raspberry Pi has a user account, `pi`, with the password
`raspberry`. We need to disable that, which first means creating a user account
for us to use:

    sudo adduser roger      # assuming your name is 'roger' :-)

(follow the prompts)

You'll also need to ensure that the new user account can use `sudo`. To do this:

    sudo usermod -a -G sudo roger

Then SSH in using that user account:
    
    # on another PC
    ssh roger@raspberrypi

If that works, then you can disable the `pi` user. Do this as your new user
account, to make sure that _something_ has `sudo` privileges:

    # on the Raspberry Pi, as yourself
    sudo usermod -L pi
    sudo usermod --expiredate 1 pi
    sudo chsh -s /bin/false pi
    sudo deluser pi sudo

If you want to avoid being prompted for a password every time you use SSH, then
use the `ssh-copy-id` command:

    # on another PC
    ssh-copy-id roger@raspberrypi

## Personalisation

I keep my dotfiles on github, so I want to install those:

    mkdir -p Source/rlipscombe
    git clone https://github.com/rlipscombe/dotfiles.git Source/rlipscombe/dotfiles
    ./Source/rlipscombe/dotfiles/mk

    git clone https://github.com/rlipscombe/vimrc.git Source/rlipscombe/vimrc
    pushd Source/rlipscombe/vimrc
    git submodule init
    git submodule update
    popd
    ./Source/rlipscombe/vimrc/mk

## Basic Packages

I prefer vim to nvi (or nano), so:

    sudo apt-get -y install vim-nox

Might as well remove some unwanted packages as well:

    sudo apt-get -y remove minecraft-pi sonic-pi wolfram-engine
    sudo apt-get -y autoremove

## Reinstalling Ruby

Earlier, the `apt-get autoremove` removed Ruby, so we'll put that back,
and we might need the `ruby-dev` package as well:

    sudo apt-get -y install ruby ruby-dev   # installs Ruby 2.1

## RVM prerequisites

Also install some things we'll need later:

    sudo apt-get -y install gawk libreadline6-dev libssl-dev \
                    libyaml-dev libsqlite3-dev \
                    sqlite3 autoconf libgdbm-dev libncurses5-dev \
                    automake libtool bison libffi-dev

## Installing node.js

Dashing uses `execjs`, which requires a JavaScript runtime. We'll use node.js;
see http://node-arm.herokuapp.com/

    wget http://node-arm.herokuapp.com/node_latest_armhf.deb
    sudo dpkg -i node_latest_armhf.deb

## `dashing` user

I'm going to run it under a different user account:

    sudo useradd -d /opt/dashing -s /bin/false -m -U dashing

To install dashing as the `dashing` user, we need to become that user:

    sudo -s -u dashing

## Installing Ruby using RVM

We installed Ruby above, but we'll need to use RVM (https://rvm.io/) to manage
our Gemsets, so:

    # as the 'dashing' user...
    gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
    \curl -sSL https://get.rvm.io | bash -s stable --autolibs=read-fail
    source ~/.rvm/scripts/rvm

Note the `--autolibs=read-fail`, per http://stackoverflow.com/a/17219765/8446

    rvm install 2.2.1

This will take around **2 hours** to complete.

Why use RVM? Because otherwise `gem` (and `bundler`) will occasionally want to
install things globally, and they'll ask for a password for `sudo`, and the
`dashing` user doesn't have `sudo` privileges. It's a big pile of complication
I could do without, so: use RVM, even if it's more moving parts.

Create a file `~/.gemrc` containing the following:

    gem: --no-rdoc --no-ri --verbose

## IP: Installing Dashing

As a dashboard, I'm going to use **Dashing** from http://dashing.io/.

Use the correct version of Ruby:

    # As the 'dashing' user:
    source ~/.rvm/scripts/rvm
    rvm use 2.2.1

We'll need bundler:

    gem install bundler

To install dashing, we can use:

    gem install dashing

This skips building documentation, and it's verbose, because it takes a while,
and I like to have something to look at.

Check it's working:

    cd ~
    dashing new dashboard   # creates a new dashboard in the 'dashboard' directory.
    cd dashboard
    bundle install          # only needed for the first dashboard.
    dashing start

## TODO: Making dashing start automatically

<--- HERE

## TODO: Chromium Kiosk
## TODO: Jenkins Tunnel

