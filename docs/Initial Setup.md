# Initial Setup
> Using my old ThinkPad, flashing it with a new operating system, turning it into a server and optimizing it for server usage.

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
One of my main concerns was keeping the ThinkPad in good condition, and preventing any battery issues. Luckily, using TLP, you can keep the battery limited to a certain charge range.

First, I had to install TLP
```bash
sudo apt update
sudo apt install tlp tlp-rdw
```

Then, I had to configure TLP using this `.conf` file.
```bash
sudo vim /etc/tlp.conf
```

I had to set these following lines:
```conf
START_CHARGE_THRESH_BAT0=40
START_CHARGE_THRESH_BAT0=60
```
To allow the battery to start charging at 40%, and stop charging at 60% to keep the battery operating at the most healthy ranges.
> Also, the ThinkPad T480s only runs on one battery, so I only have to do this for `BAT0`.

Then, I had to make sure I installed the required drivers to communicate with the ThinkPad battery:
```bash
sudo apt update
sudo apt install tp-smapi-dkms acpi-call-dkms
```

Then, I could start TLP and set the charge thresholds.
```bash
sudo tlp start
sudo tlp setcharge 40 60 BAT0
```

The last step of implementing this change was to actually let the battery drain to below 40% to begin using my custom charging settings.

I verified that my settings were in effect by running:
```bash
sudo tlp-stat -b
```

Finally, the server was ready for [Docker](Docker.md) installation.
