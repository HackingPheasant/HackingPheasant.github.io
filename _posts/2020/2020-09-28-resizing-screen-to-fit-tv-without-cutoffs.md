---
layout: post
author: HackingPheasant
title: Linux - Resizing screen to fit TV, without cutoffs! 
categories: [guide]
tags: [linux]
---

This isn't a proper blog post or walk-through, its just a quick brain dump so I can document the future process for myself, lest I do [this](https://xkcd.com/979/) to myself.

Aim is to resize my screen to that the edges of the image aren't cut off by my TV's bezel, below is some *real quick and rough* documentation of how I did it
This will require a lot of trial and error

```bash
xrandr -q #To query curent info (--query also works)
xrandr -s <num> #To change/test current options (use to figure out your wanted screensize) or
xrandr --output <connected monitor name> --mode <name of mode> # same as above line, substitue required info with info from the query statement
# Example usage of above two lines
xrandr -s 2 #To select the second option or it can be done like so
xrandr --output HDMI-3 --mode 1280x720
```

So after a bit of testing I landed on 1200x768 being the correct size for my screen, with nothing being cut off.
To add this new size/mode into xrandr we are gonna want to rely on a tool to give us the correct info, for this example we will use `gtf`

```bash
gtf 1200 768 60 # gtf <width> <height> <refresh rate>
  
  # 1200x768 @ 60.00 Hz (GTF) hsync: 47.70 kHz; pclk: 74.79 MHz
  Modeline "1200x768_60.00"  74.79  1200 1256 1384 1568  768 769 772 795  -HSync +Vsync
```
We are going to want to copy everything after the word `Modeline` and we will use that info to add a new mode into xrandr
```bash
xrandr --newmode "1200x768_60.00"  74.79  1200 1256 1384 1568  768 769 772 795  -HSync +Vsync
xrandr --addmode HDMI-3 "1200x768_60.00"
xrandr --output HDMI-3 --mode 1200x768_60.00
```
Line by line, here is what the above does:
- line 1 creates a new mode with the info from gtf
- line 2 makes it available for our wanted output, which in this case is HDMI-3, again reference xrandr --query to see the info corresponding to your screen name
- line 3 switches the mode on HDMI-3 to our new mode


Now if we want to scale it to a resolution of 1920x1080 (For example) we can do it by so:
```
1920/1200 = 1.6
1080/768 = 1.40625
```
Replace the appropriate values with your own, in both above maths and in the following commands.

Now to scale it we can do:
```bash
xrandr --output HDMI-3 --mode 1200x768_60.00 --panning 1920x1080 --scale 1.6x1.40625
```

If you want to restore/revert to the original resolution you can do:
```bash
xrandr --output HDMI-3 --mode 1200x768_60.00 --panning 1200x768 --scale 1x1
```

This last tidbit of info came from [@mmacm](https://unix.stackexchange.com/users/36858/macm) with their [answer](https://unix.stackexchange.com/a/483774/187310) on a unix stack exchange question.

This info took a few hours to find/figure out, so consolidating the info into one page seems useful to my future self and others, even if its a bit rough around the edges.

:)
