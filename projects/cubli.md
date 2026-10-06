---
layout: default
title: Cubli
---

- [Todos \& Current Issues (for rapid personal reference)](#todos--current-issues-for-rapid-personal-reference)
- [Overview](#overview)
- [Mechanical Parts](#mechanical-parts)
- [Electronics](#electronics)
- [Firmware](#firmware)
  - [Platform.IO and mosquiTTo reminders](#platformio-and-mosquitto-reminders)
- [Modeling and Dynamics](#modeling-and-dynamics)
  - [Nomenclature and Values (for quick reference)](#nomenclature-and-values-for-quick-reference)
    - [Experimental Values (Cubli intrinsics)](#experimental-values-cubli-intrinsics)
    - [Analytical Variables](#analytical-variables)
  - [Frames of Reference](#frames-of-reference)
  - [Desired State-Space Realization](#desired-state-space-realization)
  - [Flywheel Inertia Calcs](#flywheel-inertia-calcs)
  - [Inertia Tensor Experiments](#inertia-tensor-experiments)
  - [Measurement Model](#measurement-model)
- [Torque-Based Motor Control](#torque-based-motor-control)


# Todos & Current Issues (for rapid personal reference)

- Finish torque-based motor inner-loop control
  - Need to fit data to a first-order model of motor output torque
  - switch to Serial from MQTT for high-frequency data collection fidelity
  - Characterize $t_{TC}$ for reaching torque target
- Characterize $I_{zz}$ of cubli using string suspended spin test
- Migrate state space realization into Simulink and intuitively verify system behaves as expected 
- Sanity check the measurement model using real world experiment and validate $C$ with collected data

# Overview

github: https://github.com/slin1923/custom_cubli

*updated 9/16/2026*

<figure align="center">
  <img src="/assets/images/cubli/assembled_cubli_flywheel_side.jpg" width="800">
  <figcaption>CUBLI</figcaption>
</figure>

Cubli is the first of my projects for which I will use my portfolio as running documentation for.  Cubli is inspired by the project of the same name developed at ETH Zurich by the Institute for Dynamic Systems and Control. Using a set of 3 reaction wheels, this robot should be able to balance on a corner.  While on a corner, it should also be able to slowly rotate cw/ccw. ReM-RC's video demonstrates what this balancing should look like, though he does not track yaw angles.  As someone who wants to fly satellites, I figured this project was a decent way to get my hands dirty and get some fun ADCS and firmware experience under my belt.  

The emphasis of this project is solely on dynamics, state estimation, and controls. I want to give a HUGE thank you to ReM-RC for open-sourcing the hardware for this awesome project.  His version is fundamentally similar to the research-grade hardware of ETH Zurich, except redesigned with cheap, accessible COTS parts.  All of the part files and schematics for his project you can find on his github repo.  I did make a few hardware modifications to this project which you can find on my repo. 

I didn't just follow ReM-RC's documentation and work instructions to a T.  That would make this project meaningless and no more of an exercise in engineering than a lego set is. **I am building custom controllers**.  A quick glance at RemRC's controller firmware shows that he used a simple, effective, model-free, and well-tuned proportional controller on both gyroscope and accelerometer readings. While that certainly works, I'm an idiot and intentionally want to shoehorn in advanced controllers. I want to model the dynamics of the cube and at least TRY to implement literally every type of controller I know: P, PI, PID, pole-placement, LQR, MPC, H-$\infty$, Adaptive Control, hell if I design a rig that automatically stands the cube back up to its equilibrium point even a RL policy is on the table!  Such ambition... get to work!

When completed, I want this to be my magnum opus personal project. Unlike many of my other posts, the intended audience for this post are people in the robotics/GNC field and contains a lot of deep technical concepts sans explanation. 

<figure align="center">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/AJQZFHJzwt4?start=836" title="ReM-RC's original demonstration" allowfullscreen></iframe>
</figure>

# Mechanical Parts

ReM-RC designed 90% of my cube. Annoyingly, all his [original Thingiverse files](https://www.thingiverse.com/thing:6695891) are already non-parametric .STLs.  I reverse-designed them to be parametric Onshape files :).  I also designed some additional parts of my own you can find below. 

If you have an Onshape account (free), you can export, or copy + modify any of the parts below.   
- [Original ReM-RC hardware reverse-engineered and modifiable](https://cad.onshape.com/documents/9db53cb7fffbe7f9bdffecbf/w/8b2b0a3894528416909fe18f/e/54f5cadad0079e412704754e): *most of the original hardware is here*
- [Custom Cubli corner bracket](https://cad.onshape.com/documents/10eeab58e3585dbe52929d41/w/f9cb36a879b1aec020436003/e/f00089ac6634819386f3b227): *to protect cubli sharp corners*
- [Custom Cubli battery hardstop and speaker holder](https://cad.onshape.com/documents/cab1210c8b19e6f436395f9d/w/911c586d0007dd9d1f17d39b/e/0d5b19efdc2096d2093d3bce): *battery hardstop and speaker holder combined into one part*
- [Custom Cubli NIDEC motor connector retainer](https://cad.onshape.com/documents/26e80f0ebc20d2290574231d/w/883115062f373953e935bcf8/e/6b278096f302a735771e8acb): *because the connector kept disconnecting from the motor*

Most parts FDM printed in PLA and joined via heatset threaded inserts.  Larger flat plate-like parts laser cut out of aluminum.  

<figure align="center">
  <img src="/assets/images/cubli/speaker_holder.jpg" width="250">
  <img src="/assets/images/cubli/connector_retain_2.jpg" width="250">
  <img src="/assets/images/cubli/connector_retain_3.jpg" width="250">
  <figcaption>some of my custom parts</figcaption>
</figure>

Here's a quick gif to appreciate ReM-RC's design: the flywheel runs clear and true

<figure align="center">
  <img src="/assets/images/cubli/cubli flywheel face design.gif" width="600">
  <figcaption>Flywheel face of cubli demonstrating flywheel clearance spin</figcaption>
</figure>

# Electronics 

Again, credit to ReM-RC for hardware scoping and design.  Also, a quick shoutout to oshwlab user [malebuffy](https://oshwlab.com/malebuffy/self-balancing-cube) for design of the cubli breakout board and saving me the headache of soldering a thousand pins myself on a DIY PCB. Manufactured by JLCPCB.  

- Motors: 3x NIDEC-24H BLDC motors. 2-channel Encoded.  PWM speed control. Binary braking and direction pins. The full datasheet is [here](../assets/pdfs/Nidec24H%20BLDC%20Product%20Data%20Sheet.pdf). 
- Sensor: MPG6050 6-DoF IMU on an Arduino breakout.  I2C bus. 
- CPU: ESP32-WROOM-32
- Power: 12.5V 650 mAh 3 cell LiPo battery (classic for drones)
- Peripherals
  - Voltage divider for battery sense
  - Buzzer for simple user information relay (eg: 1 buzz for calibration mode, 2 buzz for starting control loop, continuous beep for low-battery)

<figure align="center">
  <img src="/assets/images/cubli/cubli_breakout.jpg" width="400">
  <img src="/assets/images/cubli/old_circuitboard.jpg" width="400">
  <figcaption>Cubli breakout board.  On the right was the shit I had to deal with my first iteration</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/cubli/cubli_internal_motors.jpg" width="400">
  <img src="/assets/images/cubli/cubli_internal_breakout.jpg" width="400">
  <figcaption>Some pictures of my hardware and where they sit inside the fully assembled cubli</figcaption>
</figure>

# Firmware

**This section onward is where my own work truly begins and I diverge from ReM-RC**.

 In the past, I always flashed to ESP32s using the USB cable and also used the same cable as my data connection for Serial.  I would have gone crazy if I needed to do that for Cubli.  Not only because I iterate and flash frequently, but more critically because I would need IMU data streamed back to me without the mechanical attachment of a cable affecting Cubli's dynamics. Platform.IO is a way to flash over wifi and mosquiTTo is a way to publish and subscribe to OTA messages (not dissimilar to ROS). 

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

This is where the fun begins. A lot of dynamic modeling is analytical derivation. Full derivations can be found on the document linked below. On this page you will find key design decisions and results. 

## Nomenclature and Values (for quick reference)

### Experimental Values (Cubli intrinsics)

### Analytical Variables

## Frames of Reference

I defined 4 frames of reference for this project. All are outlined below. All are right-hand convention cartesian.
- $AF$: Accelerometer frame, the frame of reference with origin and axes aligned with the MPU6050's sensor axes.  Rotates with body
- $TF$: Torque frame, the frame of reference with axes aligned with the direction of torque applied by the wheels.  Rotates with body
- $O$: Inertial frame, sits at the pivot origin and is well... inertial
- $O'$: Body frame, initially indistinguishable from $O$, and is used as the primary FoR for cubli's state. Rotates with body

<figure align="center">
  <img src="/assets/images/cubli/FORs.jpg" width="600">
  <figcaption>The frames of reference for Cubli (side view). Flywheels shown in gray for reference</figcaption>
</figure>

A few extra notes for clarity
- $AF, TF, O'$ are all fixed relative to each other.  So the direction cosine matrix between each is static.
- $TF$ in the figure is centered on the CoM of cubli, but this is technically not a requirement, since torque is a free vector. 

## Desired State-Space Realization

At the center of EVERYTHING for cubli is its state space (SS) realization. Let's rip the band aid off.

$$
x = [\theta, \phi, \psi, \dot{\theta}, \dot{\phi}, \dot{\psi}, \tau_1, \tau_2, \tau_3]^T \in \mathbf{R}_{9\times 1}\\
y = [a_x, a_y, a_z, g_x, g_y, g_z]^T_{AF} \in \mathbf{R}_{6\times 1}\\
u = [\tau_{1c}, \tau_{2c}, \tau_{3c}]^T_{TF} \in \mathbf{R}_{3\times 1}\\
\dot{x} = [A_{9\times 9}]x + [B_{9\times 3}]u\\
y = [C_{6\times 9}]x + [D_{6\times 3}]u
$$

Getting the easy out of the way first, the definition for $y, u, A ,B, C, D$ should not require too much explanation.  
- $y$ is my observation (measurement) vector and is simply the 6 raw numbers output by the MPU6050.  $a_i$ is linear acceleration along the $i$th axis [kgm/s^2] and $g_i$ is the angular velocity about the $i$th axis [rad/s]. For clarity, I indicate that this vector is expressed in $AF$ using subscripts. 
- $u$ is my control input command vector and consists of the desired torque I want the 3 motors to output.  I indicate that this vector is expressed in $TF$, so each scalar term $\tau_{ic}$ corresponds directly with the output motor $i$. 
- $A, B, C, D$ complete my linear state space model.  Of course, at this point I haven't found them yet, and it will require "some" math, but they will soon be nicely defined. 

Justifying my choice of state vector definition $x$ requires deeper clarification and rationale. 
$

## Flywheel Inertia Calcs 

## Inertia Tensor Experiments
To ID my CoM I performed three independent drop tests along 3 axes from equilibrium positions.  $\dot{\theta}$ data is read straight off the IMU (currenlty debating whether I should implement a complementary filter to fuse IMU and accel data), integrated, or differentiated, and appled to $\ddot{\theta} = \frac{g}{l}\sin{\theta}$. I least-squares fit for the parameter $l$. 

## Measurement Model

# Torque-Based Motor Control



