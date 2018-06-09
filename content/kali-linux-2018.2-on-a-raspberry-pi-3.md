---
title: "How to Install Kali Linux 2018.2 on a Raspberry Pi 3"
date: 2018-06-07T18:54:59+01:00
draft: true
type: "post"
---

About 15 years or so, I decided to see what the fuss with Linux was all about. I started looking into all those different distros and since I like the idea of customization, I decided to go with Gentoo. To cut a long story short, I didn't even manage to succesfully install it...

A lot of things have changed within the years, but things can still go wrong and that was my experience installing Kali Linux on a Raspberry Pi 3.

So, I decided to write a short guide about it.

I used the custom [Sticky Fingers Kali-Pi](https://whitedome.com.au/re4son/download/sticky-fingers-kali-pi/), which is supposed to make things easier if you're using a TFT screen with your Pi, although I'm not using one. I will address each issue I faced specifically and then provide the terminal commands you need to enter so, I trust you are at still a bit familiar with the Linux Terminal.

### Change the default password

First thing you'll want to do is change the default password.

```console
root@kali-pi:~# passwd
```

### Fix Screen Resolution & Mouse Lag

My first issue was mouse lag. If you are using a wired mouse, you won't experience it, but I am using a wireless one and that can be a problem with certain manufacturers.

My second issue was the screen resolution. My monitor was not detected by Kali Linux, so I ended up with a 640 x 480 resolutionn. While that was great in 1993, it doesn't look that great in 2018!

Raspberry Pi does not have a BIOS. Instead, all the system configuration paramaters are stored in a boot partition, more specifically in a text file named "config.txt". In order to fix certain issues you need to mount this boot partition, since it's not mounted by default.

```console
root@kali-pi:~# mount /dev/mmcblk0p1 /boot
root@kali-pi:~# cd /boot
```

Now that the partition is mounted, you can edit the necessary files.

#### Change Video Resolution

"Config.txt" does not exist by default, so you need to create it. 

```console
root@kali-pi:/boot# nano config.txt
```

You can see all the configuration parameters at the [RPiconfig page](https://elinux.org/RPiconfig). I used the ones below that gave me a 1920 x 1080 @ 60 Hz resolution. You just need to enter the parameters you require in the "config.txt" file. 

```
# Force the monitor to HDMI mode so that sound will be sent over HDMI cable
hdmi_drive=2
# Set monitor mode to CEA
hdmi_group=1
# Set CEA mode to mode 16. Resolution 1920 x 1080 @ 60 Hz
hdmi_mode=16
# Disable overscan
disable_overscan=1
```

Save the file by pressing Ctrl + C, selecting yes and exit.

##### Fix Mouse Lag

In order to fix the mouse lag, you need to edit the "cmdline.txt"

```
root@kali-pi:/boot# nano cmdline.txt
```

which is also in the same folder and add

```
usbhid.mousepoll=0
```

at the end of the long line.

Be careful to type it exactly like and just leave a space after the existing text.

Save the file and exit.

You could actually stop here and have a working Kali Linux installation, but feel free to read ahead if you want to setup services such as SSH and VNC and also install the full version of the distro.


### Generate new SSH keys and add the SSH server service on boot

Delete the old host keys.
```console
root@kali-pi:~# /bin/rm -v /etc/ssh/ssh_host_*
```

Regenerate the host keys.
```console
root@kali-pi:~# dpkg-reconfigure openssh-server
```

Restart the SSH server.
```console
root@kali-pi:~# systemctl restart ssh
```

Remove and add the SSH server service on boot.
```console
root@kali-pi:~# update-rc.d -f ssh remove
root@kali-pi:~# update-rc.d -f ssh defaults
```

Open the sshd_config file.
```console
root@kali-pi:~# nano /etc/ssh/sshd_config 
```

Make sure that 'PermitRootLogin' is set to yes, so that you can connect to SSH using the root acount.

Restart the SSH server to apply all changes.
```console
root@kali-pi:~# service ssh restart
```

The following command ensures that the SSH persistence survives a reboot.
```console
root@kali-pi:~# update-rc.d -f ssh enable 2 3 4 5
```

### Enable auto-login for root account on system start

If you don't want to login on boot everytime, you can run the following commands. This is especially useful if you're planning to use the Pi in a headless mode (without a screen connected).

```console
root@kali-pi:~# cd /usr/local/src/re4son-kernel_4*
```

```console
root@kali-pi:/usr/local/src/re4son-kernel_4.9.80-20180410# ./re4son-pi-tft-setup -a root
```

### Install x11vnc

Sometimes SSH is not enough and you need to use the GUI. That's where VNC comes in handy. The following installs a VNC server on the Raspberry Pi, but you also need to install VNC software on the device that you wish to view the Pi's interface from. I'm using [VNC Viewer](https://www.realvnc.com/en/connect/download/viewer/).

Install x11vnc.
```console
root@kali-pi:~# apt-get install x11vnc
```

Set a password for remote VNC connection.
```console
root@kali-pi:~# x11vnc -storepasswd
```

Enable the x11vnc service on boot.
```console
root@kali-pi:~# cd /usr/local/bin
root@kali-pi:/usr/local/bin# nano sharex11vnc
```

```bash
#!/bin/sh

x11vnc -ncache 10 -auth guess -repeat -nap -loop -forever -rfbauth ~/.vnc/passwd -desktop "VNC ${USER}@${HOSTNAME}"|grep -Eo "[0-9]{4}">~/.vnc/port.txt

# This tells you which port x11vnc is using
zenity --info --text="Your VNC port is 'cat~/.vnc/port.txt'"
```

What the flags mean:
-ncache: Caches the screen content for rapid retrieval that can reduce lag.
-auth guess: Used to guess the XAUTHORITY file for the display
-nap: Monitors activity and takes longer naps if it is low.
-forever: Keeps listening for more connections rather than exiting as soon as the first client disconnects.
-repeat: VNC clients are connected and VNC keyboard input is not idle for more than 5 minutes. This works around a repeating keystrokes bug (triggered by long processing delays between key down and key up client events: either from large screen changes or high latency).
-rfbauth passwd-file: Authenticates the password file
-rfbport port: TCP port for RFB protocol

Set permissions for the newly created file.
```console
root@kali-pi:~# chmod 755 /usr/local/bin/sharex11vnc
```

Enale autostart for script file.
Go to applications --> Settings --> Session and Startup.
Click on the Application Autostart tab.
Click on Add.
Type the name you want, a brief description and 'x11vnc' as the command.

### Install full version of Kali Linux

Finally, you might want to install the full version of Kali Linux in order to unleash its full power.

```console
root@kali-pi:~# apt-get update
root@kali-pi:~# apt-get install gparted
```

Run gparted and resize the '/' mount point so that it takes the whole size of your micro SD card.

```console
root@kali-pi:~# apt-get install kali-linux-full
root@kali-pi:~# apt-get upgrade
```

_And that should do it_. 

Now, you have a working installation of Kali Linux!