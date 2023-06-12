# MPFS OPC-UA Industrial Edge Demo

This demo guide describes how to run the OPC-UA Industrial Edge Demo and demonstrates the basic functionality of the PolarFire SoC Video Kit, the PolarFire SoC Icicle Kit, the Stepper Motor, and their communication with each other over the OPC-UA protocol.

This solution is developed on PolarFire® SoC video kit, PolarFire® SoC Icicle kit, Multi Stepper Click Board, and Multi Stepper Motor. OPC-UA Client UAExpert acts as the main controller, which controls the PolarFire SoC Video Kit and the Stepper Motor on the Icicle kit. OPC-UA control signals are sent from UAExpert to the PolarFire SoC Video Kit and to Icicle Kit.

Open Platform Communications Unified Architecture (OPC-UA) is a machine-to-machine communication protocol that has become increasingly popular in industrial automation. The OPC-UA protocol is based on a client-server architecture, where the client initiates the communication and the server provides the requested data. The communication is structured around a set of predefined services that define the interaction between clients and servers.

## Running the Demo

In this demo, OPC-UA Client UAExpert acts as the main controller, which controls the PolarFire SoC video kit and the Stepper Motor on the Icicle kit. Control signals from UAExpert are sent over the OPC/UA channel to the PolarFire SoC Video Kit to start/stop video streaming and Icicle Kit to spin the stepper motor in clockwise or anti-clockwise direction.

For a step-by-step guide on how to run the demo, please click the App Note link [AN4977](https://www.microchip.com/en-us/application-notes/an4977). It contains detailed instructions to assist you throughout the process.
