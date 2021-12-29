---
title: "Monitoring the Weather"
date: "2017-06-11T19:59:58.000Z"
description: "Featuring InfluxDB, Grafana and Arduinos"
---


The weather has always fascinated me, ever since I was a little kid.  Watching clouds grow through sunny summer days, the eventual abrupt rainfall and cacophony of an evening thunderstorm.  Sudden run-for-your-life rainfall, and gusty winds that send everything scattered across the yard (https://www.youtube.com/watch?v=FRIaKMdO-Vw).

Over the years, I have built many little widgets and gadgets for measuring and observing the weather, from homemade anemometers made of 2 litre soda bottles, to seaweed hygrometers and a balloon barometer.

However, this is the IoT age, and I thought it only right that I have a go at making an internet-connected weather station, without just stumping up the money and buying one off the shelf.

I have to admit that after some careful experimentation with making my own anemometer and wind direction indicator, that I did end up buying a few bits and bobs off eBay to make my life considerably easier, especially in terms of calibration - but all the clever stuff in the Arduinos is all mine.
 

**Measurements I find interesting and can measure easily:**

* Temperature (well, duh.)
* Humidity
* Wind Speed
* Wind Direction
* Atmospheric Pressure
* Rainfall
* Solar Intensity
* Air Pollution (effectively a measure of particles 1-10μm in the air)


**Interesting measurements that are harder to measure:**

* Cloud cover Percentage.
* Air charged particles (theoretically a measure of percentage chance of a lightning strike)
* Dew Point (I’ve tried a number of ways to measure this, but it turns out to be Hard, and easier to measure temperature and humidity and calculate it).

 

## Making the Measurements

**Temperature:**

I tried a number of sensors for this, starting off with the DS1820 from Maxim and eventually settling on the Arduino ubiquitous DHT22 which does temperature with a range of-40 to 80°C temperature readings ±0.5°C accuracy. 

I found the DS1820 to be a bit of a special snowflake to set up, and the process was getting a little too maddening, so I moved on and tried a few other sensors (including a K-type Thermocouple and also a Thermistor) which eventually lead me to the DHT11, and subsequently the DHT22 (higher range)

**Humidity:**

Doing this from first principles is Difficult.  A lot of my early experiments were borne out of experimenting with Dew Points.  My original plan was to have a first surface mirror thermally bonded to a peltier module, then shine a laser at the mirror and detect it with a photodiode.  The theory in my mind was that if you drive the temperature down by putting power into the Peltier module, when dew forms on the mirror, it’ll change the refractive index of the mirror and make the light shine somewhere else so the dew could be detected by the photodiode.

In practice, it was such a bugger to get working as it required a whole bunch of temperature sensors - one on the cold side,  one on the hot side, and an ambient temperature.  The driving Arduino would measure the ambient temperature, then ramp up the current through the Peltier (via PWM switching with a beefy MOSFET) to chill the mirror, and stop every degree or so, take a laser measurement and then ramp the temperature down again.  

I tried a number of control mechanisms, including some proportional–integral–derivative controller (PID controller) libraries, but couldn’t ever get the thing to work right.

One of the biggest problems is that Peltier modules draw a *lot* of current when they’re pumping heat, I think mine was drawing roughly 12A at 12V and as a result, the switching MOSFET / Transistor was getting quite toasty, so I put it on the same heatsink as the hot-side of the Peltier module.  The hot-side DS1820 was to measure the temperature and switch another MOSFET on which controlled a cooling fan.


It’s a promising idea, and I’ll probably come back to this one. 

On the other hand, after I discovered the DHT11 and subsequently, the DHT22 sensors, I could get accurate temperature and humidity measurements thrown in for free, in an inexpensive arduino-friendly breakout board format for a nice low price. I think I paid about £3.50 each for them when I bought a half dozen. 


**Wind Speed**

Simple theory - Make an anemometer and figure out some way to measure every time it goes round once, then figure out the circumference and figure out the wind speed from that.

I definitely wanted something with good bearings so there’s as little friction as possible.

What has good bearings? Ooh.  Hard drives.  They’re good. 

I made this unholy contraption out of some little aluminium pots for cooking souffles in, and some bits of old aluminium window frame, and an old Maxtor hard disk.

I decided that I’d see whether I could use the tiny currents induced in the stepper motor drive coils as the motor turns to drive the wind speed counter. 

The biggest problem with this idea is that the voltages involved are so teeny as to be almost unmeasurable, so I built an amplifier with a seriously high gain AD620 Instrumentation Amplifier, which ended up giving me a PWM signal with TTL voltages that at least the arduino could read.. Then found a frequency counter library.  And it worked..  Until it got wet, then it stopped working until it dried out.

So I shelved that idea indefinitely and bought an off-the-shelf anemometer from eBay that cost around £3, and contains a reed switch, which closes once every revolution.  A little online digging http://mile-end.co.uk/blog/?p=86 shows someone else doing effectively the same thing as me, and all you have to do is count the clicks and multiply by 1.492 to get miles per hour.

**Wind Direction**

This was fun.  

Another application for low-friction bearings, so I had another go with another buggered hard drive.

This time, I took the platters off, and marked one of them up with a Gray Code pattern.  

I made a cutaway in the base of the hard drive body, and mounted a circuit board with 4 reflective photodetector sensors, giving a 4-bit gray code output.  In theory.

In practice, it was a bugger to get working.  The black bits were still quite reflective to infrared light, and the shiny bits were so reflective that the signal tended to bleed into the other channels’ signals.

So I ditched this idea, and bought a second hand Absolute rotary position encoder off eBay.  An Omron E6CP-A in case you’re curious, which gives a 8-bit, 12v Gray code output, and has a handy 5mm shaft to connect to the wind vane (I’m yet to make this bit).

Wind direction remains a work in progress.  I need to design my wind vane and plasma cut it and mount the whole thing to the encoder and write the software for an Arduino to interpret. 

**Atmospheric Pressure.**

I didn’t have any bright ideas of how to measure this from first principles, apart possibly from pointing a camera at a barometer and using openCV to read it. 

Trawling eBay for interesting Arduino sensors, I discovered the Adafruit BMP180, which has a very usable pressure sensing range of 300-1100 hPa (1 hPa = 1 millibar) and a temperature sensor thrown in for free (which I don’t use, because I already have the DHT22).

**Rainfall**

I bought another off-the-shelf rainfall sensor and modified it to make it more Arduino Friendly.

The one I bought was actually part of a Wireless rain gauge, simple tipping bucket that triggered a reed switch.  The original one used some frequency in the 433MHz range, and I had a good poke at it with CubicSDR, but wasn’t able to make any sense of the frequency and encoding it used, so I took a different route.

I hooked it up to its wireless receiver, and tipped the bucket.  The metered rainfall showed 0.3mm, so I knew that one bucket tip equated to 0.3mm rainfall on the surface of the funnel.

The next thing I did was to take the sender unit apart, remove the battery compartment and circuit board, and replace it with a little bit of veroboard with a pull-up resistor and a unipolar Hall Effect switch.  I upgraded the magnet from the teeny block of ferrite to a little Neodymium magnet, and tested it a few times both by running water through it, and manually flicking it over by hand, and was getting a nice, reproducible TTL-like falling edge whenever the bucket flipped.

**Solar Intensity**

I had a play with a raw Light Dependent Resistor (LDR) but was having calibration issues between trying to figure out how much light corresponded with which resistance.  LDRs are good for relative light levels, but they’re next-to useless for absolute values.

I discovered a sensor called BH1750FVI which produces a 16-bit I2C output giving an absolute value of 1 to 65535 lux.  

I haven’t integrated this into the system yet, but will soon. 

**Air Pollution**

I am aware of the ‘enterprise’ ways of doing airborne particle measurement (/hat tip to Malvern Instruments who have some seriously clever kit for particle size and counting in almost any fluid you can imagine) - but I’m cheap ;)

I discovered the PPD42NS which effectively measures particles in the air via Infrared Scattering. (https://github.com/SeeedDocument/Grove_Dust_Sensor/raw/master/resource/ShinyeiPPD42NS_Deconstruction_TracyAllen.pdf)

 I haven’t yet integrated this into the sensor network, as it’s currently in use at work measuring airborne dust particles in our server room. 

 

**Cloud cover Percentage.**

This is a theoretical sensor that I haven’t built yet.  My idea is to have a Raspberry Pi camera facing up, and take a photo every hour, and process it with openCV to estimate the amount of the image that isn’t blue, and use that as a measurement of cloud cover.  I’m aware that it won’t work at night. 


**Air charged particles**

I first came across this idea around a dozen years ago, but never got around to building one.  

http://www.techlib.com/files/cloud.pdf

I’m not sure if I ever will, but it’s a nice idea.


## CHOICE OF MICROCONTROLLER.

I have collected a fair number of microcontrollers for experimentation purposes over the years, from the fairly expensive but of limited use BASIC Stamp, to the incredibly inexpensive PIC16F84’s.  That said, my go-to favourite for light-weight processing is always the Arduino.  I tend to buy Arduino Nanos in bulk on eBay, ordering 10 or 20 at a time and then treating them effectively like disposable devices and soldering them directly into circuits. 

In this project though, because there are so many sensors, and each Arduino has so little in the way of memory, I opted for using a few separate Arduinos, and assigning a couple of sensors to each of them.  Combined with the problem that a few sensors have dramatically different operating modes, for example, Rainfall sensing is interrupt driven, producing a -_- pulse every time the bucket tips, whilst Wind Speed averages the number of pulses every couple of seconds and figures out the mean wind speed in that period.  

Separating the sensors onto their own Arduinos effectively simplified the programming, because each component could be tested individually and verified before adding to the sensor network.

**Interconnectivity between Arduinos**

There are a few options for this, I looked at a few of them in the initial testing.   I bought a bunch of RF Serial modules that operate at 433MHz which looked promising, but brings up questions about scheduling timeslots in which sensors communicate to avoid collisions, or whether to have the master broadcast a “HELLO Sensor WIND, are you there? What is your current value” and do it that way.. 

I looked at Arduino Ethernet, but this requires not only a lot of cabling (one ethernet cable per Arduino), but the Ethernet boards aren’t cheap either.

I did a brief bit of digging into Arduino I2C Master/Slave stuff, but found that the distances that I2C busses were supported over were actually quite short, and if I wanted to have any significant separation between the sensor and the receiver I would need quite a lot of i2C amplifiers.

**Enter CANbus.**

CANbus was developed in the 1980s by Bosch for simplifying wiring looms in cars and allowing many engine management systems to communicate easily. (https://en.wikipedia.org/wiki/CAN_bus). 

The primary benefit of using CANbus in this scenario is that it only needs 2 wires between devices, and they can be connected to the same two pieces of wire (because it’s a bus).

This cuts the amount of wiring down dramatically.  It also can operate over staggeringly large distances, if you only need a 50kbit/s throughput (and frankly, I can’t see this scenario generating any more data than that, ever), then the maximum bus length is 1km! (http://digital.ni.com/public.nsf/allkb/D5DD09186EBBFA128625795A000FC025)


CANbus modules for Arduino are typically based around the MCP2515, and use SPI for connectivity to the Arduino.  I used this https://github.com/coryjfowler/MCP_CAN_lib library for programming the Arduino, and allocated a MessageID based on the type of sensor (and the priority).

Because Rainfall sensing is interrupt driven (eliminates the requirement for the Arduino to keep a non-volatile count of the amount of rain in a given time period), it has the lowest message ID, and subsequently the highest priority.  This means that Rainfall messages will always get back to the receiver. 


I have currently got the weather sensors in an outbuilding, connected back to a Raspberry Pi (with another MCP2515 module) over a single pair of twisted wires.


Connecting the Raspberry Pi to the CANbus was surprisingly straightforward, I followed instructions here: https://harrisonsand.com/can-on-the-raspberry-pi/

I only had to make one change - presumably because the newer kernel changed the format for the `/boot/config.txt` file:

The article lists the lines to be added to the `config.txt` file as: 

```
dtparam=spi=on
dtoverlay=mcp2515-can0-overlay,oscillator=16000000,interrupt=25 
dtoverlay=spi-bcm2835-overlay
```

However I discovered that it doesn’t work, and you have to remove -overlay from the configuration lines:

```
dtparam=spi=on
dtoverlay=mcp2515-can0,oscillator=16000000,interrupt=25 
dtoverlay=spi-bcm2835
```

Then it just works.

The Raspberry Pi has an ethernet port, and connects to the network like any other device.  

## Data Storage and Graphing

Initially, whilst testing this all out, I was using my laptop which has Ubuntu on it, and an intermediate Arduino acting as the sensor master, converting CANbus messages to Serial messages and squirting them out over `/dev/ttyUSB0`. 

I wanted to make the whole thing smaller however, and a full size laptop was a step in the right direction, but not quite good enough.

I had a quick hunt on the Internet for Time Series Databases, because using Postgres would’ve been possible, but didn’t feel quite right somehow. 

I found OpenTSDB, written in Java, and InfluxDB written in Go.

Installation of InfluxDB looked easier (and indeed, it was, given there’s native .deb packages) and I didn’t really feel like farting about with Java.

 
```
wget https://dl.influxdata.com/influxdb/releases/influxdb_1.2.4_amd64.deb
sudo dpkg -i influxdb_1.2.4_amd64.deb
```

I had the entire system up and running in a couple of hours with InfluxDB and Python - I wrote a noddy script to read the serial input with PySerial and squirt the values into InfluxDB, then a quick and dirty Flask application to display the latest values, and the highest day’s temperature. 

InfluxDB has a rather nifty SQL-like query language, so it’s possible to write something like:

`SELECT MAX(temperature) from “weather.historical” WHERE time > NOW() - 24h;`

To get the maximum temperature recorded in 24 hours.

Works pretty well.

My initial intention was to write a graphing thing with d3js, but in the process of searching for graphing libraries that supported time series databases, I rediscovered Grafana - a thing for making dashboards.  I shoved that on the laptop too, and was able to get a full weather dashboard in a matter of minutes.  Sod the d3js thing.  This is easier.

When I swapped from Laptop to Raspberry Pi, my plan was to install InfluxDB and Grafana on the ‘pi too, and have it all nicely accessible.  However, the .deb packages for Influx and Grafana for armhf are a little out of date, and lack the nice features of the current version I’d been playing with.

It would also have made it difficult to make it internet accessible without forwarding ports on the router and so on.

Instead, I spun up a lightweight VM on Digitalocean, and configured the python to write the InfluxDB data directly up to the cloud.  

Grafana runs far more happily on the VM than it ever did on the ‘pi. 

 

 