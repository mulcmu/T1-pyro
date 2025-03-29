To build these cables you need crimpers for Dupont and JST XH terminals, terminals to crimp and the housings.

https://www.amazon.com/YMYP-JST-XH-0-25-1-5mm%C2%B2-JST-SM2-54-Connector/dp/B0C6DKZC4K/ 

https://www.amazon.com/WOWOONE-620pcs-Connector-Housing-Terminal/dp/B0897KGG59/

### Cable from Pi USB to stock USB hub board

The camera and external usb stock pcb uses a usb hub IC.  There is also a transistor circut to power the case light on and off.  The 5 pin 2.54 JST XH connector needs to be wired to one of the Pi's usb ports and the control line for the light connected to the PI GPIO header.

Usb breakout pcb

https://www.amazon.com/MakerHawk-Adapter-Converter-Interface-Breakout/dp/B07MBSTZYG

https://www.amazon.com/Teansic-Breakout-Adapter-2-54mm-Module/dp/B09ZNK2Q5L

Connections

![./images/usb cord.jpg](https://raw.githubusercontent.com/mulcmu/T1-pyro/refs/heads/main/images/usb%20cord.jpg)

The brown wire goes off to the GPIO Dupont connector.

Use some heat shrink tubing to clean up/protect the usb pcb

Green and white data lines twisted for good measure

### Power Cable

The lower core board has a 5v supply that powered the original motherboard.  There is not enough room to power the Pi via the usb connector when installed so it needs to be powered by the GPIO header.

One side has another 5 pin 2.54 JST XH connector.  The other side needs dupont terminals.  A double row dupont connector 2x6 is recommended for convivence and to keep from messing up the connections.

Two red wires are the 5v and two black wires are the common.

Connections

![./images/power cord.jpg](https://raw.githubusercontent.com/mulcmu/T1-pyro/refs/heads/main/images/power%20cord.JPG)

Optional for [load cell] support:

Two wires need run from the PI GPIO RX/TX to the modified lower function adapter board.  These wires terminate in a separate 5 pin Dupont housing.
