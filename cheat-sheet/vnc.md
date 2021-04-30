## VNC

## Install

```bash
sudo apt update

# install tigerVNC (RECOMMENDED)
sudo apt install -y xserver-xorg-core tigervnc-standalone-server tigervnc-xorg-extension tigervnc-viewer
# or install tightVnc
sudo apt install -y vnc4server

# set vnc password
vncpasswd
```



## Install GUI

xfce4

```bash
sudo apt install -y xfce4
```

or gnome

```bash
sudo apt install ubuntu-gnome-desktop
sudo systemctl start gdm
sudo systemctl enable gdm
```



## Configure xstartup

```bash
# create xstartup file if not exists
touch ~/.vnc/xstartup && chmod +x ~/.vnc/xstartup
# modify
vim ~/.vnc/xstartup
```

use xfce4

```bash
#!/bin/sh
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
startxfce4 &
```

use gnome

```bash
#!/bin/sh
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
vncconfig -iconic &
dbus-launch --exit-with-session gnome-session &
```

if not work, try this version:

```bash
#!/bin/sh

# Uncomment the following two lines for normal desktop:
#unset SESSION_MANAGER
# exec /etc/X11/xinit/xinitrc

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
#x-terminal-emulator -geometry 1280x930 -ls -title "$VNCDESKTOP Desktop" &
x-window-manager &

# Fix to make GNOME work
export XKL_XMODMAP_DISABLE=1
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
gnome-session --session=ubuntu -geometry 1280x930 -ls -title "$VNCDESKTOP Desktop" &

gnome-panel &
gnome-settings-daemon &
metacity &
nautilus &
gnome-terminal &
```



## Operations

```bash
# start vnc screen x, use PORT 590x by default
vncserver
# or
# specify pixels and depth, and listen 0.0.0.0 instead of 127.0.0.1 only
vncserver -geometry 1366x768 -depth 24 -localhost no :1

# stop vnc screen x
vncserver -kill :1
vncserver -kill :*

# See if VNC Server is active
netstat -anp | grep vnc
# or
ss -tulpn | egrep -i 'vnc|590'

# see who is connected
vncserver -list
```



## Viewer

Use the responded viewer from client side.



## Setup Service

```bash
sudo vim /etc/systemd/system/vncservice@.service
```

```bash
[Unit]
Description=Start TightVNC server at startup
After=syslog.target network.target

[Service]
Type=forking
User=jzj
Group=jzj
WorkingDirectory=/home/USERNAME

PIDFile=/home/USERNAME/.vnc/%H:%i.pid
ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/bin/vncserver -depth 24 -geometry 1366x768 :%i
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable vncserver@1
sudo systemctl start vncserver@1
```



## BUG for tigerVNC

Cannot login into account, run command in ssh to unlock.

```bash
loginctl unlock-session
```

https://askubuntu.com/questions/1224957/i-cannot-log-in-a-vnc-session-after-the-screen-locks-authentification-error



## References

https://www.teknotut.com/en/install-vnc-server-with-gnome-display-on-ubuntu-18-04/

https://www.cyberciti.biz/faq/install-and-configure-tigervnc-server-on-ubuntu-18-04/

