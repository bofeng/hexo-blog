---
title: 'Install ubuntu on Acer C720 Chromebook'
date: 2014-08-16 21:00:00
tags: [ubuntu,chromebook]
published: true
hideInList: false
feature: 
---
The price of Acer C720 chromebook now is just $179 in bestbuy, plus tax it is still less than $200, with 16G SSD and up to 8.5 hrs battery life, the idea is you can buy such a laptop and install ubuntu on it to make it as you dev machine, for example, you can buy one for your kids and it is really good for them to learn how to code.

<!-- more -->


### Steps

1. Before you do these steps, it is a good idea to have a recovery image, get a SD card or USB Flash Drive, burn image to it, following this page: Restore your Chromebook or you can do this step on another computer only if you failed this installation.
2. Press esc + refresh + power to go to dev mode
3. As soon as you see Recovery Mode pop up—the screen with the yellow exclamation point—press ctrl+d. This will bring up a prompt asking if you want to turn on dev mode, press enter to confirm this.
4. After it rebooted, you will see a red exclamation, leave it alone to restart chrome os.
5. When you are in chrome os, set your wifi, etc, then login.
6. Open your chrome (web browser), go to http://goo.gl/fd3zc to download Crouton, it should save a file named crouton in your Downloads folder.
7. Press ctrl + alt + t go to chrome os terminal “crosh”, then type “shell”.
8. Install ubuntu: `sudo sh -e ~/Downloads/crouton -a i386 -r trusty -t audio,core,gtk-extra,keyboard,x11,chrome,cli-extra,extension,unity`
9. Now you gotta get some coffee and wait for a while, it usually takes 20~30 minutes, after it is installed, system will ask you for username and password. Then the system will give a hint to go chroot, type: `sudo enter-chroot`, then type `startunity`. It will bring you to ubuntu, or you can type “sudo startunity” if you want to enter ubuntu as root user.
10. Install extra softwares: after you enter ubuntu, you may want to install some softwares to setup your dev environment, for example, you may want to install classic gnome terminal instead of XTerm which is builtin, or some editors you want to use like vim: `sudo apt-get install gnome-terminal vim aptitude`

### Switch b/t ChromeOS and Ubuntu

The most excited thing may be you could freely switch b/t ubuntu and chromeos, when you are in ubuntu, just use ctrl + alt + f1 (the left arrow key near esc) to go back to chromeos; when you are in chromeos, use ctrl + alt + shift + f1 to enter ubuntu.

Want to quit ubuntu? just click logout in ubuntu, it will bring you back to chromeos. To re-start it after you quit ubuntu, in your chromeos, use ctrl + alt + t to open crosh, type “shell” then type “sudo enter-chroot” and then type “startunity”.


### Reference

1. [http://lifehacker.com/how-to-install-linux-on-a-chromebook-and-unlock-its-ful-509039343](http://lifehacker.com/how-to-install-linux-on-a-chromebook-and-unlock-its-ful-509039343)
1. [http://raywaldo.com/2013/10/5-steps-install-linux-chromebook/](http://raywaldo.com/2013/10/5-steps-install-linux-chromebook/)
