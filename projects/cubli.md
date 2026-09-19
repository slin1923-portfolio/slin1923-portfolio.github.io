---
layout: default
title: Cubli
---

- [Todos \& Current Issues](#todos--current-issues)
- [Overview](#overview)
- [Mechanical Parts](#mechanical-parts)
- [Electronics](#electronics)
- [Firmware](#firmware)
  - [Platform.IO and mosquiTTo reminders](#platformio-and-mosquitto-reminders)
- [Modeling and Dynamics](#modeling-and-dynamics)
  - [CoM ID](#com-id)
  - [System ID](#system-id)
  - [Simulink](#simulink)

# Todos & Current Issues

- Need to characterize CoM of cubli using a set of 3 fall-tests
  - perform fall tests on rubber mat to prevent corner slippage from influencing dynamics
  - 10x falls (both sides) on all 3 corners
  - least-squares fit the CoM position
- System ID
  - hang cubli by a string and read system IMU response to full-speed step function. 
- Translate to simulink

# Overview

github: https://github.com/slin1923/custom_cubli

*updated 9/16/2026*

<figure align="center">
  <img src="/assets/images/cubli/assembled_cubli_flywheel_side.jpg" width="800">
  <figcaption>CUBLI</figcaption>
</figure>

Cubli is the first of my projects for which I will use my portfolio as running documentation of the current state of the project.  Cubli is inspired by the project of the same name developed at ETH Zurich by the Institute for Dynamic Systems and Control. Using a set of 3 reaction wheels, this robot should be able to balance on a corner or an edge.  While on a corner, it should also be able to slowly rotate cw/ccw. See RemRC's video below for what this cube should be able to do.  As someone who wants to fly satellites, I figured this project was a decent way to get my hands dirty and get some fun ADCS experience on real hardware under my belt.  

The purpose of this project is for me to have a physical sandbox for testing controllers.  The emphasis of this project is solely on dynamics and controls.  I want to give a HUGE thank you to RemRC for open-sourcing the hardware for this awesome project.  His version is fundamentally similar to the research-grade hardware of ETH Zurich, except redesigned with cheap, accessible COTS parts.  All of the part files and schematics for his project you can find on his github repo.  I did however make just a few hardware modifications to this project which you can find on my repo. 

Now I didn't just follow RemRC's documentation and work instructions to a T.  That would make this project meaningless and no more of an exercise in engineering than a lego set is. **I am building a bunch of custom controllers**.  A quick glance at RemRC's controller firmware shows that he used a very simple, but well-tuned proportional controller on both gyroscope and accelerometer readings. I can only assume he did not have to model the system dynamics and do any system ID. Now while what he did certainly works, I'm an idiot and intentionally want to use more advanced controllers. I want to model the dynamics of the cube and at least TRY to implement literally every type of controller I know: P, PI, PID, pole-placement, LQR, MPC, H-$\infty$, Adaptive Control, hell if I design a rig that automatically stands the cube back up to its equilibrium point even a RL policy is on the table!  Such ambition... get to work!

When completed, this may be my magnum opus personal project. I do not explain background theory. Unlike many of my other posts, the intended audience for this post are people in the robotics/GNC field. 

<figure align="center">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/AJQZFHJzwt4?start=836" title="ReM-RC's original demonstration" allowfullscreen></iframe>
</figure>

# Mechanical Parts

Mechanical design is not the focus of the project. ReM-RC designed 90% of my cube, however, all his [original Thingiverse files](https://www.thingiverse.com/thing:6695891) are already STLs.  I reverse-designed them to be parametric Onshape files :).  I also desiged some additional parts of my own you can find below. 

If you have an Onshape account (free), you can export, or copy + modify any of the parts below.   
- [Original remRC hardware reverse-engineered and modifiable](https://cad.onshape.com/documents/9db53cb7fffbe7f9bdffecbf/w/8b2b0a3894528416909fe18f/e/54f5cadad0079e412704754e): *most of the original hardware is here*
- [Custom Cubli corner bracket](https://cad.onshape.com/documents/10eeab58e3585dbe52929d41/w/f9cb36a879b1aec020436003/e/f00089ac6634819386f3b227): *to protect cubli sharp corners*
- [Custom Cubli battery hardstop and speaker holder](https://cad.onshape.com/documents/cab1210c8b19e6f436395f9d/w/911c586d0007dd9d1f17d39b/e/0d5b19efdc2096d2093d3bce): *battery hardstop and speaker holder combined into one part*
- [Custom Cubli NIDEC motor connector retainer](https://cad.onshape.com/documents/26e80f0ebc20d2290574231d/w/883115062f373953e935bcf8/e/6b278096f302a735771e8acb): *because the connector kept disconnecting from the motor*

<figure align="center">
  <img src="/assets/images/cubli/speaker_holder.jpg" width="400">
  <figcaption>Custom Cubli battery hardstop and speaker holder</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/cubli/connector_retain_2.jpg" width="400">
  <img src="/assets/images/cubli/connector_retain_3.jpg" width="400">
  <figcaption>Custom Cubli NIDEC motor connector retainer</figcaption>
</figure>

Here's a quick gif to appreciate ReM-RC's design: the flywheel runs clear and true!

<figure align="center">
  <img src="/assets/images/cubli/cubli flywheel face design.gif" width="600">
  <figcaption>Flywheel face of cubli demonstrating flywheel clearance spin</figcaption>
</figure>

# Electronics 

I used The Nidec24H Brushless DC motor for this project, per ReM-RC (again hardware was not the focus of this project). The full datasheet is [here](../assets/pdfs/Nidec24H%20BLDC%20Product%20Data%20Sheet.pdf). 

Here is my DIY-soldered controller board.  I soldered mine slightly differently from ReM-RC since I used a slightly different ESP32 WROOM board.  So I also have my own schematic. 

<figure align="center">
  <img src="/assets/images/cubli/cubli_controller.jpg" width="600">
  <figcaption>DIY soldered cubli controller</figcaption>
</figure>

My onboard sensor is the MPG6050 IMU, a small powerful 6 DoF board. 

<figure align="center">
  <img src="/assets/images/cubli/mpu6050_imu.jpg" width="600">
  <figcaption>Cubli's onboard IMU</figcaption>
</figure>

# Firmware

In the past, I always flashed to ESP32s using the USB cable and also used the same cable as my data connection for Serial.  I would have gone crazy if I needed to do that for Cubli.  Not only because I iterate and flash frequently, but more critically because I would need IMU data streamed back to me without the mechanical attachment of a cable affecting Cubli's dynamics! Platform.IO is a way to flash over wifi and mosquiTTo is a way to publish and subscribe to OTA messages (not dissimilar to ROS). 

## Platform.IO and mosquiTTo reminders

The workflow for OTA flashing

1. Open the project in Platform.io (the folder must contain a file ```platformio.ini```)
2. Make sure ```platformio.ini``` contains at least the following 
   ```
   [env:esp32dev]
    platform = espressif32
    board = esp32dev
    framework = arduino

    # comment the next two lines to upload via USB
    upload_protocol = espota
    upload_port = 192.168.0.220
   ```
3. compile and upload (*check mark and arrow in bottom left of VSCode Window*)

The workflow for OTA IMU data readout
1. Open a terminal and run ```& "C:\Program Files\mosquitto\mosquitto.exe" -c "C:\mosquitto-config\mosquitto.conf" -v```.  All this does is start mosquiTTo
2. Open a new terminal and run ```& "C:\Program Files\mosquitto\mosquitto_sub" -h localhost -t esp32/accel``` for live IMU readouts.
3. ALTERNATIVELY, run ```python log_imu.py``` and then run ```python analyze_imu.py``` if running a proper system ID test.

Reminders
- to make sure ArduinoOTA is not locked out of the ESP32: 
  - Every ```main.cpp``` wrapper flashed to the ESP32 must contain ```#include <ArduinoOTA.h>```
  - Every ```void setup()``` method needs to contain 
    ```
    ArduinoOTA.setHostname("esp32-imu");
    ArduinoOTA.begin();
    ```
  - In as many places as possible within ```void loop()```, especially where you may get stuck within a conditional, add 
    ```
    ArduinoOTA.handle();
    ```
  - essentially, the firmware needs to no only be executing the control loop but must always be listening for new code too.
- Troubleshoot: if will not upload OTA, check in order
  - ping the IP address and see if you get a response, if no response, reset the ESP32 and try pinging again
  - use IP scanner to see if address has changed.  
  - check for Windows system Firewall (may have been reactivated after software update)
  - check for brownout (insufficient power to ESP32 due to depleted battery)
  - check for ```ArduinoOTA.handle();``` blind spots
  - ask Claude
- Troubleshoot: if cannot get messages back
  - Make sure ```esp32/accel``` matches the ```mqtt_topic``` in ```main.cpp```
  - run ```net stop mosquitto``` and try again (essentially restart mqtt).

# Modeling and Dynamics

Of the top of my head (with many assumptions thrown into the mix), I think there are 3 main things I need to ID in order to build a good (albeit still wrong) model of cubli.  essentially this is an inverted pendulum with all 3 angular DoF, so I will model it as such. 
1. CoM of the cube location $l$
2. mass of the cube $m$
3. I need a map of PWM to imparted-angular-momentum for each motor-flywheel pair (I can assume they are all the same, but I shouldn't).  

The mass of the cube is trivially measurable at **1093.1 g**, just north of a kilo. I also have the mass of each flywheel at **79.5g** per wheel although this is actually negligible because we only care about the wheel's inertia tensor. it's still nice to have an idea of how heavy my flywheels are, and as a sanity check they take up roughly 25% of the total mass of Cubli. The other two items on the list above are a bit less trivial but some well-designed tests will do the trick. 

Below I keep track of things I THINK and HOPE I will not have to model, but it may come into play.  So regarding the following, I am ignoring, not ignorant. 
- The motors transient response may be non-neglibly long.  If it is so short of a rise time that it's within my controller's frequency I'll be ok. 
- The motors might have some sort of hysteresis such that responses on the ramp-up look different from response from the ramp-down.  hopefully this is negligible. 

I get raw IMU data back and plotted at 100 Hz.  Importantly, timestamps for data are and SHOULD not be based on laptop time-of-receival but labeled directly on the ESP32 before sending for accuracy. 

## CoM ID
To ID my CoM I performed three independent drop tests along 3 axes from equilibrium positions.  $\dot{\theta}$ data is read straight off the IMU (currenlty debating whether I should implement a complementary filter to fuse IMU and accel data), integrated, or differentiated, and appled to $\ddot{\theta} = \frac{g}{l}\sin{\theta}$. I least-squares fit for the parameter $l$. 

<figure align="center">
  <img src="/assets/images/cubli/droptest_cubli.gif" width="600">
  <figcaption>examples drop test. </figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/cubli/axis_2_drop_test.png" width="600">
  <figcaption>IMU drop test raw data.  You can see the 5 different drops. </figcaption>
</figure>

## System ID

## Simulink