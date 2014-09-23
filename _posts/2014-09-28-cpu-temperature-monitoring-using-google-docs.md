---
layout: post
title: CPU Temperature Monitoring using Google Docs
---
I have a Gateway laptop that I bought (refurbished) about five years ago.  It's running Linux Mint, and I use it for light web browsing and playing music on the sound system. It's not
great - the build quality was pretty poor to begin with, and it's definitely showing its age - but it works, mostly.

This summer, however, it began to turn itself off.  Hard shutdown, no warning, just instantly dead.  The most common cause of this is overheating - the system shutting itself
off to prevent damage.  But the laptop wasn't that hot to the touch.  It was running its fans on high speed most of the time, but I'd felt laptops get a lot hotter before.  The problem
got worse, though.  Eventually it was so bad that it would shut off within minutes of booting, even after hours of sitting idle.

I started to think it might be the thermal paste - the goop that transfers heat from the CPU and GPU to the heat sink, and from there out of the case.  What if it had dried out?  I could
replace it, but I didn't relish the prospect of cracking the case on this thing for no reason (luckily, it had become useless anyway, so I could risk accidentally destroying it).

I wanted to be sure that overheating was the cause of the shutdowns. I needed to log the temperature readings from the motherboard sensors, and I wanted to send them to a healthy system
so I could see what was happening.  I decided to see if I could send them to a Google Docs spreadsheet - I could collect the data there, and also analyze it or export it as necessary.

First of all, I installed the `sensors` package.  Running this gave me output like:

    acpitz-virtual-0
    Adapter: Virtual device
    temp1:        +94.0°C  (crit = +100.0°C)
    temp2:        +87.0°C  (crit = +100.0°C)
    
    k10temp-pci-00c3
    Adapter: PCI adapter
    temp1:        +94.1°C  (high = +95.0°C)

Okay, so I have three sensors in this machine, and they were running pretty freaking hot. Not good. Now I needed to parse that output for just the current values, removing all of the other
stuff.  After some command-line fiddling, I settled on:

`sensors | grep ^temp | tr -d "°C+" | tr -s " " | cut -d" " -f2 | paste -sd " "`

which resulted in one line with all three values on it: `94.0 87.0 94.1`.

In Google Docs, I created a spreadsheet with the columns I wanted - a timestamp and three sensor fields. To collect the data, I linked a survery form to the spreadsheet - I'd be able
to write a script to submit the sensor values using a simple HTTP POST.  The source code for the form contained the field names I'd need to send.

My final script would post the sensor values every 30 seconds. It looked like this:

{% highlight bash %}
#!/bin/bash

TARGET=https://docs.google.com/forms/d/1Atc69Dy4ZFnOQJMKd9grcd5cy8iYNT0T-BwYJFXjVpA/formResponse
S1NAME=entry.1507057940
S2NAME=entry.1234041023
S3NAME=entry.1730611246

while true
do
 FIELDS=$(sensors | grep ^temp | tr -d "°C+" | tr -s " " | cut -d" " -f2 | paste -sd " ")
 S1VAL=$(echo $FIELDS | cut -d' ' -f1)
 S2VAL=$(echo $FIELDS | cut -d' ' -f2)
 S3VAL=$(echo $FIELDS | cut -d' ' -f3)
 echo "$S1NAME=$S1VAL&$S2NAME=$S2VAL&$S3NAME=$S3VAL" | POST -d "$TARGET"
 sleep 30
done
{% endhighlight %}

(The form is no longer accepting responses.) I set it up as a cron job to run on system startup: `@reboot /root/templog.sh`.

With the spreadsheet happily accepting my sensor values, I rebooted the machine a few times, letting it turn itself off each time and cool down for a while. While I waited, I put together
a chart view of the sensor data. The result was [this](https://docs.google.com/spreadsheets/d/1ZrOIIzPUbQYuPiaLOiLyeDF8wbLX7Mv_wAF5c83HS4U/pubchart?oid=1554113740&format=interactive):

<iframe src="https://docs.google.com/spreadsheets/d/1ZrOIIzPUbQYuPiaLOiLyeDF8wbLX7Mv_wAF5c83HS4U/pubchart?oid=1554113740&amp;format=interactive" style="height:380px;width:100%;border:0;"></iframe>

In the end, I didn't need to do much analysis. It was clear that in each case, the system temperature climbed until sensors #1 and #3 were near 100°, and then the system shut down, 
registering no data points until it started up again at a much lower temperature.

So heat was definitely the culprit. I ordered a tube of thermal paste.

I opened up the laptop, carefully collecting all the tiny screws (a strongish magnet in a stainless steel bowl works great!).  The heat sink assembly was mounted near the
very bottom of the case.  I unscrewed and removed the heat sink, exposing the thermal paste on the CPU and GPU. It was fairly dry and crumbly. I scraped it off, removing the last remnants 
with isopropyl alcohol, and applied the new paste.  With that done, I reassembled the laptop and (to put it mildly) was quite relieved when it booted up normally.

Here are the [post-op temperature readings](https://docs.google.com/spreadsheets/d/1ZrOIIzPUbQYuPiaLOiLyeDF8wbLX7Mv_wAF5c83HS4U/pubchart?oid=2022279560&format=interactive):

<iframe src="https://docs.google.com/spreadsheets/d/1ZrOIIzPUbQYuPiaLOiLyeDF8wbLX7Mv_wAF5c83HS4U/pubchart?oid=2022279560&amp;format=interactive" style="height:380px;width:100%;border:0;"></iframe>

Pretty good. It still runs a bit hotter than I'd like, particularly under load, but it hasn't turned itself off once since it got the new thermal paste. All in all, a very satisfying result.










