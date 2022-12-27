# Repair of a System76 Launch v1.3

The great thing about an open source hardware development is the easy
access to schematics and PCB layout.
The entire keyboard project is available here: https://github.com/system76/launch

The keyboard has the following initial state of failure.
It is no longer visible in the in the **lsusb** print out, but the integrated
USB hub is still available.
The following **lsusb -t** entry is available:


    /:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/12p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/2p, 480M
            |__ Port 2: Dev 9, If 0, Class=Hub, Driver=hub/5p, 480M

The **dmesg** posts the following after the keyboard is attached to the
system:


    [224015.633544] usb 3-1.2: new high-speed USB device number 12 using xhci_hcd
    [224015.780839] usb 3-1.2: New USB device found, idVendor=3384, idProduct=4216, bcdDevice= 6.22
    [224015.780847] usb 3-1.2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    [224015.780850] usb 3-1.2: Product: USB4216 Smart Hub
    [224015.780853] usb 3-1.2: Manufacturer: Microchip
    [224015.782226] hub 3-1.2:1.0: USB hub found
    [224015.782292] hub 3-1.2:1.0: 5 ports detected


In case a USB device is attached to the left side USB-A port of the integrated
hub it is detected.


    /:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/12p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/2p, 480M
            |__ Port 2: Dev 9, If 0, Class=Hub, Driver=hub/5p, 480M
                |__ Port 3: Dev 10, If 1, Class=Human Interface Device, Driver=usbhid, 12M
                |__ Port 3: Dev 10, If 2, Class=Human Interface Device, Driver=usbhid, 12M
                |__ Port 3: Dev 10, If 0, Class=Human Interface Device, Driver=usbhid, 12M


In case a USB device is attached to the right side USB-A port of the integrated
hub it is **not** detected.

# Repair Log

After no visible defects like liquid damage or smoldered components are found
the measurement of all voltage levels is performed.

## Measurements

| Net      | Location | Expected | Actual  |
|----------|----------|----------|---------|
| VBUS     | U9.5     | 5.0 V    | 5.2 V   |
| VCC      | U16.1    | 5.0 V    | 5.2 V   |
| 3.3V     | U18.5    | 3.3 V    | 3.3 V   |
| 1.1V     | C19.1    | 1.1 V    | 1.146 V |
| CR_VBUS  | U9.1     | 5.0 V    | 0 V     |
| AR_VBUS  | U23.1    | 5.0 V    | 0 V     |
| CL_VBUS  | U20.1    | 5.0 V    | 0 V     |
| AL_VBUS  | U10.1    | 5.0 V    | 5.2 V   |
| AVR_VBUS | U22.1    | 5.0 V    | 0 V     |
| PRT_CTL1 | U9.3     | 5.0 V    | 0 V     |
| PRT_CTL2 | U23.3    | 5.0 V    | 2.6 V   |
| PRT_CTL3 | U10.4    | 5.0 V    | 5.2 V   |
| PRT_CTL4 | U20.4    | 5.0 V    | 0 V     |
| PRT_CTL6 | U22.4    | 5.0 V    | 0 V     |

## Assumption

U9, U10, U20, U22 and U23 are AP22811 over current sensors and all but U10
report an over current to the hub and disable the corresponding VBUS signal.
An over current is reported even if no USB device is attached to the hub port.
Therefore, the assumption is that U9, U20, U22 and U23 are all broken.

## Confirmation

The first goal of this repair is to get the keyboard running again.
Therefore, the focus is on U22 and to confirm the assumption, that it is broken
the part is soldered out of the PCB.
Instead of generating the 5V AVR_VBUS signal from the VBUS via U22 the voltage
is provided by an external source.
The following image shows the board with a removed U22 and the attached
external source to AVR_VBUS (U22 pin 1) and the GND.

![External AVR_VBUS](/images/removed_u22_external_AVR_VBUS.jpeg?raw=true)

With the external supply of AVR_VBUS the current does not exceed 15 mA which is
way below the AP22811 threshold (min 2.2 A).
In addition to that the keyboard device is available again:


    /:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/12p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/2p, 480M
            |__ Port 2: Dev 84, If 0, Class=Hub, Driver=hub/5p, 480M
            |__ Port 5: Dev 86, If 2, Class=Human Interface Device, Driver=usbhid, 12M
            |__ Port 5: Dev 86, If 0, Class=Human Interface Device, Driver=usbhid, 12M
            |__ Port 5: Dev 86, If 1, Class=Human Interface Device, Driver=usbhid, 12M


At this point only the **Fn** and the **0** keys are mounted on the device (to
disable the blinding background LEDs).
Pressing the zero key on the keyboard produces a **0** on the screen.
The assumption about the broken parts seams to hold.

## Quick and Dirty Repair

Until the replacements parts are available a fast fix is performed.
The external voltage supply is removed and the working U10 is soldered to the
position of U22.
Afterwards the system behaves the same way as described above and at least the
keyboard is working again.
But all USB hub ports are now unavailable.

## Complete Repair


