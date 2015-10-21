---
title: Raspberry Pi Dashboard
date: 2015-09-14 12:17
---

## Overview

On the corner of my desk, I have a 20" monitor which displays a dashboard
showing various useful information. Occasionally, I replace the computer
responsible for displaying it, and every time I forget to write down what I
did.

This time, I'm writing it down.

## Raspberry Pi

This time, the machine in question is a Raspberry Pi Model B, running Raspbian,
though there's nothing Raspberry Pi-specific to these instructions.

The monitor is a Dell 2005FPW, the Raspberry Pi is in a Pibow case, and I've
used some double-sided &quot;Hook &amp; Loop Pads&quot; (i.e. velcro, but not
Velcro) to attach the two.

## Update Packages

    sudo apt-get update
    sudo apt-get upgrade

## Vim

I'm going to be using `vi` a lot to edit files on the Pi. The default installed
vi is `vim.tiny`, which is terrible. The first thing I'm going to do is install
something a bit more full-featured:

    sudo apt-get -y install vim-nox

## My user account

By default, you can log into the RPi by using the `pi` user, password
`raspberry`. Since everyone in the world knows this password by now, I added my
own user account as follows:

    sudo adduser roger      # follow the prompts

I had to give myself `sudoers` privileges as well:

    sudo usermod -a -G sudo roger

Then I copied my public SSH key to the RPi, using `ssh-copy-id`.

Then I disabled the `pi` user:

    sudo passwd pi -l
    sudo usermod -s /bin/false pi

## Dashing Server

To serve the dashboards, I'm going to use [Dashing](https://dashing.io), so:

    sudo useradd -U dashing -d /home/dashing -m -s /bin/false

Become the `dashing` user:

    su -s /bin/bash -l dashing

Install dashing (also see the instructions at http://dashing.io/#setup):

    sudo apt-get install -y build-essential ruby-dev
    sudo gem install dashing

We'll need an upstart script for the dashing server. Put the following in
`/etc/init/dashing.conf`:

    description "Dashing dashboard server"
    
    respawn
    start on runlevel [23]

    setuid dashing
    setgid dashing

    script
      cd /home/dashing/dashing/
      dashing start
    end script

