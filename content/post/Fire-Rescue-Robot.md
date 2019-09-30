---

title: Fire Rescue Robot 
draft: false
layout: post
date: 2017-12-16T00:00:00-06:00
tags:
  - mechanical
  - academic
  - software
description: |
 December 2017:
 Senior design project to design and prototype a remote
 controlled fire rescue robot
 
---

<!--
<br>
<img class="img-responsive center_img" style="max-width: 75%" src="/img/back right view.png">
<br>
-->
<br>
<video class="img-responsive center_img" style="max-width: 75%"  src="/img/fire-rescue.webm" type="video/webm" controls autoplay loop>
Sorry, your browser doesn't support embedded videos, 
but don't worry, you can <a href="/img/fire-rescue.webm">download it</a>
and watch it with your favorite video player!
</video>
<br>

In Fall of 2017 I took the Mechanical Engineering Design and Methodology course,
as a precursor to Senior Design the next semester. This course was structured to
teach methods through hands-on design work for a project throughout the
semester. The project for this semester was to create a fire rescue robot with
the goal of entering a high-temperature environment and locating an injured
pet. We used several concept generation methods to brainstorm subsystems for the
three main subsystems: thermal protection, movement, and control. The leading
concepts for each of these systems was then prototyped and integrated together
to create a beta prototype of the full vehicle. The robot was placed in a 90
degree Celcius oven to test it's performance in high temperature environment.
Afterwards the robot was placed in a dark room, and was remote controlled to
identify the pet based off the heat differential between it and the ambient
temperature.

For this project I was responsible for the entirety of electronic and software
systems. The control board of the robot was an ESP8266, an inexpensive
microcontroller with a on-board wifi chip. We used ultrasonic sensors for object
avoidance and an 8x8 thermal imaging camera to identify the pet. Using the
Arduino bootloader, I developed software which hosted an HTML webpage that was
used for remote control of the robot. The user would login to the access point
from the ESP8266 and open the web page in a browser. From the web page the
thermal and ultrasonic sensors could be read. Additionally movement commands
could be sent to the robot. Real-time bidirectional communication was achieved
through a WebSocket server hosted on the microcontroller. Sensor data was
decoded in Javascript by the client webpage and the thermal data was displayed
through a heatmap. 

The full report describing the robot in detail and the design methodology behind
it is linked [here](/assets/img/ME_366J_Final-Report.pdf).
Linked above is a video showing the operation of the
robot from the client page and the robot itself in use. The software used to
control the robot is hosted on Github
[here](https://github.com/josephvoss/ME-366J_Fire-Rescue/).

<br>
<img class="img-responsive center_img" style="max-width: 75%" src="/img/IMG_20180228_210610.jpg">
<br>
<img class="img-responsive center_img" style="max-width: 75%" src="/img/IMG_20180228_210401.jpg">
<br>
<img class="img-responsive center_img" style="max-width: 75%" src="/img/IMG_20180228_210450.jpg">


<!--
<br>
<img class="img-responsive center_img" src="/img/Full_car.jpg">
<br>
-->
