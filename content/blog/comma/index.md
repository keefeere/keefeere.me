---
title: "Installing Comma.ai OpenPilot on Honda Clarity PHEV"
date: 2025-08-28
draft: false
tags: ["Honda Clarity", "PHEV", "Comma.ai", "OpenPilot", "DIY", "Car Mods"]
description: "A step-by-step guide to installing Comma.ai's OpenPilot system on the \
Honda Clarity PHEV with practical notes and driving impressions."
---

# Installing Comma.ai OpenPilot on Honda Clarity PHEV

## Introduction

![Sunset Comma view](/blog/comma/resources/IMG_3878_wide.jpg)

From the moment I bought my car, the windshield had been damaged, and it was
replaced during the 0th service. Unfortunately, this caused the camera calibration
to fail, and as a result, the lane-keeping system never worked for me all these years.

Recently, someone in a chat reminded me about
[comma.ai](https://comma.ai/) and their current development status.

I had seen their early attempts a long time ago, back when they were trying to
control a Prius using a bunch of web cameras taped together with wires everywhere.
Later, they moved to a phone mounted on the windshield. Now, they offer a
"nearly finished" [solution](https://comma.ai/shop/comma-3x) — a full Linux
computer with three cameras on board and a wiring system that plugs directly
into the factory camera harness, no soldering needed. It intercepts the factory
camera control signals and "drives" the car!

I’m glad my camera was not working, because otherwise, I might never have bought
this autopilot.

## Purchase & Setup

Buying the system was complicated — everything had to be ordered from the USA and
customs clearance was a hassle. But the installation itself is quite simple and
took me about an hour.
The key is to install the
[Proxy Board](https://shop.retropilot.org/product/honda-clarity-proxy-board-kit/)
in-line with the CAN gateway (a box behind the infortainment screen) and connect
the wiring harness in line with the camera connector. The small
[Harness Box](https://comma.ai/shop/harness-box) connects here, managing CAN line
switching when the autopilot computer is plugged or unplugged, and converts the
physical interface to OBD-C — encapsulating OBD signals into a Type-C interface.

I expected a slightly more advanced cruise control and lane-keeping, but got much
more. Below, I outline the advantages compared to stock or my assumptions about it.

## Pros

1. **City driving support:** Lane keeping works all the time when cruise is on,
   even if speed hold is off. You can drive in the city, let go of the wheel, and
   the car stays centered in the lane. Unlike stock, it doesn't nag you to hold
   the wheel unless there are sharp maneuvers.
2. **Automatic lane changes:** Indicate your turn and gently nudge the steering
   wheel; the system understands you want to change lanes and does so smoothly.
   Note: Since there are no blind spot sensors, you still need to watch your
   surroundings.
3. **Offline OSM maps:** The system loads offline OpenStreetMap data and knows
   speed limits by location — cities, villages, and so on. It automatically
   adjusts speed limits on cruise, slowing down when entering towns even if no
   cars are ahead.
4. **Advanced distance sensitivity:** It keeps a safe distance, not tailgating
   until radar triggers braking. It detects cars far ahead and slows down
   proactively.
5. **Traffic jam behavior:** No "Stopped" message requiring you to press buttons
   or gas. The car waits as long as needed in traffic and moves when the car ahead
   moves, with no timeout that I've noticed.
6. **Driver monitoring:** If you fall asleep, the system beeps and gently brakes,
   disabling cruise control.
7. **Phone usage detection:** It detects when you use your phone and alerts you.
8. **Speed reduction on curves:** After some calibration, it can reduce speed
   based on map or visual data for sharp turns. Not always perfectly early, but on
   winding rural roads, I hardly had to adjust cruise speed manually.

## Cons

The biggest downside for my car is the steering column is limited to a very small
torque the autopilot can apply. It won’t handle tight corners like hairpins or
90-degree turns and will constantly ask you to take the wheel. This can be fixed
with a [patched](https://wirelessnet2.medium.com/eps-fw-modifications-for-the-honda-clarity-39990-trw-a020-beta-373b3e7ba528)
EPS firmware, which I plan to try.

There is also a feature called the
[Comma Pedal](https://www.etsy.com/de-en/listing/952895642/openpilot-comma-pedal-toyota-honda-gm-vw?ls=s&ga_order=most_relevant&ga_search_type=all&ga_view_type=gallery&ga_search_query=beartech+honda+pedal&ref=sr_gallery-1-9&nob=1&content_source=6d0515b4-a6f2-4555-b42f-53c31f66a636%253Acbee4559f0abd99bdc3b20ad887711154c52014f&organic_search_click=1&logging_key=6d0515b4-a6f2-4555-b42f-53c31f66a636%3Acbee4559f0abd99bdc3b20ad887711154c52014f&variation0=4797921102)
— a gas pedal interceptor that lets the car accelerate faster than the stock
cruise control allows. System can work with stock ACC, but people says that pedal
is better.

## Installation Guide

### Parts needed

- [Comma 3X Device](https://comma.ai/shop/comma-3x) (without car harness)
- [Comma Harness Box](https://comma.ai/shop/harness-box)
- [Proxy Board](https://shop.retropilot.org/product/honda-clarity-proxy-board-kit/)
  to insert into the CAN gateway; sold with wiring (harness) for Honda Clarity

![Comma in box vertical](/blog/comma/resources/IMG_3846.jpeg)
![Comma in box horizontal](/blog/comma/resources/IMG_3847.jpeg)
*Comma with proxy board and harness*

### Tools required

- Phillips cross screwdriver
- Pry tool

### Step-by-step installation

1. Remove the panel under the infotainment screen using pry tool.  
2. Remove the infotainment screen itself (two screws under the panel).
3. Pull off the side kick panel and the panel above the passenger’s feet. Be
    careful — some wires for footwell lighting are taped here.  
4. Remove the panels above the infotainment screen — they are clipped in place.
   Use the pry tool carefully and pull towards yourself.  
5. Remove the glove box (carefully, there are light connectors too):
    - One screw inside the glove box behind a cover
    - Two screws underneath the glove box
    - Three screws on top of the glove box  
6. Locate the CAN gateway behind the infotainment screen:
    ![Place of can gateway](/blog/comma/resources/IMG_3853.jpeg)
    *CAN gateway behind this panel*
7. Proxy board connects behind CAN gateway. Orientation does not matter it is
    not polarized.
    ![Can gateway and proxy board](/blog/comma/resources/IMG_3852.jpeg)
    *Proxy board connected*
8. After connecting the proxy board, turn on the car and check for errors.
   The light on the proxy board should be on; it is visible through the white plastic.
9. *Optional* Wrap Proxy board with fabric tape.
    ![Proxy board wraped with tape](/blog/comma/resources/IMG_3855.jpeg)
    *Proxy board wrapped*
10. Zip tie proxy board to free ear of the bracket near the gateway
    ![Proxy board zip tied](/blog/comma/resources/IMG_3856.jpeg)
    ![Place of zip tie](/blog/comma/resources/IMG_3857.jpeg)
    *Where zip tie goes*
11. Before assembling the trim, check again for errors by turning on the car.
12. Remove the camera cover — it comes off by sliding it upwards toward the
    headliner, then down towards you.
13. Find the camera connector located behind the rearview mirror. The car must
    be off when you unplug it.  
14. Disconnect the camera connector — there is a clip on the connector, gently
    pry and pull it off.  
15. Connect the Comma harness to the camera wiring (connectors are different
    male/female, so no mix-up).  
    ![Comma with harness](/blog/comma/resources/IMG_3851.jpeg)
    *Comma with harness*
16. Connect the Harness Box to the harness wiring. Wrap the harness if you wish
    ![Camera harness](/blog/comma/resources/IMG_3867.jpeg)
    ![Camera harness wrapped](/blog/comma/resources/IMG_3868.jpeg)
    *Camera harness wrapped, harness box is under the harness*
17. Cut some part of base to make type-c cable free out
    ![TypeC out](/blog/comma/resources/IMG_3869.jpeg)
    *Type-C out*
18. Make cut-out in Camera cover
    ![Camera cover cut-out](/blog/comma/resources/IMG_3870.jpeg)
    *Camera cover cut-out*
19. Link the Harness Box to the Comma device via the Type-C cable.  
20. Mount the Comma device in the center of the windshield as high as possible,
    considering you will need to remove it upward and the camera housing might
    interfere.  
    ![General view](/blog/comma/resources/IMG_3872.jpeg)
    ![General view 2](/blog/comma/resources/IMG_3873.jpeg)
    *Eventually you should get something like this*
21. Not need to connect the Proxy Board to the Harness Box — they communicate
    over CAN.  
22. Reassemble everything in reverse order.  
23. Turn on the car; the Comma device interface will appear. Connect to your
    Wi-Fi and select the OpenPilot branch.
    - I used *[sunnypilot/release-c3](https://github.com/sunnypilot/sunnypilot/tree/release-c3)*
      -- this review is based on that branch.
    - You literally just type ‘sunnypilot/release-c3‘ for example, into the
    URL field.  
24. Optionally, register your device on the Comma [Connect](https://connect.comma.ai)
    website and link it to your account.  
25. Optionally, download offline OSM maps in OpenPilot settings.  
26. Calibrate the camera by driving straight at 40 km/h or faster for about a
    minute.  
27. Start using the system as you would normally use Adaptive Cruise Control  
    (ACC): press **SET** to activate cruise, use **+/-** to adjust speed, and  
    **CANCEL/RESUME** to control it. When ACC is engaged, Lane Keeping Assist  
    (LKAS) will automatically activate. In Sunnypilot, LKAS will also remain  
    active even if ACC is disengaged - thanks to MADS feature. Most drivers will
    not need to use the **Main** button; typically it is enough to press the
    LKAS button once per drive, after which OpenPilot will manage it
    automatically when ACC is on.
28. Enjoy your enhanced driving experience!

![Sunset Comma view](/blog/comma/resources/IMG_3878.jpeg)

## Additional Notes

The standard OpenPilot release does not officially support the Honda Clarity. To
use it, you need to rely on forks maintained by the community. For example, there
is a stock-like fork by *vanillagorilla* that adds Clarity support (but I can't
manage to run it), or more feature-rich custom forks that extend compatibility
across many vehicles and provide additional functionality.
A comparison of popular branches can be found
here: [OpenPilot Branch Comparisons](https://bderkhan.com/openpilot-branch-comparisons/).

For those new to the ecosystem, a helpful glossary of terms and phrases is
available at [Comma OpenPilot Terms and Phrases](https://bderkhan.com/comma-openpilot-terms-and-phrases/).
The site also contains plenty of in‑depth resources and guides on the topic.

## Conclusion

Installing Comma.ai’s OpenPilot on the Honda Clarity PHEV adds significant
improvements over the stock system, especially in city driving, lane changing,
and adaptive speed control. While some limitations remain due to hardware
constraints like steering torque, the system is a powerful upgrade that enhances
safety and convenience. The installation is straightforward for those
comfortable with basic disassembly and wiring, and the open-source nature allows
for further customization and improvements.
