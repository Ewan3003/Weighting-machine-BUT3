 Weighting-machine-BUT3-TI-B

 We built a weighting machine in a group project : Rémi & Ewan 

The device consists of an aluminium rod with a very specific geometry. Strain gauges are completely sticked on the aluminium rod, these sensors are changing resistors according to the constraint applyed on the edge of the rod. This is why one part of the rod is screwed on the solid basis of the complete device. A metrology work has been done to determine the uncertainty of the measurements in different metrolgy cases scenarios.

We are using a HX711 Module connected to an ESP-32 to collect and visualise the data. Data is processed and sent to a NodeRed server with MQTT protocol.  
At the same time, the program is calculating the incertainty on the current measurement which is also sent to the NodeRed server on a different topic.

Check the main code for further information. 

 


