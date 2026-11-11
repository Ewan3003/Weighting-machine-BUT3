 ## Weighting machine

 I built a weighting machine from scratch : 

<img width="1984" height="1186" alt="BA4241ED-91E1-4A0F-97D7-63CC95D81753" src="https://github.com/user-attachments/assets/2f08357f-5fc5-42c0-919b-7b6ab4bd4488" />

The device consists of an aluminium rod with a very specific geometry. Strain gauges are completely sticked on the aluminium rod, these sensors are changing resistors according to the constraint applyed on the edge of the rod. This is why one part of the rod is screwed on the solid basis of the complete device. I had to come up with a mechanical protocol first in ordre to understand how the aluminium rod would react to the weight constraints : 

<img width="400" height="334" alt="742B14AD-32A2-4768-88B0-B374C27C8F5A" src="https://github.com/user-attachments/assets/49c447cf-61d7-4866-ad80-c0f03e68d922" />

A metrology work has been done to determine the uncertainty of the measurements in different metrolgy cases scenarios. I even created a traceable calibration certificate as if the product following a strict metrology protocol. 

I am using a 24 bit CAN (HX711 Module) wired to an ESP-32 through a Wheatstone bridge to acquire the signals.

Data is processed by the microcontroller and sent to a NodeRed server with MQTT protocol.
At the same time, the program is calculating the uncertainty on the current measurement which is also sent to the NodeRed server on a different topic.

Check the main code for further information. 

 


