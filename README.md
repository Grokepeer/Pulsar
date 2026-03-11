<div align="center">

# Pulsar

<img src="./Images/pulsar-logo.png" width="300px">

</div>

> [!WARNING]
> Attention! This project is currently under rapid development so some of the content below might be not yet available or incorrect. If you are interested in the project and/or the development of the following, add the repo to your watch list and star it!

<div align="justify">

This document contains an overview of the project, a simple description of its parts and its development process, basic instructions on how to build your own Pulsar and how to use it, along with some renders and pictures.<br>
I hope you will enjoy the process and you are always welcome to open an issue for feedbacks and questions.

- Want to know more about the project? Take a look at the [synopsis](#synopsis).
- Curious about the development? Read more [here](#technical-overview)
- Ready to build your own Pulsar? Jump [here](#production)
- Already holding onto your very own Pulsar, but unsure of how to use it? [This way](#operation)

## Synopsis

The Pulsar is a highly portable, off-grid (and optionally on-grid), radio communication system. The goal is to create a small (hand-held), battery powered, system that can utilize multiple frequencies and high-power RF power amplifiers to allow long-range, decentralized, reliable and secure, communication between two or more nodes.<br>
Through the use of an optional 4G/5G module, the system can also communicate via traditional networks.

The main computer can connect to other devices (like smartphones, computers and even CSRF controllers) via bluetooth LE and serial wired ports. The conneted devices can send, via the serial line with the main computer, data that they need to send over the radio link. The main computer then uses multiple RF modules and, if installed, the 4G/5G module, to communicate with another Pulsar (or compatible device), and send the data (encrypted), once it's on the other Pulsar it can be sent via serial connection to the specific destination device.

There's the option to add a small SBC running OpenWRT and a 4G/5G cellular module. This way the main computer will have access to the 4G/5G lines and you will have a fully featured router and 4G/5G modem that you can directly connect to.

The system is open-source and open-hardware so that anybody that is interested can build one on its own and even modify the design or give his/her feedback.

## Table of contents

- [Repository Structure](#repository-structure)
- [Technical Overview](#technical-overview)
    - [Radio Modules](#radio-modules)
    - [Central Processor Unit](#central-logic-unit)
    - [Power Supply Unit](#power-supply-unit)
    - [Case](#case)
- [Production](#production)
    - [PCB Sourcing](#pcb-sourcing)
    - [PCB Assembly](#pcb-assembly)
    - [3D Printing](#3d-printing)
    - [Assembly and Firmware](#assembly-and-firmware)
- [Operation](#operation)
    - [To Arrive Soon]()

## Repository Structure

The repository is divided into 4 sections.
- The base folder along with the [images](./Images/) folder contains the README, LICENSE and some useful informations to get started. All of the informations contained in these folders are accessible in an organized matter in this README.
- The [3D Files](./3D%20Files/) folder contains 3D files (mainly STEP) that you will need to print to build your Pulsar (read more about this [here](#3d-printing)). It also contains some 3D models of the PCBs so that you can use them if you wish to integrate the PCBs into your own design.
- The [PCB Files](./PCB%20Files/) folder contains gerber files to send to your preferred PCB manufacturer, BOM files so that you can order all necessary components and the PCB's schematics.
- The [Firmware](./Firmware/) folder contains the STM32CubeMX project with all the STM32H733VG code and configuration files. A pre-compiled binary file will be provided on release. It's not recommended to build the code yourself, nor to modify it, if you don't know what you are doing.

## Technical Overview

Over this chapter I'm going to talk about each of the parts that make up the system, from the electronics to the 3D printed case.

To give a general overview of the project, I'll give a quick list of its parts. The Pulsar is powered by a battery (3S LiPo/LiIon), uses a DC/DC PSU to power the main logic board (CLU), a multitude of RF modules, an optional 4G/5G module, the router SBC and a GPS module. The SBC and 4G/5G module can be turned on/off from the uC via relays. There's also a rotary selector and a small display that the user can interface with. A 3D printed case encloses everything.

### Radio Modules

The radio modules are SemTech SX1262 and SX1281 based modules that use LoRa modulation over a range of frequency as low as 100 MHz and as high as 2.4 GHz. LoRa is a popular, efficient and well adopted modulation system, so choosing it, rather then another, allows me to have access to a great choice of transceiver chips and plenty of documentation.

The modules are separated from the main board and power supply so they can be swapped easily and freely depending on preference. This allows me to plan the development of more powerful radio modules down the line.

The radio modules I'm going to use at first are Ebyte LoRa modules. These modules integrate an RF PA with an SX1262 or SX1281 chip and allow direct communication to the SemTech chip. So in case I swap the module for a different one, from another brand or a custom build one, I can always use the same interface, as long as I keep using the SemTech chip. This also means that I can use the SemTech provided programming documentation, indepedently from the module manufacturer, should it change eventually.

### Central Logic Unit

The main board, or CLU, connects together all modules and user interfaces. It does not provide power, that is left to the [PSU](#power-supply-unit).

The board has 4 MODx connectors that offer multiple GPIOs and an SPI interface, to connect to the RF modules (in particular, to connect directly to the SemTech SX1262 and SX1281 chips), 3 UARTs that can be used to connect to external computers and controllers, a single I2C that is used to connect to the internal screen and a GPIO that connects to the rotary selector and the SBC's relays.<br>
On the baord you will also find an SD card slot that allows the system to save log files during operation, a coin battery that allows the internal RTC to remain active without the main battery, a small DIP switch that is used for configuration of the board, the JDY-23 bluetooth module, and the STM32H733VGT6 at the heart of the system.

The CLU inside the Pulsar is a 4 layer, 60 x 75 mm PCB.

<div align="center">
<p float="left">
    <img src="./Images/clu-toppcb.png" width="49%">
    <img src="./Images/clu-bottompcb.png" width="49%">
    <img src="./Images/clu-topiso.png" width="49%">
    <img src="./Images/clu-bottomiso.png" width="49%">
</p>
</div>

### Power Supply Unit

To power the entire system with high-power RF modules, microcontrollers, an SBC and a 4G/5G modem, a proper DC/DC power supply is necessary. The RF modules use both 3.3 V and 5.0 V, the microcontroller uses 3.3V, the SBC and the modem use 5.0 V, so it was necessary to design a 2 rail DC/DC PSU. To allow the addition of a more powerful RF PA further down the line or to have a dedicated line that powers the SBC, I also added the possibility to enable a 3rd rail at a voltage that can be defined easily after production, I will refer to this rail as the 0V0 DC/DC rail.

Since this system is also battery powered, it needs to be efficient, so the entire DC/DC needs to be extrememly efficient, and I wanted it to be able to switch to an external power source whenever necessary, without shutting down.

The PSU inside the Pulsar is a 4 layer, 60 x 60 mm PCB with 3, up to 3 A each, DC to DC rails and an ideal diode controller that can hot-swap between two power sources at voltages ranging from 8 V to 16 V. Each rail's switching controller can be chosen between a 3 A variant and a 2 A one, depending on projected consumption. On my design I decided to go with two 3 A, 5.0 V rails and a 2 A, 3.3 V. Based on simulations the 3.3 V rail can reach 94.8% efficiency at 50% load.

<div align="center">
<p float="left">
    <img src="./Images/psu-toppcb.png" width="49%">
    <img src="./Images/psu-bottompcb.png" width="49%">
    <img src="./Images/psu-topiso.png" width="49%">
    <img src="./Images/psu-bottomiso.png" width="49%">
</p>
</div>

### Case

Case infos...

## Production

Over the course of this chapter I will go into some detail about how to procure yourself everything you need to create your own Pulsar, how to assemble it and program it.

### PCB Sourcing

In the [PCB Files](./PCB%20Files/) folder you will find the gerbers .zip files of all the PCBs that are necessary to build a functioning Pulsar. These files contain the PCB production informations such as copper pours polygons and drill holes. You will need to download all the gerber files and send them to your preferred PCB manufacturer (such as JLCPCB).

Particular PCB requirements and generic informations are provided here:
- The PSU board is a 4 layer, 60 x 60 mm board that doesn't need matched impedance nor a particularly high copper weight (1 oz. outer, 0.5 oz. inner will be fine). The board original thickness is 1.6 mm but you can make it thinner or thicker (although it might affect the ability to fit into the case).

## Operation

In the midst of this chapter you will find some suggestions regarding the ways of the firmware.

</div>