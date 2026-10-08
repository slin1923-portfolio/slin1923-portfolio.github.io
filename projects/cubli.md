---
layout: default
title: Cubli
---

- [Todos + Current Issues (for quick personal reference)](#todos--current-issues-for-quick-personal-reference)
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
  - [Dynamics Model $A, B$](#dynamics-model-a-b)
  - [Measurement Model $C, D$](#measurement-model-c-d)
  - [$I\_w$ Calcs](#i_w-calcs)
  - [Finding the $I\_c$ tensor and $r\_{CM}$](#finding-the-i_c-tensor-and-r_cm)
    - [Finding $I\_{zz}$](#finding-i_zz)
    - [Finding $I\_{xx}$ and $r\_{CoM}$](#finding-i_xx-and-r_com)
- [Inner Loop Motor Torque Control](#inner-loop-motor-torque-control)


# Todos + Current Issues (for quick personal reference)

- Finish torque-based motor inner-loop control
  - Need to fit data to a first-order model of motor output torque
  - switch to Serial from MQTT for high-frequency data collection fidelity
  - Characterize $t_{TC}$ for reaching torque target
- Characterize $I_{zz}$ of cubli using string suspended spin test
- Migrate state space realization into Simulink and intuitively verify system behaves as expected 
- Sanity check the measurement model using real world experiment and validate $C$ with collected data
- Big live telemetry progress lately!
  
<figure align="center">
  <img src="/assets/images/cubli/cubli_telemetry_clip.gif" width="600">
  <figcaption>Controlled torque pulses with live telemetry of control input and gyroscope measurement.</figcaption>
</figure>

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

This is where the fun begins. A lot of dynamic modeling is analytical derivation. Full derivations can be found on the document linked below. On this page you will find key design decisions and results. Another large chunk of modeling is empirical, and involve experimental design and data collection.  You will also find those experiments and their results here. 

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
\begin{align*}
x &:= [\theta, \phi, \psi, \dot{\theta}, \dot{\phi}, \dot{\psi}, \tau_1, \tau_2, \tau_3]^T \in \mathbf{R}_{9\times 1}\\
y &:= [a_x, a_y, a_z, g_x, g_y, g_z]^T_{AF} \in \mathbf{R}_{6\times 1}\\
u &:= [\tau_{1c}, \tau_{2c}, \tau_{3c}]^T_{TF} \in \mathbf{R}_{3\times 1}\\
\dot{x} &= [A_{9\times 9}]x + [B_{9\times 3}]u\\
y &= [C_{6\times 9}]x + [D_{6\times 3}]u
\end{align*}
$$

Getting the easy out of the way first, the definition for $y, u, A ,B, C, D$ should not require too much explanation.  
- $y$ is my observation (measurement) vector and is simply the 6 raw numbers output by the MPU6050.  $a_i$ is linear acceleration along the $i$th axis [kgm/s^2] and $g_i$ is the angular velocity about the $i$th axis [rad/s]. For clarity, I indicate that this vector is expressed in $AF$ using subscripts. 
- $u$ is my control input command vector and consists of the desired torque I want the 3 motors to output.  I indicate that this vector is expressed in $TF$, so each scalar term $\tau_{ic}$ corresponds directly with the output motor $i$. 
- $A, B, C, D$ complete my linear state space model.  Of course, at this point I haven't found them yet, and doing so will be sooo fun. 

Justifying my choice of state vector definition $x$ requires deeper clarification and rationale. 
- **$\theta, \phi, \psi$ correspond to $X-Y'-Z'$ euler angles respectively**, and (assuming a stationary pivot point such that $O$ and $O'$ origin remain coincident) track the attitude of $O'$ relative to $O$ and hence Cubli's attitude. 
- I did not use quaternions even though they are standard practice in aerospace because I am solving a regulation problem where my system stabilizes at $\theta = \phi = \dot{\theta} = \dot{\phi} = 0$, so I felt quaternions for the sake of avoiding a singularity (which occurs at $\phi = 90^o$ for this particular Euler Angle convention) I never expect to come close to would be unnecessarily complicated.  Also I still kinda fear quaternions because they are so physically unintuitive. 
- $\psi$ is a state I leave free.  At the balancing point, $\psi$ will look like cubli's yaw angle. Cubli needs to be able to track external yaw angles provided by me while maintaining balance. 
- I augmented my state with $[\tau_1, \tau_2, \tau_3]$ even though they are not intrinsically encoded within cubli's free dynamics because otherwise, $D\neq 0$ which is something I am trying to avoid. In general, it is standard practice to have $D =0$ since not doing so means when feeding back on the error term, $u' = K(y(u) - x)$, meaning the new control input becomes a function of the old control input, which can cause a host of problems in an unideal system with noisy actuators. Why $D\neq 0$ without the augmented states is not immediately obvious but becomes clear when you start actually trying to derive the measurement model $C$.  The reasoning is
  - $a_i \in y$ is certainly dependent on the angular acceleration of the cubli ($\ddot{\theta}, \ddot{\phi}$).
  - $\ddot{\theta}, \ddot{\phi}$ would both be dependent on $u$ in an unaugmented state vector definition.  
  - this makes $a_i$ indirectly a function of $u$, which is a nono
- I want to emphasize that $\tau_i$ in $x$ is different from $\tau_{ic}$ in the $u$.  $\tau_i$ is the torque that motor $i$ is CURRENTLY outputting, while $\tau_{ic}$ is the new torque COMMAND (hence the $c$ subscript) the controller is telling motor $i$ to achieve.  There are empirical methods to ID this first-order transient behavior between $\tau_i$ and $\tau_{ic}$ which are touched on later. 

## Dynamics Model $A, B$

Now let's derive $A$ and $B$ matrices. This is where I hide all the algebra, rotation matrices, and partial derivatives in my pdf document and spare you the eyesore.  In short, the process is

1. Convert Euler Angle rates $[\dot{\theta}, \dot{\phi}, \dot{\psi}]$ to a body-frame angular velocity $\omega_{O'}$. 
2. Set up and solve the resulting Lagrangian Mechanics problem which will yield a set of equations where $\ddot{\theta} = f(x)$. And likewise for $\ddot{\phi}$ and $\ddot{\psi}$. 
3. Take a small-angle approximation on $\theta, \phi$.
4. Jacobian linearization about ($\theta = \phi = \dot{\theta} = \dot{\phi} = 0$)

The final system dynamics written out explicitly is

$$
\begin{bmatrix}

\end{bmatrix}
$$

## Measurement Model $C, D$

Unlike dynamics modeling, measurement modeling is strictly a kinematics problem.  Again, there is no shortage of mathematical gymnastics that you need to find in my pdf, but most of the work here is simply knowing how to apply the golden rule of rotational kinematics

$$
a_r = a_s - \alpha \times r - 2 (\omega \times v_r) - \omega \times (\omega \times r)
$$

The final measurement model written out explicitly is

$$

$$

## $I_w$ Calcs 

Analysis is done (for now) and I need to get to know Cubli nice and personal. Low hanging fruit is the rotational inertia of the flywheel.  I need this because eventually I am going to allocate $\alpha_i$ from my motor encoders to $\tau_i \in u$ and to do this I need $I_{w}$ to apply $\tau_i = I_{w} \alpha_i$. 

Honestly this step is so trivial I am just going to spam you with my hand-calcs and some pictures I took for documentation purposes.  $\boxed{I_{w} \sim 2.3036*10^{-4} ~\text{kg m}^2}$. 

<figure align="center">
  <img src="/assets/images/cubli/flywheel_inertia_calcs.jpg" width="700">
  <figcaption>Hand Calcs</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/cubli/flywheel_inertia.jpg" width="600">
  <figcaption>Onshape Mass Properties analysis with overridden mass</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/cubli/flywheel_inertia_2.jpg" width="600">
  <figcaption>dimension measurements</figcaption>
</figure>

## Finding the $I_c$ tensor and $r_{CM}$

The problem of finding the inertia tensor of cubli has allowed me to exercise some creativity in experimental design. Since all of my kinematics/dynamics are calculated wrt $O'$, that is also how I define $I_c$. Now ideally I would have a perfect CAD model with all the right mass properties set or overridden such that a simple computer evaluation would give me $I_c$, but I do NOT have this luxury (this is something I may do in the future ONLY if necessary aka I need a more precise inertia tensor). 

The overall problem is that I need to find
$$
I_c = 
\begin{bmatrix}
I_{xx} & I_{xy} & I_{xz}\\
I_{yx} & I_{yy} & I_{yz}\\
I_{zx} & I_{zy} & I_{zz}
\end{bmatrix}
$$

Where $I$ is symmetric by nature so that $I_{ij} = I_{ji}$, leaving only 6 unique terms to find.  BUT, I make the relatively safe assumption that Cubli is rotationally trisymmetric about the z-axis of $O'$, implying $I_{xx} = I_{yy}$ and also that the z-axis of $O'$ is a principal axis of rotation! This simplifies $I$ to be 

$$
I_c = 
\begin{bmatrix}
I_{xx} & 0 & 0\\
0 & I_{xx} & 0\\
0 & 0 & I_{zz}
\end{bmatrix}
$$

Voila!  I need only to identify 2 inertia terms of my Cubli! Observing $A$ we also see that I will need $r_{CM}$ the location of the center of mass of cubli, which per the rotationally symmetric assumption, should also lie on the z-axis of $O'$. The value hunt begins!

<figure align="center">
  <img src="/assets/images/cubli/cubli_inertias_figure.jpg" width="700">
  <figcaption>Diagram of the values I am trying to find and their references. Geometry shown on the right illustrates how cubli can be approximated as some equivalent cylinder due to symmetry.</figcaption>
</figure>

### Finding $I_{zz}$

$I_{zz}$ is the relatively easiest value to find.  Since I know $I_w$ and all 3 motors are encoded, 

### Finding $I_{xx}$ and $r_{CoM}$

*havent gotten around to documenting yet, but here's a sneak peak*

<figure align="center">
  <img src="/assets/images/cubli/cublii_swing_clip.gif" width="600">
  <figcaption>A series of controlled swings gives me all the information I need</figcaption>
</figure>

# Inner Loop Motor Torque Control



