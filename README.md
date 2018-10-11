The On Stage Lyrics Browser is an attempt to build a simple teleprompt
system for on-stage performances. 

The collection of scripts here, properly installed on a Raspberry Pi, 
will produce a simple, text-based browser for lyrics on a USB device.

[Setup](#to-install) is fairly straightforward, and [preparing the files/USB flash
drive](#to-use) are simple. Interfacing [pedals](#hardware) with the Raspberry PI takes a bit
of work to operate properly, but can be easily achieved.

<p align="center">
  <img src="./img/set-list.png" alt="Set List Example"
       width="540" height="694">
</p>

<p align="center">
  <img src="./img/single-page-lyrics.png" alt="Lyric View Example"
       width="540" height="694">
</p>

 
## Included:
 * a udev rule to auto mount/umount USB upon insertion/removal
 * a script that controls the interface display functions
 * a script that sends mount information to the interface
 * a script that sends umount information to the interface

## To install:
This assumes a recent copy of Raspian is installed and everything is
default.

#### Configure networking
Connect your PI to a network and be sure you can access APT repositories.
This does not need to persist beyond initial setup.

#### Install dependencies
```sh
$ sudo apt install -y python3-rpi.gpio git
```

#### Install scripts
```sh
$ cd ~
$ git clone https://github.com/joshuabettigole/on-stage-lyrics-browser.git
$ chmod +x on-stage-lyrics-browser/bin/*
```

#### Rotate screen
This works best with the screen orientation rotated 90° or 270°
```sh
$ sudo nano /boot/config.txt
```
Add or edit the following:
```nano
display_rotate=1
```

#### Symlink udev rules
```sh
$ sudo ln -s ~/on-stage-lyrics-browser/etc/udev/rules.d/* /etc/udev/rules.d/
```

#### Setup autorun
Add the following to ~/.profile
```nano
setfont /usr/share/consolefonts/Lat15-TerminusBold32x16.psf.gz
LDIR=$(find /media -maxdepth 2 -type d -name 'lyrics' -print0 -quit 2> /dev/null)

if [ -d $LDIR ]; then
        echo -n $LDIR | ~/on-stage-lyrics-browser/bin/lyricsbrowser.py
else
        ~/on-stage-lyrics-browser/bin/lyricsbrowser.py
fi
```
This sets the font size to something reasonably large - feel free to modify.
It also looks to see if a USB was present and mounted during boot, checks
for a lyrics directory, and passes that directory along to start the 
interface script upon login.

#### Setup autologin, disable network on boot
```sh
$ sudo raspi-config\
```

To boot directly to the lyrics browser, enable autologin
> Boot Options -> Desktop/CLI -> Console Autologin

To save time on bootup, disable networking
> Boot Options -> Wait for Network at Boot -> No

#### Restart
Reboot your Pi and if all worked, you should see the lyrics browser
after a few moments.

## To use:
#### USB Formatting
* The USB flash drive you use should be FAT formatted, with or without
partitioning.
* On the flash drive, you should have a <strong>lyrics</strong>
directory (all lowercase)
* Within the <strong>lyrics</strong> directory, the lyrics files need to
be placed in alphabetical order. The easiest way to do this is to number
them.
```sh
pi@raspberry:/media/usbhd-sda $ ls -l lyrics
total 64
-rwxrwxr-x 1 root users  1679 Oct  3  2018 01 - The Song That Never Ends.txt
-rwxrwxr-x 1 root users   869 Oct 11  2018 02 - Mary Had A Little Lamb.txt
-rwxrwxr-x 1 root users 11886 Oct 11  2018 03 - 99 Bottles of Beer.txt
-rwxrwxr-x 1 root users   735 Oct 11  2018 04 - Never Gonna Give You Up.txt
-rwxrwxr-x 1 root users   414 Oct 11  2018 05 - Spongebob Theme Song.txt
```

#### File Formatting
Files must be in .txt format (.rtf and .doc do not work). At
some point in the future, support for adding colors will be considered.

If lyric lines are too long to fit within the width of the page, line
breaks will be added and additional lines will be prefixed with " - ".

If lyrics for one song are too long to fit on one page, subsequent pages
will be added, as necessary, for browsing. The lower right corner of the
screen will display <strong>"Next Page ->"</strong> and the upper right 
corner displays the current page number: <strong>"Page #/##"</strong>
<p align="center">
  <img src="./img/multi-page-lyrics.png" alt="Multi-Page Example"
       width="540" height="694">
</p>

#### Starting
Upon bootup, if a USB device is present, it should be mounted and the
lyrics directory path (if it exists) passed to the browser script. 

If no USB device is present at boot, the browser should alert you to 
<strong>Insert USB Media</strong>. When a USB device is inserted, the 
browser will be notified and will automatically display the set list.

A USB device can be removed and reconnected at any point. The browser 
will be notified and act accordingly.

#### Browsing
The browser depends on GPIO interrupts for navigation. There is no 
facility for keyboard browsing. Three buttons/pedals are necessary to
use the full features of the program, <strong>Previous</strong>, 
<strong>Menu/Select</strong>, and <strong>Next</strong>. The 
<strong>Previous</strong> button/pedal can be omitted at the cost of 
ease of navigation. See the [Hardware](#hardware) section for
details.

#### Localization
Interface language is in English (sorry, that's all I know). If you 
care to modify the language for your locale, modify the strings in the 
_e dictionary within the bin/lyricsbrowser.py file:
```python
# Setup language strings
_e = {
    'welcome': "Welcome",
    'load_media': "Insert USB Media",
    'select': "Select",
    'menu': "Menu",
    'next': "Next",
    'prev': "Prev",
    'first': "First",
    'last': "Last",
    'prev_page': "Prev Page",
    'prev_song': "Prev Song",
    'next_page': "Next Page",
    'next_song': "Next Song",
    'page': "Page"
}
```

## Hardware
#### Requirements:
A properly configured Raspberry Pi attached to a widescreen monitor is
a start. A keyboard and network connection is required to get past the
installation process, but then should not be required at all.

A three button/pedal box of your own choosing. Buttons/pedals should be 
normally open (normally closed untested).

While some attempts are made in code to handle switch debouncing, there
is still risk that one button cycle can advance multiple pages. It is 
<strong>HIGHLY</strong> recommend that a simple debounce circuit be 
added between the PI and the pedals/buttons. The following schematic
outlines a fairly effective debounce circuit:

<p align="center">
  <img src="./img/debounce_schematic.png" alt="Debounce Schematic"
       width="600" height="341">
</p>

[See Schematic](./img/debounce_schematic.png)

Without a hardware debounce circuit, the pedals/buttons should be
connected to:

* Previous: GPIO 13
* Menu: GPIO 19
* Next: GPIO 26

(these are pins 33,35,37)

The exact GPIO pins can be altered in the bin/lyricsbrowser.py file
```python
gpiopin_prev = 13
gpiopin_next = 26
gpiopin_menu = 19
```

## TODO:
* Work on a way to format text with color/bold. There is still no plan
to use anything but .txt files, however.
* Build a shell script to simplify install.
* Preconfigured Raspberry PI OS image
* Provide a hardware list
* Offer pre-built debounce interface

## License:
The On Stage Lyrics Browser is licensed under the terms of the GNU Affero
General Public License v3.0 and is available for free. See the
[License](./LICENSE) for details.

## Credits:
udev rules adapted from:\
https://www.axllent.org/docs/view/auto-mounting-usb-storage/

switch debounce circuit from:\
https://www.logiswitch.net/switch-debounce-diy_tutorial/method-4-hardware-debounce-for-spst-switches
