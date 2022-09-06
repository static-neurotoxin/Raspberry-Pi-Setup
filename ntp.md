# No build NTP setup instructions

Start with base Raspbian install

---

## setup GPS hat (Adafruit ultimate GPS HAT)

Edit `/boot/config.txt`
`sudo emacs /boot/config.txt`
add the following line
`dtoverlay=pps-gpio,gpiopin=4`

---

edit `/etc/modules`
`sudo nano /etc/modules`
add the following line:
`pps-gpio`

---

## setup GPS hat (Uputronics Raspberry Pi GPS/RTC Expansion Board)

`sudo raspi-config`
Select Interfacing Options and Enable I2C

---

`sudo apt-get install python-smbus i2c-tools`
`sudo i2cdetect -y 1`

```
.    0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --  
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- 42 -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- 52 -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

Here you can see the RTC (0x52) and GPS (0x42) on the I2C bus.

---

`sudo emacs /boot/config.txt`
Add a line with:
`dtoverlay=i2c-rtc,rv3028`

`sudo reboot`

---

On reboot rerun the i2cdetect line and you should see 52 has been replaced with UU to indicate the kernel driver is loaded.

`sudo apt-get -y remove fake-hwclock`
`sudo update-rc.d -f fake-hwclock remove`
`sudo systemctl disable fake-hwclock`
`sudo emacs /lib/udev/hwclock-set`

Comment out:
    #if [ -e /run/systemd/system ] ; then
    # exit 0
    #fi
    # /sbin/hwclock --rtc=$dev --systz --badyear

To read time:
`sudo hwclock -v -r`

To write time:
`sudo hwclock -w`

sudo nano `/boot/config.txt`
`dtoverlay=pps-gpio`

edit `/etc/modules`
`sudo nano /etc/modules`
add the following line:
`pps-gpio`

## setup pps, gpsd and chrony
Install prerequisites
`sudo apt install chrony gpsd gpsd-clients pps-tools -y`

---

Disable serial login but keep serial port enabled
`sudo raspi-config`
* choose “Interface Options”
* choose “Serial Port”
* Select “No” to disable the login shell over serial port and then “Yes” to keep the serial port hardware enabled

---

Verify GPS is working properly
`sudo stty -F /dev/serial0 raw 9600 cs8 clocal -cstopb`
`cat /dev/serial0`
`sudo ppstest /dev/pps0`

---

Edit `/etc/default/gpsd`
`sudo emacs /etc/default/gpsd`

```ini
DEVICES="/dev/serial0 /dev/pps0"
GPSD_OPTIONS="-n"
```

`sudo systemctl restart gpsd.service`
`sudo systemctl enable gpsd.service`

---

Run `gpsmon` or `cgps -s` to connect to GPSD and display the GPS information
`gpsmon`
`cgps -s`

`sudo ntpshmmon`

---

Edit `/etc/chrony/chrony.conf`
`sudo emacs /etc/chrony/chrony.conf`

```apache
refclock SHM 0 refid GPS precision 1e-1 delay 0.1
refclock SHM 2 refid PPS precision 1e-7
```

`sudo systemctl restart chrony.service`

---

Enable serving time
Edit `/etc/chrony/chrony.conf`
`sudo emacs /etc/chrony/chrony.conf`

```apache
allow 192.168.0.0/24
# If you also have IPv6 on the subnet.
allow 2600:8394:a947:bf::/64
```

`sudo systemctl restart chrony.service`
