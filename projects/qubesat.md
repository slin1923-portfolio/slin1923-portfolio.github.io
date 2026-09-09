---
layout: default
title: Qubesat
---

- [Qubesat 101](#qubesat-101)
- [The Team](#the-team)
- [Design](#design)
  - [Structures](#structures)
  - [Avionics \& Payload](#avionics--payload)
  - [Comms Hardware](#comms-hardware)
- [Manufacturing and Assembly](#manufacturing-and-assembly)
- [Testing and Analysis](#testing-and-analysis)
- [Integration and Launch](#integration-and-launch)
- [Aftermath](#aftermath)


# Qubesat 101

As a part of NASA's [cubesat launch initiative](https://www.nasa.gov/kennedy/launch-services-program/cubesat-launch-initiative/), universities from across the country are given the opportunity to design and construct their own cubically-modular satellites that will launch into low earth orbit and deliver student-designed payloads and collect research data. UC Berkeley has its own cubesat team which operates under the larger umbrella of the [Space Technologies at Cal](https://stac.studentorg.berkeley.edu/) student organization. Our quantum cubesat (*appropriately named Qubesat) is a technological demonstration to test and character­ize the effects of space conditions on quantum gyroscopes using nitro­gen-vacancy centers in diamonds. We unsuccessfully launched with Astra Space's ELaNa 41 Mission in January 2022.

<figure align="center">
  <img src="/assets/images/qubesat/promo.jpg" width="600">
</figure>

I have been a part of this project for 2 years and counting, starting as a mechanical team member and progressing to mechanical lead and now transitioning into project co-lead as our current project lead is graduating. If there was one project I could define my undergraduate experience so far with, it would be our Qubesat. Find below some brief documentation on our work!  For a consolidated, conference-style poster, click below. 

<div align="center">
  <a href="../assets/pdfs/stac-qubesat-2022.pdf" style="background-color: #0969da; color: white; padding: 8px 16px; border-radius: 6px; text-decoration: none; font-weight: bold; margin: 0 4px; display: inline-block;">Qubesat Poster PDF</a>
</div>

# The Team

<figure align="center">
  <img src="/assets/images/qubesat/cubesat_team.jpg" width="600">
  <figcaption>The STAC team watching launch at Cape Canaveral (sans me)</figcaption>
</figure>

Our project consisted of a standard set of sub-teams consisting of:
- Software: focusing on flight code, data collection, and signal processing.
- Payload: focusing on the theoretical side of our project involving the laser-diamond gyroscope system.
- Mechanical: focusing on the chassis, component housing, and dynamic systems such as antenna deployment and mechanical pins and switches.
- Electrical/Comms: focusing on designing PCB and antenna hardware to transmit smoothly. Also focus on power distribution and generation from solar panels.

Outside of the students, we also worked closely with:
- Nanoracks: The company focusing on integrating cubesats into rocket.
- Astra: Our launch-provider

# Design

## Structures

<figure align="center">
  <img src="/assets/images/qubesat/exploded_structure.png" width="800">
  <figcaption>4 side structures with trusses. Designed with very tight tolerance range in order to ensure fit with deployer. Designed to withstand structural stress of launch vibrations and acceleration. </figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/qubesat/Top Plate.png" width="300">
  <img src="/assets/images/qubesat/end plate.png" width="300">
  <figcaption>Top and End plate designs</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/qubesat/bird-eye.jpg" width="600">
</figure>

## Avionics & Payload

<figure align="center">
  <img src="/assets/images/qubesat/cubesat_transparent.jpg" width="600">
  <figcaption>Replica Demo Qubesat made with clear acrylic faces for internal visibility</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/qubesat/qubesat_open.jpg" width="600">
  <figcaption>Replica Demo Qubesat made with clear acrylic faces for internal visibility</figcaption>
</figure>

From Top to Bottom:
- Payload: a laser, a diamond, an Arduino, and a couple 3-D printed pieces that comprise our quantum gyroscope
- Magnetorquers: the primary control system on Qubesat. Basically 3 densely coiled electromagnets that align with the XYZ axes. When actuated, the magnets will align with earth's magnetic field and help us detumble when we are deployed from the spacecraft.
- Battery Board: Primary power source for when our Qubesat needs to be operational but does not have sunlight yet. Secondary power source in orbit. Space rated batteries that don't go boom in the atmosphere.
- Pycubed: The "CPU" of Qubesat. It is a third-party designed board that almost all cubesats use for its versatility.
- OpenLST: The primary board for comms. It is adjacent to our antenna and is connected via a coax cable.

<figure align="center">
  <img src="/assets/images/qubesat/solar-panels-on.jpg" width="600">
  <figcaption>Qubesat with the solar-paneled side PCBs installed</figcaption>
</figure>

Side:
- Solar Side PCBs: These PCBs were custom printed for our project. Solar panels were soldered onto the outward face and electrical contact areas on the inner face are connected to the power grid.

## Comms Hardware

<figure align="center">
  <img src="/assets/images/qubesat/antenna hardware.jpg" width="600">
  <figcaption>Final Antenna Deployment system with OpenLST pictured. Bulkhead on the left is used for external upload/download via USBC</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/qubesat/antenna_hardware_2.jpg" width="600">
  <figcaption>Better view of antenna housed within and burn wire release mechanism</figcaption>
</figure>

# Manufacturing and Assembly

For manufacturing, we used a combination of in-house methods, outsourced jobs, and off-the-shelf components. Managing the budget was the trickiest part.
- Protolabs: For high-precision and geometrically complex components such as the terrained end-plates, we outsourced our manufacturing jobs to Protolabs. Raw materials and shipping costs were all included in the order. This was by far the most expensive manufacturing method we used, but there were not many cheaper alternatives for the same quality.
- Bay Area Circuits: All custom PCB designs were sent to Bay Area Circuits, a local PCB manufacturer. These jobs were also expensive, especially when rush-ordered. Cost could have been saved if we had followed a tighter schedule.
- Anodization: Our surface finish for all of our aluminum parts were anodization. This was to prevent corrosion and electrical conduction on the surface of our parts. This job was also outsourced to a local business.
- Laser Cutters (Jacobs Makerspace): All 2-D components were laser cut in Berkeley's makerspace: Jacobs Hall. This was an extremely efficient and economic option. The only costs incurred were the stock sheet material purchased online from McMaster as. There was no time overhead either, since we cut them ourselves.
- Etcheverry Machine Shop: Another extremely efficient and economic option was to use the machine shops on campus. Most of the tools there required training, but we could submit job requests to the staff members and have simple brackets and rods made for free. Time overhead did exist, but usually jobs were completed quickly.
- 3-D printers (personal): 3-D printers were the go-to option for prototyping. Extremely fast, versatile, and most importantly free (aside from the filament). I could get parts made from home on my personal printer.
- McMaster Carr / Ace Hardware: All loose components such as nuts, bolts, washers, and standoffs OR stock material such as sheet metal and rods were purchased either online from McMaster Carr, or off-the-shelf from the Berkeley Ace Hardware. Both were cheap and convenient options for materials and components we needed in bulk quantity.

Here are some miscellaneous photos of the team during assembly. 

<figure align="center">
  <img src="/assets/images/qubesat/assembly_antenna.jpg" width="400">
  <img src="/assets/images/qubesat/Sean_assembly.jpg" width="400">
  <figcaption>Assembling the Antenna Deployment System</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/qubesat/krishna_test.jpg" width="400">
  <img src="/assets/images/qubesat/drake_test.jpg" width="400">
  <figcaption>Krishna and Drake</figcaption>
</figure>

# Testing and Analysis

<figure align="center">
  <img src="/assets/images/qubesat/vibe_test.png" width="600">
  <figcaption>Vibe Test: Mounting our satellite onto a launch vibration simulator at Berkeley's SSL</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/qubesat/fit_check.jpg" width="600">
  <figcaption>Fit Check: Vidish mounting our satellite onto the deployer to see if it fits snugly</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/qubesat/payload_tests.jpg" width="600">
  <figcaption>Experimental Payload Tests (that I don't know much about)</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/qubesat/thermal rod analysis.png" width="600">
  <figcaption>Steady State Thermal analysis of central rods (which are the primary heat-sinking mode of the avionics)</figcaption>
</figure>

# Integration and Launch

Integration happened on the first day of finals week. Luckily, the core team was free for most of the day. We visited Astra's HQ in Alameda, CA and met with cubesat teams from other universities and even the Johnson Space Center.]

<figure align="center">
  <img src="/assets/images/qubesat/core_team.jpg" width="600">
  <figcaption>The team that went to integration. Left to right: Drake, Vidish, Me, Michael</figcaption>
</figure>

<figure align="center">
  <img src="/assets/images/qubesat/astra_rocket.jpg" width="600">
  <figcaption>Us with the rocket that Qubesat will be flying on</figcaption>
</figure>

Two months after integration, we were invited back to the HQ to watch the launch.  It didn't go so well...

<figure align="center">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/8jlQmW7mnYM" title="ELANA 41 Mission" allowfullscreen></iframe>
</figure>

# Aftermath

Unfortunately, during launch, Astra's ELaNa 41 mission lost control after 2nd stage separation and only reached an apogee of around 400 km before releasing the cubesat payloads and dropping back to earth. Cubesats were deployed roughly 100 km short of target orbit and with nowhere near the orbital velocity necessary.

Despite this, I, along with the rest of my Qubesat team, wanted to express our gratitude to both Astra and NASA for giving us college kids an opportunity to send a satellite into space in the first place. We derived value from this project through the process, and the end result would have just been a perk. This project brought together a group of students united under a common interest in space and turned them into an effective team of engineers, but more importantly an inseparable group of friends. Though it is cliche, I am sure that decades down the road, I won't remember the day we failed to reach orbit. I will remember biking home at 2 AM after an evening of assembly in our makerspace. I will remember losing my mind over the success rate of our antenna deployment system. I will remember all the laser cutting, 3-D printing, and finger-callusing grunt work that turned our idea into reality. And I will certainly remember all the interactions I had with my teammates, whether that be arguing over design merits or grabbing burritos after a meeting, that built us all up to be better team players and engineers.

To Astra, I say "don't worry about it". To us, rockets are fun, to you it is your livelihood. Thank you for providing us with the opportunity to launch in the first place. Thank you for welcoming us into you headquarters and showing us the behind-the-scenes of getting to space. Most importantly, thank you for giving us our first taste of "Space is Hard". Failure, especially in the industry, is almost inevitable, and as aspiring engineers in the field, we must learn to deal with it. Your resilience and perseverance will serve as an inspiration for when we run into our own obstacles in the future.

Fortunately, Qubesat is not history! Here is where we're headed:
1. STAR launch collaboration: STAR (Space Technologies and Rocketry) is a sister club here at Berkeley that focuses on launching and recovering their own single-stage rockets. We will be providing a payload similar to Qubesat. For us, any launch opportunity is welcome, and for them, they are competing for the "coolest payload". We will be launching to an altitude of 10,000 feet and recovering our payload in summer 2022.
2. Qubesat v2: Fortunately, NASA's CSLI team has granted us the opportunity to send an exact replica of Qubesat with the next available launch provider. Basically, this will be a re-do launch for us! Details tbd.
3. Cubesat v2: Our long-term vision is to create another cubesat with a new payload. We will be brainstorming ideas for new payloads (such as solar sails or space junk collectors) and submitting a proposal in Fall of 2022 for the next CSLI cohort.
