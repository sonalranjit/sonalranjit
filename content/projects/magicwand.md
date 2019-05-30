+++ 
draft = false
date = 2014-02-14T11:22:01-04:00
title = "Magic Wand"
slug = "magicwand" 
+++

![](/images/wand.jpg)

This was one of my first hardware projects I have been part of. During my sophmore year at York University I was fortunate enough to be part of a collaborative makeshop session run by the [GaMay Lab](http://gamay.lab.yorku.ca) and [Foad Hamidi](http://foadhamidi.com). During this session we had the opportunity to build anything we wanted, there were supplies such as arduino boards and 3D printers provided. I had partnered with Chitiiran Krishna Moorthy and Kajendra Seevananthan, we settle on an idea of building a Harry Potter inspired Magic Wand, we were both Harry Potter fans so we were very excited when we starting going down the rabbit hole of making a wand, it sounded doable so we went with it.

The wand recognizes a pre determined set of hand gestures and upon a correct gesture match the wand fires the green laser. The wand was meant for casual gameplay, Harry Potter enthusiasts to take part in a wizards duel.

{{< figure src="/images/wand_schematic.png" caption="Wand Schematic" >}}

The magic wand consists of 3 major components [Arduino Micro Microcontroller](https://store.arduino.cc/usa/arduino-micro), a 6 DoF mems [IMU](https://www.sparkfun.com/products/retired/10121) from Sparkfun and 5mW 532 nm laser. The IMU was used to determine the orientation of the wand, the Arduino Micro for the computation and control. The wand itself was design in CAD and 3D printed. 

## Data processing

The angular data (Yaw, Pitch and Roll) data incoming from the IMU was initially used to recognize hand gestures, but problems arising in the drift of the angular reading this was possible. A 9dof freedom would solve this problem but then the design  would have to be altered and due to the size of a 9dof IMU this was not possible. A filter could’ve been implemented for the consistent noise to be removed but don’t have understanding of this math during that time. Therefore only the accelerometer values for each axis is being used for gesture recognition, the accelerometer data was drift free but it was very noisy, but that was the tradeoff we had to live with.

{{< figure src="/images/imu_visual.png" caption="IMU Orientation Visual" >}}

A time window was set on how long it would take to complete a gesture, then it was sub divide into equal intervals. For each sub interval the total change of orientation was stored in an array. If the gesture array sequentially matched any of the pre defined gesture arrays a gesture is recognized and the laser is fired.

## Wand Design
The intial wand design was a traditional one as seen in the Harry Potter movies, during the printing and it was quite small and tight to fit all of the electronics. Therefore the 2nd and final design iteration was taken from one of the video games we usually played DOTA 2, specifically the character [Lion's Scepter](https://dota2.gamepedia.com/Lion/Equipment). The 3D designs are available on [thingiverse](https://www.thingiverse.com/thing:248254).

## TEI 2014
The wand was presented at the [Student Design Competition](https://web.archive.org/web/20150330042651/http://www.tei-conf.org/14/design.php) part of Tangible Embedded and Emobodied Interaction Conference 2014 in Munich, Germany.
Short blurb on York University's [page](http://lassonde.yorku.ca/articles/lassonde-students-engineer-magic-global-stage)

## Team
{{< figure src="/images/wandmakers.png" caption="Wandmakers from left Kajendra, Chitii, Sonal" >}}