# No build APRS rx only iGate setup instructions

Start with base Raspbian install

Install prerequisites

```sh
apt install direwolf rtl-sdr -y
```

---

Edit `~/igate.conf`

```apache
# Set the sound card to null
ADEVICE null null
CHANNEL 0

# Put your callsign here.  It is common to add "-10" to the end.
MYCALL xxxxx-1
LOGDIR /var/log/direwolf

# Enter your callsign (no "-10" this time) and your passcode.
# If you don't have one, get your passcode at http://apps.magicbug.co.uk/passcode/
IGLOGIN xxxxx 00000

# The IGSERVER defaults to North America.
# If you prefer a different server, use one of these:
#      noam.aprs2.net - North America
#      soam.aprs2.net - South America
#      euro.aprs2.net - Europe and Africa
#      asia.aprs2.net - Asia
#      aunz.aprs2.net - Oceania
IGSERVER noam.aprs2.net

# Set your iGate up to beacon itself every 60 minutes.
# IMPORTANT: Change your latitude/longitude to your own coordinates.
PBEACON sendto=IG delay=2:13 every=13:00 symbol="R&" lat=37.227578500 long=-121.875832667 altitude=70 height=30 gain=2.2 COMMENT="RX Only igate, RPi2+DireWolf+RTLSDR (Discone)"
```

---

Edit `igate.sh`
```bash
#!/bin/bash
export DISPLAY=":0"
DWCMD="direwolf -c /home/lee/igate.conf -r 24000 -D 1 -T \"Timestamp: %F %T\""
LXCMD="bash -c 'rtl_fm -f 144.39M - | $DWCMD -'"
sleep 30
/usr/bin/lxterminal -t "iGate" -e "$LXCMD"
```

---

Edit `~/.config/autostart/igate.desktop`
```ini
[Desktop Entry]
Type=Application
Name=iGate
Exec=sh -c "sudo /home/lee/igate.sh"
```
 ---

 Reboot and log into iGate

---

## Notes:

Instructions based on the following guides
* [CREATE A RECEIVE-ONLY APRS IGATE ON A RASPBERRY PI](https://www.regnatarajan.com/radio/create-aprs-receive-only-igate-raspberry-pi)
