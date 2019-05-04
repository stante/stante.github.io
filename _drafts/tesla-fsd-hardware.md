---
layout: post
title:  "Tesla's Full Self-Driving Computer"
subtitle: "An overview of the hardware of Tesla's full self-driving computer which was presented during the Tesla Autonomy Investors Day on the 22nd of April 2019."
date:   2019-04-27 10:55:06 +0200
thumbnail: "assets/thumbnail.png"
categories: self-driving
---

Tesla invited investors to their first "Tesla Autonomy Investors Day"
which was held on the 22nd of April 2019 at the Tesla Headquarters in
Palo Alto[^1]. According to Tesa, the intention of the event was to
narrow the gap in perception of what can be seen outside of Tesla
related to autonomous driving and the actual developments within the
company. This article summarizes the presentation given by Pete
Bannon.

## Full Self-Driving Computer
Tesla was working on a new hardware as the basis for the new full
self-driving capabilities of the Tesla cars which is correspondingly
called "Full Self-Driving Computer".

The main component of the "Full Self-Driving Computer" is the "Full
Self-Driving Chip" which was developed in-house by Tesla. The chip was
designed in 18 months and in July 2018 mass production started. The
new hardware replaces the HW2.5, which is based on NVIDIA's Drive
PX2[^2] hardware. Since March 2019 the old HW2.5 got replaced in the
Model S and X and since the beginning of April 2019, all Model 3 are
equipped with the new hardware as well.

The FSD hardware was designed to be able to be retrofit into existing
cars, making it necessary to have a power consumption of less than
100W. Performance wise Tesla estimated that at least 50 TOPS (billion
operations per second) are necessary to drive a car. The FSD chip was
tailored for one trait, which is autonomous driving. It was optimized
to work with a batch size of 1 to have very low latency and it is used
for neural network inference only and **not** neural network learning.
- Full Self Driving Computer
  - 18 month spend for design
  - July 2018 production started
  - Autonomous driving stack running on new hardware
  - March 2019, hardware shipped in model s and x and April in model 3
  - Advantage of custom design is to focus on only Teslas requirements
	- Requirement power consumption less than 100 Watts to retrofit into existing cars
	- Lower part costs to enable redundancy
	- At least 50 TOPS of neural network performance (estimate to drive a car)
	- Batch size of 1 (Google TPU as 256), which makes it low latency for inference
	- Modest GPU for post processing, bet was neural nets get better and better
	- Security is a focus. Without security no safe car.
	- Saftey
  - Standard Industry IP for GPU and CPU
  - On the right all video connectors for the cameras from the car
  - Power and control on the left
  - Dual redundant FSD computer
  - DRAMs left and right, flash bottom
  - Independent with own operating system
  - FSD Computer can fail, redundant Power Supply can fail(each chip has own), Cameras can fail (half on each power supply), still driving. Key metric, probablity of failure is an order of magnitude smaller than someone looing conciousness, key metric.
  - All inputs (Camera, GPS, Maps, IMU, Ultrasonic, Wheel ticks, Steering Angle) are then used to make a plan
  - Computers exchange the plan to make sure both have the same plan
  - The send signals can be validated by reading sensors and checking if the send commands (like accelerating and accelrometer) where actually done
  - Lot of reduandancy in data aquistion and monitoring

## Full Self-Driving Chip
- Full Self Driving Chip
  - FSD Package, 37.5mm x 37.5mm BGA, 2116 balls
  - 12,464 C4 bumps, 12 metal layers
  - 14nm FinFET CMOS process
  - 260mm2 (cell phone 100mm2, high end gpu 600mm2-800mm2)
  - 250 million gates
  - 6 billion transistors
  - AEC Q100 standard automotive criteria
  - Camera Serial Interface: Video Input 2,5 G pixes/s (more than enough for all cameras)
  - LPDDR4 @ 4266 Gb/s, 68 GB/s
  - Image Signal Processor, 1 G pixel/s, 24-bit pipeline, Advanced tone mapping (brings out details in shadows), Advanced noise reduction
	- Full advantage of the hdr sensors around the car (HDR cameras?)
  - Neural Network Processor
	- 2 Per chip
	- 32 MB SDRAM
	- 96x96 Mul/Add array with in-place accumulation ~ 10,000 ops per cycle
	- ReLU Hardware
	- Pooling Hardware
	- 36 TOPS @ 2 GHz
	- 72 TOPS total (exceeding 50 TOPS requirement)
  - Video Encoder
    - H.265 Encoder. Used at various places, for example backup camera.
	- User Dash Cam Feature
	- Clip logging to the cloud (Andrej)
  - GPU
    - 1 GHz
	- 600 GFlops
	- FP32, FP16
  - Main Processor
    - 12 ARM A72 64b CPU @ 2.2GHz
	- Represents 2.5 times the performance of the current solution?
  - Saftey System
    - Lock Step CPU (two CPUs that run in lock step)
	- Final Drive Control (Final arbitrator to decide if it is safe to drive the actuators, the two calculated plans)
	- Control Validation
  - Security System
    - Ensure that only software is executed that was signed by Tesla
  - Number in perspective
    - Neural Network of Narrow camera uses 35 TOPS
    - 1.5 frames per second on CPU
    - 17 frames per second on GPU
    - 2100 frames on NNA

## Neural Network Accelerator

|Operation |MOPS |%   |
|:---------|:----|:---|
|Convoltion|45123|99.2|
  - 99.7% operations are MADD2
  - Temporary data can be kept in SRAM, no need to store in DRAM
  - 32 bit add operation
  - 8 bit mul opration
  - SRAM instead of DRAM
  - 0.15% of power is actual add, rest is control, therefore minimize control (icache, register file, control)
  - Processing Steps (each cylce)
    - NNA is dominated by 32MB of SRAM banks
    - Read activation from formatter 256 bytes
	- Weight data is read 128 byte
	- Data combined in muladd array
	- 9216 multiply add
	- 2 GHZ
	- 36.8 TOPS
	- data is unloaded to relu unit and optionally pooling unit
	- data aggregated to write buffers
	- data stored to SRAM (128bytes per cycle)
  - Bandwidth
    - Read 128 and 256 bytes per cycle
	- Write 128 bytes
  - Instructions
   - Eltwise is element wise
  - Neural Network Compiler
    - Can take existing network (like for older cars)
	- Performs layer fusion
  - Programming model
    - Set input buffer address (presumable image from camera)
	- Set output buffer address
	- Set pointer to network weights
	- Execute program (running for 1 to 2 million cycles), interrupt when done

## Conclusion / Results

- Planned to be hold regularly for updates regarding autonomous driving
- New full self driving chip
- Complete overhaull of neural net for vision recognition
- Goal was to find best chip for autonmous driving. No chip was designed from ground up for neural nets. 
- Pete Bannon, VP of Silicon Engineering to desing chip.
  
  
- Results 
  - Power (running full autopilot stack driving around)
	- HW 2.5 57 Watts
    - FSD 72 Watts
	- NNA 15Watts
  - Relative Costs
    - 80% of HW2.5
  - FPS
    - 110 with HW2.5
	- 2300 with FSD (all 4 accelerators)
  - Comparison with drive xavier
    - 144TOPS vs 21 TOPS
- Q&A
  - Other Activation functions? Tanh and sigmoid
  - Why 14nm? When design started not all IPs were available in 10nm.
  - Musk: Design finished 1.5 year before, new design is ongoing
  - Musk: Tesla solution is tailored to one trade, self driving. Nivida has many customers and makes general solutions.
  - Musk: Lidar is a fools errand. Everyone relying on lidar is doomed.
  - What is primary objective of next generation chip(1:43)? Pete whispers safety. 3 times better than current system. Two years away.
  - Cost savings pays for the development
  - Greatly enhance the functional capabilities in next generation
  - Fabin at Samsung in Austin, Tx
  - Dozen of patens are filled
  - Constant update of NN weights
  - 8 cameras, 12 Ultrasonics
  - Even with old hardware, tesa has the data gatering ability
  - Nvidia and Waymo talk with a lot of confidence because of their simulation capabilities. Answer: Simulatoins do not capture the weirdness of the real world.

[^1]: [Tesla Autonomy Investors Day](https://ir.tesla.com/events/event-details/tesla-autonomy-investor-day)
[^2]: [Look inside Tesla’s onboard Nvidia supercomputer for self-driving](https://electrek.co/2017/05/22/tesla-nvidia-supercomputer-self-driving-autopilot/)
