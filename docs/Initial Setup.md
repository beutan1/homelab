# Initial Setup


## Ubuntu Server
I started by downloading Ubuntu 24.04.4 LTS and flashing it to my thumb drive, to install onto my Thinkpad.

After booting into safe mode, I had to configure the install:

### Installation process
After plugging the Ethernet cable and installation media into the laptop and booting into safe mode:

1. Left proxy address blank
2. named the server `thinkpad-ubuntu`
3. installed openssh server to SSH into laptop and to use `scp` down the line for media transfer

> I then tried to set up a wifi connection, when in reality, all I needed was an ethernet connection. so, I didn't need to `sudo vim 50-cloud-init.yaml`

## Font size change for Thinkpad screen
run `sudo dpkg-reconfigure console-setup` and select 16x32.

## TLP Battery Management for Server Longevity
ADD THINGS HERE ABOUT HOW YOU CONFIGURED TLP

And then after this, the server was ready for [[Docker]] installation.