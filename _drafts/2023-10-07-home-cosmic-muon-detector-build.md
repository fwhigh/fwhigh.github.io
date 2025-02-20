---
title: "Home Cosmic Muon Detector Build"
date: 2023-10-07 12:00:00 -0700
comments: true
author: "Will High"
header:
  overlay_color: "#000"
categories: 
  - Hardware
  - Featured
  - Raspberry Pi
excerpt: Home cosmic muon detector build
toc: true
toc_sticky: true
author_profile: true
classes: wide
---

# Overview

I wanted to build a muon detector out of two 
[Mightyohm Geiger counters](https://mightyohm.com/blog/products/geiger-counter/)
AND-ed together 

I wanted to refamiliarize myself with a bunch of things
I picked up as a physics student: soldering, electronic circuits,
breadboarding, hardware/IoT control. 



# Materials

* (2x) [Mightyohm Geiger counters](https://mightyohm.com/blog/products/geiger-counter/)
* Soldering kit to assemble the Geiger counters. I like Plusivo's
* Raspberry Pi 4. I like the Canakit because it comes with housing, an SD card USB dongle, and a bunch of other essentials
* Breadboard
* Buncha breadboard jumper wires, especially male to female
* (2x) 2N2222 NPN transistors. I bet 2N3904 or some kinda PNP transistors will work fine, but I didn't test them
* (2x) 2k Ohm resistors
* 100 Ohm resistor
* (2x) Tactile or other breadboard switches for testing
* Red LED
* A GPIO to breadboard adapter. I like the [SparkFun Pi Wedge](https://www.sparkfun.com/products/13717)
* A computer to log into your headless RPi4

# Assemble the Geiger counters

There are a million online resources and kits for learning to solder. 
I did a bunch of SparkFun and Amazon learn-to-solder kits to get used to it.
My kids got some neat toys out of it.

# Power them from your breadboard

You can power the Geiger counter board with the batteries removed 
using the PULSE header at 3.3V. 
Note the PULSE header on the Geiger counter board are not labeled properly.
See my [previous post]({% post_url 2023-09-22-mightyohm-geiger-rpi4 %}) 
to get it all sorted out (tl;dr GND and VCC, pins 1 and 3, are mislabeled and 
need to be flipped).

Mightohm does not recommend powering the Geiger counter boards from the RPi GPIO
3.3V supply like I have done because of max current concerns. 
But I did it anyway and I 
have not yet fried my teensy puter.

That said I will probably switch to a proper breadboard power supply like the
[SparkFun Breadboard Power Supply 5V/3.3V](https://www.sparkfun.com/products/114).

# Assemble an AND gate on the breadboard

I followed [AND Gate 2 schematic from GSN](https://www.gsnetwork.com/and-gate/)
with some transistors and it worked fine. I added switches after
the resistors and before the transistor bases for manual testing.
I also switched the 2k Ohm diode with a 100 Ohm to increase brightness
for the Geiger counters' super short pulses.
I powered the AND gate at 5 V using the RPi.

The reason I built an AND gate from transistors myself was to (re)learn
circuits, transistors, and breadboarding, but I 
probably could have used an AND gate integrated circuit like 7408.

# AND the Geiger counters

Connect one Geiger counter to the first AND input (transistor base, before the 2k Ohm resistor)
and the second Geiger counter to the second AND input.
The breadboard LED should light up when both Geiger counters fire simultaneously.

# Signal and noise

The only way I'm aware two Geiger counters can fire at the same instant are
* coincidence (any combination of 2 different cosmic rays, beta decays, gamma rays) 
* a cosmic muon passing through both Geiger-Muller tubes at nearly the speed of light
* a gamma ray passing through both Geiger-Muller tubes at the speed of light

MIT says cosmic ray muons are > 20x more energetic than gamma rays at 20 MeV vs < 1 MeV.
I don't have control over detecting 20 MeV vs < 1 MeV with this kit that I'm 
aware of, so I'm not sure I can get rid of gamma ray signals.

# Learn more

* [Special relativity in the school laboratory: a simple
apparatus for cosmic-ray muon detection](https://iopscience.iop.org/article/10.1088/0031-9120/50/3/317), P Singh and H Hedgeland 2015 Phys. Educ. 50 317
* UC Berkeley Physics 111B [Advanced Experimentation Laboratory](https://experimentationlab.berkeley.edu/syllabus) experiment on [muons](http://experimentationlab.berkeley.edu/sites/default/files/writeups/MNO.pdf)
* [MIT physics undergrad lab course](https://ocw.mit.edu/courses/8-13-14-experimental-physics-i-ii-junior-lab-fall-2016-spring-2017/pages/experiments/the-speed-and-mean-life-of-cosmic-ray-muons/) on [muons](https://ocw.mit.edu/courses/8-13-14-experimental-physics-i-ii-junior-lab-fall-2016-spring-2017/dba397e119acfe92807caeb1509c401e_MIT8_13-14F16-S17exp14.pdf)
