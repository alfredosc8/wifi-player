# wifi-player
A simple way to turn your embedded linux device into a internet radio player

# how it works
Well basically you bind a key on a numeric keypad to a certain streaming server URL and once a key is pressed the stream starts playing.

# hardware part of the project

* Any device with USB support running on linux. I use D-Link DIR-320 revB router flashed with OpenWRT's stabled version of 14.07 Barrier Breaker
* USB hub, any unpowered one will do just fine
* A simple audio adapter
* Numeric keypad. Even though you might use regular keyboard just as well I prefer keypad for its lightweight size and esthetic value
* Set of speakers, obviously ;-)

# software part of the project

Simply put wifi-player user **madplay** to play the streams, **wget** to pipe the stream into **madplay** via stdout and **triggerhappy** for binding keys to launch/stop the player. I'm putting all the scripts in /root since there's only one user in OpenWRT and it's root. You might find your homedir a better place depending on the platform you decide to use.

### let's install some stuff on the device

Some of the software is probably already installed on OpenWRT 14.07 by default, but just in case let's do this:

```
opkg install kmod-usb-hid kmod-usb-core kmod-ledtrig-usbdev kmod-hid kmod-hid-generic
opkg install kmod-sound-core kmod-usb-audio madplay usbutils
opkg install kmod-sound-cs5535audio
opkg install alsa-lib alsa-utils
opkg install triggerhappy
opkg install kmod-sound-i8x0 kmod-sound-soc-core usb-modeswitch
opkg install kmod-usb-uhci
opkg install kmod-button-hotplug
opkg install kmod-gpio-button-hotplug
opkg install kmod-usb-ohci
opkg install kmod-usb2
```

### binding keypad to the stations (/etc/triggerhappy/trigger.d/example.conf)
```
KEY_KP0                 0       /root/play-station1
KEY_KP1                 0       /root/play-station2
# <...>
KEY_KP9                 0       /root/play-station9
KEY_ENTER               0       /root/kill_radio
```
You can find out your keycodes by typing this in terminal: *thd --dump /dev/input/event**

### the script responsible for turning off the player (/root/kill_radio)
```
kill -9 `cat /tmp/madplay.pid`
killall madplay
```

### starting the stream of the first station (/root/play-station1)
```
/root/kill_radio
sleep 1
wget -q -O - http://address.of.your.station/128.mp3 | madplay - & echo $! > /tmp/madplay.pid
```

# few notes
This D-Link router doesn't have enough flash memory to contain bigger software players, namely mpd/mpd. Using mpd you will find yourself not restricted to only mp3 streams anymore as mpd supports aac/ogg/wma too.

And finally, turning the device into really a wireless player is as simple as configuring wireless client mode in OpenWRT. So make sure you have AP running somewhere in your house.
