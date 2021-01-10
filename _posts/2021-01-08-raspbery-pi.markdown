---
title:  "Raspberry Pi"
date:   2020-01-09 00:00:00 +0000
categories: raspberrypi
tags:
    - compute
    - raspberry pi
    - single-board computers
classes: wide
toc: true
header: 
    overlay_image: /assets/images/raspberry-pi-logo.jpg 
    overlay_filter: rgba(0, 0, 0, 0.4)
published: true
---
A brief introduction to the Raspberry Pi Foundation and their latest single board computer.

**What is Raspberry Pi?**

The Rapsberry Pi foundation is a educational charity founded by [Eben Upton](https://en.wikipedia.org/wiki/Eben_Upton) in 2008. Its humble roots began as a means of inpsiring young children in an approachable yet practical manner to turn to 'computers'. In the hopes that 

generations will pursue studying computer science in further education, thus filling the large skills gap at the time. In essence the foundation produces low cost single-board comptuers (a complete computer built on a single circuit board) but, since its first public release in 2012 it has become much more than an electronics manufacturer. After the intial release of the Raspberry Pi the foundation had huge [success](https://www.theguardian.com/technology/2015/feb/18/raspberry-pi-becomes-best-selling-british-computer), the product sold out and had an amazing [community](https://www.raspberrypi.org/community/) response. Today, the Pi foundation has gone above and beyond what it first set out to do, now at the forefront of a huge [global movement](https://www.raspberrypi.org/stories/), inspiring and helping young people all over the world learn about computing. Code Club and CoderDojo are part of the foundation althought these programs are separate from the Raspberry Pi hardware, these are clubs aim to ensure every child has the opportunity to access learning about computing.

Raspberry Pi operates in the open source ecosystem; running linux, its main supported OS Raspbian and a huge range of open source software. As a result, the Pi is not only used as a tool for education but by users who are looking to harness the control the Pi gives you over the device - they are used as media centres, web servers, game emulation machines, IoT devices, robotics and more.

[Here](https://www.notion.so/Raspberry-Pi-4da2759f0f6f484da66381db1c7639ba) is a great interview of Eben Upton by Retro Man Cave (RMC) where Eben talks in length about the evolution of the Pi foundation, from its fantastic origin story to where the foundation sees itself in the future.

**Technical Specifications of Raspberry Pi**

Currently, there a wide variety of different Raspberry Pi computers to choose from. Each one fits a specific purpose be it price, size, compute power and accessiblity. I'll mainly be focusing on the Rapsberry Pi 4B which at the time of writing this is the latest and most powerful release from the foundation. Each Pi however, has at its core, like any 'normal' computer, a cpu, gpu, ram which make up the compute processes of the device. Where each generation of Pi compute units see huge improvements to performance, device architecture, memory speeds and display output.

The specs of the latest Pi model, the Raspberry Pi 4B, it has a 1.5-Ghz Broadcom quad-core Cortex-A72 (BCM2711B0) CPU and ARM VideoCoreVI GPU, LPDDR4 RAM (from 2GB to 8GB), 2 USB 3.0 ports, 2 USB 2.0 ports, dual micro-HDMI ports, 3.5mm audio jack, full-sized gigabit ethernet port, 802.11ac dual-band Wifi, Bluetooth 5.0 and 50 mb/s micro SD card reader. Atop the board there is a Camera Serial Interface and a Display Serial Interface. As well as all this, the best part of any Pi is its set of 40 GPIO pins; these allow you connect lights, fans, motors, sensors and a range of HATs (expansion boards that sit atop the Pi). The device is powered over USB Type-C and only requires a steady 3 amps of power and 5 volts.

Raspberry Pi 4B (mine has small heat sinks on the Pi for cooling):
![A picture of my Raspberry Pi 4B](/assets/images/raspberry-pi4b-1.jpg)

**Operating sytem** 

The Rapsberry Pi foundation, along with the Open Source commuity, have developed a number of [operating systems](https://www.raspberrypi.org/software/operating-systems/#raspberry-pi-os-32-bit) for their devices; their standard Raspberry Pi OS, previously known as Raspbian OS, being based on Debian and is extremely well optimised for the Raspberry Pi hardware. Raspberry Pi OS comes with over 35,000 packages: pre-compiled software bundle in a format that makes for easy installation and Bookshelf: a collection of the Free PDFs released by the publishing company Raspberry Pi Press which automates automatically for each new release from a number of their press outlets; MagPi, Hackspace and Wireframe. All of which contain useful information on the latest computing hardware, projects and tutorials.

A number of 3rd parties have developed well optimised Raspbery Pi Operating Systems, this allows for complete flexibility on the use of your Raspberry Pi. A definitive Raspberry Pi compatible OS list can be found [here](https://raspberrypi.stackexchange.com/questions/534/definitive-list-of-operating-systems). 

As the raspberry Pi boots from the Micro SD inserted into the card reader, this means you can have different OS' on different micro SD cards  from that allows you to have multiple uses for a single Pi or have multiple Pi's with a single use (see [Kubernetes on Pi](https://ubuntu.com/tutorials/how-to-kubernetes-cluster-on-raspberry-pi#1-overview), more on that later).

**What do you need to get started?**

All the parts separately:

[Raspberry Pi 4B](https://thepihut.com/products/raspberry-pi-4-model-b) (2GB)- £33.90

[Micro SD Minimum 16GB](https://www.amazon.co.uk/dp/B07V4DZBFG/ref=twister_B07HM3RLBS?_encoding=UTF8&th=1) (faster the better) - £10.00

[Micro-HDMI Cable](https://thepihut.com/products/micro-hdmi-to-standard-hdmi-a-cable) - £5.00

[USB Type-C Power Cable & Supply](https://thepihut.com/products/raspberry-pi-psu-uk?src=raspberrypi)  - £7.50

[Case (Not necessary but looks cool)](https://thepihut.com/products/aluminium-armour-heatsink-case-for-raspberry-pi-4?variant=31139034038334) - £12.00

[Or an official starter kit (2GB version)](https://thepihut.com/products/raspberry-pi-starter-kit) - £58

Once you have all the necessary parts, make sure you flash the correct operating system to the MicroSD card. Plug it in and begin! It really is that simple. The Raspbian OS is akin to any normal desktop GUI so should be relatively familiar for most users. It even contains a free Pi version of Minecraft!

Well what are you waiting for? Get started! [Here](https://projects.raspberrypi.org/en/projects) is an exhaustive list of the all the official Raspberry Pi projects.

**What do I plan to do with the Raspberry Pi?**

I have a number of projects in mind for the Raspberry Pi so keep your eyes peeled:

- Learning about Kubernetes by creating a cluster of Raspberry Pi's
- Managing Docker containers using Docker Swarm
- Turn the Pi into a portable Ethical Hacking machine using Kali Linux
- Building a LAMP Server with Wordpress
- Add a camera module and create a Machine Learning object tracker using Tensor Flow
- Block all online ads using Pi-Hole