---
title: "The Mightyohm Geiger Counter on Raspberry Pi 4"
date: 2023-09-22 12:00:00 -0700
comments: true
author: "Will High"
header:
  overlay_color: "#000"
categories: 
  - Hardware
  - Featured
  - Raspberry Pi
excerpt: I slogged my way to a working Mightyohm Geiger Counter integration with a Raspberry Pi 4.
toc: true
toc_sticky: true
author_profile: true
classes: wide
---

# tl;dr

* The [Mightyohm Geiger counter PCB](https://mightyohm.com/blog/products/geiger-counter/) has a big old bug:
  * the J6 Pulse header has ground (black triangle) marked incorrectly on the PCB
  * the true pinout is the opposite direction indicated
    * pin 1 with the black triangle is actually PULSE 
    * pin 3 at the corner is actually GND = ground
  * the [schematic diagram](https://mightyohm.com/files/geiger/geiger_sch_fixedR5R6.png) is also misleadingly labeled with ground on the bottom at pin 1
  * I discovered all this when my rpi4 wouldn't turn on and I started verifying connections with a multimeter and also by reading [this comment](https://mightyohm.com/forum/viewtopic.php?p=10237&sid=830752b96a53c17de5f5e2461ea91b48#p10237), which turned out to be right
* The serial port readout is `/dev/ttyS0` for the rpi4. It's `/dev/ttyAMA0` for other rpi flavors
* All the other instructions I found online are correct. I will summarize below
* Some multimeter meter fiddling leads me to believe the J5 ICSP header is also incorrectly labeled on the PCB, though I am not using this header for anything. J5 ground is on the bottom left, not the top right, when the the PCB is oriented with the headers facing down. The true location of ground seems to match the schematic

# Intro

Everything about the 
[Banana Random Number Generator](https://www.hackster.io/news/the-just-bananas-method-for-generating-true-random-numbers-0e67c763dc1a)
resonnates with me. It has physics,  
it has computer science,
it is DIY, and it is ridiculous. I've wanted to build it but
kids and life have prevented me from having hobbies...

Until now! N years later I've decided to try to 
do some Raspberry Pi'ing with a couple [home DNS servers](https://pi-hole.net/)
and separately re-learn soldering.
I came across the [Mightyohm Geiger counter](https://mightyohm.com/blog/products/geiger-counter/)
and the planets and stars came into alignment.

I cobbled together some 5-10 year old crumbs off the internet
and after an hour or so I finally got a readout from the Geiger counter.

Here's the recipe.

# Materials

1. [Mightyohm Geiger counter](https://mightyohm.com/blog/products/geiger-counter/)
1. Soldering equipment for the Geiger counter
1. A Raspberry Pi 4
    1. I love the [Canakit starter kit](https://www.canakit.com/raspberry-pi-4-starter-kit.html) because
it comes with a case, fan, heat sinks, micro SD card, micro SD card reader USB dongle for burning new OS images, power supply, and a power switch
    1. Don't forget a mouse, keyboard, and optional HDMI monitor (unless you're going headless). Get an RPi 400 to get mouse and keyboard bundled with the rpi4
    1. I have used and liked Amazon and [Adafruit](https://www.adafruit.com/) for rpi supplies
1. Breadboarding jumper wires, especially female to female. I liked [these](https://www.amazon.com/EDGELEC-Breadboard-Optional-Assorted-Multicolored/dp/B07GD2BWPY/).

# Procedure

## Assemble the Geiger counter

It's fun and easy. I accidentally soldered my integrated circuit chips directly to the board! 
But it still worked. That ended up being the easiest thing to solder besides the header pins.

## Connect the Geiger counter to the Raspberry Pi 4 GPIO

1. Turn off the Raspberry Pi and unplug it.
1. Turn off the Geiger counter and take out the batteries.
1. Connect
    1. J6 PULSE pin 1 (incorrectly marked with the black triangle) to GPIO pin 1 3.3V
    1. J6 PULSE pin 3 to GPIO pin 14 GND
    1. J2 SERIAL pin 1 (correctly marked with the black triangle) to GPIO pin 6 GND
    1. J2 SERIAL pin 4 to GPIO pin 8 UART TXD (transmit)
    1. J2 SERIAL pin 5 to GPIO pin 10 UART RXD (receive)

<figure>
    <a href="/assets/mightyohm-geiger-rpi4/pinout.png"><img width="70%" src="/assets/mightyohm-geiger-rpi4/pinout.png" /></a>
    <figcaption width="70%">Pinout photo between Mightohm Geiger counter and Raspberry Pi 4. Rpi4 pin 1 is the bottom left pin on the black ribbon cable. Red to GPIO 1, brown to GPIO 6, orange to GPIO 8, yellow to GPIO 10, and
    black to GPIO 14.</figcaption>
</figure>

Credit to [this thread](https://mightyohm.com/forum/viewtopic.php?t=4524)
and [this post](https://www.instructables.com/Portable-Raspberry-Pi-Geiger-Counter-With-Display/) 
for getting the pinout mostly right. 
Only one commenter on the first thread hinted at the incorrect PCB specification. 
The second post didn't say anything about it (is this bug new?).

## Turn on the Raspberry Pi 4 and read the data

### Connect via VS Code

I like to connect to my rpi's via VS Code.

1. Add your rpi to `~/.ssh/config`
1. Command-shift-p then "remote-ssh: connect to host" by name specified in your ssh config
1. Enter password
1. Command-shift-p then "view: toggle terminal" for a command shell
1. Create an ssh key on your local computer with `cd ~/.ssh && ssg-keygen`
1. Copy your public key to your rpi

With the last 2 steps you don't need to enter a password.
My typical ssh config entry:

{% include gist_embed.html data_gist_id="fwhigh/92a985dd8c494949a36433641c14e2e6" data_gist_file="ssh-config" %}

### Enable serial comms (the UART pins)

[This article was](https://pimylifeup.com/raspberry-pi-serial/) helpful. 

1. Log into your rpi.
1. On your rpi, execute `sudo raspi-config` in a shell.
1. Select "Interfacing Options".
1. Select "Serial"
1. Set "login shell to be accessible over serial" to NO
1. Set "make use of the Serial Port Hardware" to YES
1. Exit the config and reboot (you will be prompted to reboot, otherwise do `sudo reboot`)

### Read the data

[This entry](https://www.instructables.com/Portable-Raspberry-Pi-Geiger-Counter-With-Display/) 
was great but not quite correct for rpi4.
Initializing `serial.Serial` to point to `/dev/ttyS0` instead of `/dev/ttyAMA0` works.

{% include gist_embed.html data_gist_id="fwhigh/92a985dd8c494949a36433641c14e2e6" data_gist_file="geiger.py" %}
