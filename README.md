# 3-Node-Packetized-Radio-Network-CSMA-CA
This model is intended to simulate a 3- node PHY / MAC network. The goal is to find the mechanisms related to CSMA / CA protocol within our source code.
For the purpose of this simulation, we are only concerned with primarily the MAC layer.
For CSMA/CA MAC (Figures 13 and 14), the medium must be quiet for the period of DIFS before transmitting, then the node starts the random Contention Window (CW) period. At the end of the CW, if the medium is still quiet, the node starts to transmit a Data frame; otherwise it waits for another quiet DIFS period. When the transmitting node hears an Ack frame within the SIFS period, it will transmit the next Data frame; otherwise it enters the DIFS period again.
![image](https://user-images.githubusercontent.com/77028776/127751222-77b0f14a-57a3-4787-a705-ffcc96cc9540.png)

## Acknowledgement Frames



## Backoff / Contention Window
