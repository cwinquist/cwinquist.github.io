---
layout: post
title: Building a SAS JBOD
---

### The Problem

After dealing with several disk failures over the past year and picking up some 
new disks to cover for the now out of warranty failure prone disks, I needed
some additional space to be able to copy everything over and a better ability
to handle drive failures without downtime. My temporary solution of disks and
a hot wired power supply was just not cutting it any more.

![Four disks and a hotwired power supply](/assets/images/diy-das-2017/IMG_20171107_222702.jpg)

After checking around ServeTheHome, the popular solution was to buy a used
commercial DAS or rackmount server chassis. This is a cost effective and easy 
way to add capacity but there were a couple issues for me. My rack is only for 
network equipment and has a max depth of about 14 inches. While there is no
reason a disk enclosure couldn't fit in this space, it would be tight and most 
racks are deeper so this isn't exactly a large target market for companies to 
produce this stuff.

Also, noise is genrally not a concern in a datacenter environment but since
this is in my basement the jet engine fan noise of a 40mm fan is a no go.

The closest thing I could find off the shelf was the Lenovo SA120. It is fairly 
shallow to the point where I **may** have been able to fit it in the rack with
some extensions. It has the 40mm fans but these can be controlled using SES
to make the noise somewhat manageable. In the end, I decided to build something 
rather than take chances with **may** fit and *somewhat manageable noise*.

### Parts

#### Case

I went with the Azza Solano 1000R because it had the most 5.25" bays I could 
find for a reasonable price. While the aesthetic may not appeal to everyone, it
was going to be in a closet anyways so the main drivers were 

* able to fit in the space I had in the closet
* able to hold as many drives as possible
* as cheap as possible

![Front of case with no cages](/assets/images/diy-das-2017/IMG_20171116_232147.jpg)

#### Power Supply

Since it's only disks most reasonably power supplies won't be a problem. The 
main requirement that's a little out of the ordinary here is the large number 
of molex connectors required

* 2x molex per cage
* 1x molex for the SAS expander

With three cages this means 7x molex connectors with splitters not being 
advised - melting and burning things in the context of computer hardware is
bad. I ended up using an old Seasonic I had pulled from an old desktop.

To control it, I just have an ATX connector with a power switch as a slightly
neater alternative to a paper clip. They can be had for $5 or so and makes
things a lot cleaner. SuperMicro also makes a stand alone controller that also
can handle 3-pin fans that can be had for $50 or so. Since the back of the case 
was open enough and the cages had fans this was not needed for this build.

#### SAS Expander

While it would be possible to run SFF-8088 breakout cables into the back of the case directly, the total length would be limited to 1m when used with SATA 
drives. This doesn't work too well with the layout of where the DAS will sit 
so I had to add a SAS expander.

![SAS Expander card](/assets/images/diy-das-2017/IMG_20171212_220605.jpg)

Here I went with the LSI based Intel RES2SV240. Readily available for reasonable prices
and able to run SATA drives at full speed. Also capable of running a double
link back to the host so it should even be possible to run a few SSDs on it
in a pinch without bottlenecking too badly.

#### Hotswap 5 in 3 cages

![Front of SuperMicro cage](/assets/images/diy-das-2017/IMG_20171117_205640.jpg)
![Back of SuperMicro cage](/assets/images/diy-das-2017/IMG_20171117_205648.jpg)

Here I went with three SuperMicro CSE-M35TQB mobile racks. They cost a little 
more than similar products but have a few advantages.

* 92mm fans mean less noise
* Compatible with SAS
* Supports SES

The last point is fairly key if a bit obscure. SES makes it so you can easily
locate drives if you have a side channel connector attached. So if da8 is 
throwing smart errors, you can turn on the light for whatever da8 is and you
don't need to label serial numbers on the front of the drives.

One other thing I didn't know until the enclosures showed up - there is a 
SFF-8087 breakout cable with sidebands included in the box that can connect
4 of the 5 bays.

One limitation with the sideband is that it only supports 4 bays per cable
so for the 5th bay you would need a second sideband connector. Still this beats
having to print labels so I can pull the right drive in case of failure.

#### Cabling

![Side of case with one set of SFF-8087 hanging out](/assets/images/diy-das-2017/IMG_20171116_232127.jpg)
![Back of case with SFF-8088 ports](/assets/images/diy-das-2017/IMG_20171116_232138.jpg)

This project involved loads of SFF-8087/8088 cabling. Supermicro's online store
is very reasonable on the pricing of this stuff and shipped quickly. They even
had a double SFF-8088 bracket that connected to SFF-8087 on the expander card 
making everything reasonably neat even if most of the cable options were quite
long. They are also quite stiff which makes cable management a bit of a 
challenge.

### Construction

The first snag with this build was the fact that the case has tabs between each
bay designed to make it easy to put in DVD drives or single 5.25" to 3.5"
brackets. The Supermicro cages have flat sides and will not fit around the tabs.
I could not find a way to bend them down without bending up the bays themselves
and making a worse mess of things so I was forced to get out my trusty hacksaw.

![Cutting case](/assets/images/diy-das-2017/IMG_20171210_201914.jpg)

Next up was the drive cages. First of all, they need to be rejumpered since 
the default is to run the sidechannel in I2C mode and it needs to be in SGPIO 
mode for the Intel SAS expander.

![Rejumpering drive cage](/assets/images/diy-das-2017/IMG_20171212_210355.jpg)

Finally, the included 92mm fans run at high RPM and are a bit louder than I 
wanted. It was a close decision but they were still audible in the room even
with the cage in the closet so I replaced them with Noctua 92mm fans. This makes
the setup much quieter at the expensive of a bit of airflow.

![New fan on drive cage](/assets/images/diy-das-2017/IMG_20171212_211535.jpg)

I inserted the mostly wired drive cages into the front of the case. The
fit was a bit tight but all 3 cages went into the top 9 bays allowing the bottom
bay for stuffing spare PSU cables (6 pin, SATA, etc).

![Front of case with drive cages](/assets/images/diy-das-2017/IMG_20171212_214617.jpg)
![Side view with unwired drive cages](/assets/images/diy-das-2017/IMG_20171212_214627.jpg)
After wiring everything up this is the finished product, with 15 3.5" hotswap bays allowing for more 
storage than I'll need for the forseeable future. 

Power usage with no disks measured 28.8W at the wall. This doesn't seem 
terrible since we're not exactly in the most efficient part of a 450W power 
supply's range and there's a few fans and a SAS expander running in there.

![Finished disk array](/assets/images/diy-das-2017/IMG_20171212_232032.jpg)

Here is the JBOD taking its place next to the server in the closet. The black
cable betwen the two is the SFF-8088 cable.

![Server and disk array](/assets/images/diy-das-2017/IMG_20171212_232010.jpg)
