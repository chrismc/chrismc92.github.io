---
layout: post
title: "Hexadecisonic: First Steps"
quote: "The beginning of my journey to replicate a tiny noise box"
image: /media/2015-01-31-Hexadecisonic-First-Steps/SoundMachineHeader.jpg
video: false
comments: true
dark: false
---
[Patrick McCarthy](http://www.rothmobot.com/), one of my colleagues at the [Fab Lab](http://www.msichicago.org/whats-here/fab-lab/), has a small sound effects box that he uses when leading workshops. 

{% include image.html url="/media/2015-01-31-Hexadecisonic-First-Steps/SoundMachine.jpg" width="50%" description="It's quite stylish" %} It has a lot of noises and effects and it's a pretty cool way to help keep presentations lively and keep people engaged. A lot of the sounds aren't particularly useful though (gunshots and excessively lengthy screaming come to mind), so I decided to try my hand at making a customizable one. I dubbed my version the Hexadecisonic, since I wanted it to have 16 sounds like the box it's based on.

For my version of the sound board, I chose to make use of an ATmega328P as the main controller. Since the Fab Lab is a heavy consumer of Arduino-based projects, we have a lot of these chips on hand. Additionally, using the Arduino environment helps make programming a fairly painless process, allowing me to focus on the hardware. 

The ATmega certainly doesn't have the space to store the 16 sounds I need, so files have to be read off an external SD card. I didn't want to have to order anything special for this project, so rather than using an SD shield, I chose to access the card by directly wiring it to an Arduino using instructions found in [this Instructables project](http://www.instructables.com/id/Cheap-DIY-SD-card-breadboard-socket/?ALLSTEPS). For similar reasons, I chose to use the surface mount momentary switches that are part of [every Fab Lab's standard inventory](https://docs.google.com/spreadsheet/pub?key=0AtIlZyLn99e6dGRleUJTY043a3FucUhFUVVBYTdxS3c&single=true&gid=0&output=html), and a voltage divider rather than a regulator to supply the 3.3V that the SD card requires. 

I dumped all the parts into EagleCAD and came up with this schematic:

{% include image.html url="/media/2015-01-31-Hexadecisonic-First-Steps/HexaDeciSonicSchematic1.png" width="50%" description="Hexadecisonic Schematic V1.0 <a href='/media/2015-01-31-Hexadecisonic-First-Steps/HexaDeciSonicSchematic1.pdf'>Full PDF</a>" %}

For the most part I was able to use Eagle parts from the Fab Lab libraries. Using the same technique found in the Instructable, I used a set of header pins to connect to the SD card to the MCU. I used a solder pad component in order to connect the power supply and speaker. In future versions, I may replace these with components that actually mount to the board.

The bulk of my time was definitely spent routing the board. Board layout isn't my strong suit, and the Fab Lab process for milling boards makes double sided boards a bit difficult for me to manufacture, meaning I was restricted to using only the top layer. 

After a few hours of effort, I finally had a semi-viable board:

{% include image.html url="/media/2015-01-31-Hexadecisonic-First-Steps/HexaDeciSonicBoard1.png" width="50%" description="Hexadecisonic Board V1.0" %}

I learned a lot during the layout, so it was definitely worthwhile. However, the clearances on some traces (particularly those near the microcontroller) are rather tight, so I may have to revisit board layout and figure out how to mill two sided boards after all. I also FINALLY understand and FULLY APPRECIATE why our single-pole switches have four pads instead of two.

Next I have to take a look at setting this board up to be made with the Fab Lab's Roland Modela mini-mill. It's something I've tried my hand at in the past, but I've never been 100% successful, so fingers crossed, this is the time I finally figure it out. 

Stay tuned!
