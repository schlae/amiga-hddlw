# Herr Doktor Diskettenlaufwerk (Amiga-HDDLW)

Late-model Amiga computers use customized floppy drives (like the Chinon
FZ-357A) to be able to read and write high-density floppy disks
(colloquially known as 1.44MB floppies, although the Amiga can store 1.76MB
because it packs in four additional sectors per track.)

Why a customized floppy drive? The Amiga custom chip, Paula, only supports
the slower bit rate used by double-density floppy disks. Instead of spending
a bunch of money to change the Paula chip, Commodore engineers got a
customized floppy drive that spun the disk at half speed when it detected
a high-density floppy disk.

In the 1990s, some 3rd party vendors released Amiga compatible floppy drives.
One such design is the AMTRADE "The Real HD-Drive" A357. It uses a modified
Sony MPF920-E or a TEAC FD-235HF (both fairly common) along with a small
adapter circuit board containing a 16V8 PAL.

I've done a bit of reverse engineering to understand how the A357 works,
and I've been able to build a functional replica. It's not an exact copy
because the original PAL is locked. I'm calling it the Herr Doktor
Diskettenlaufwerk, or HDDLW.

*Please note that I have not fabbed out this design yet, and although the
design is tested and expected to work, go forward at your own risk.*

## Fabrication

The board is a simple 2-layer affair with a small handful of parts. The
dimensions are 93.5mm x 20.3mm (3.68" x 0.8").

[Schematic](https://github.com/schlae/amiga-hddlw/blob/main/amiga-hddlw.pdf)

[Bill of Materials](https://github.com/schlae/amiga-hddlw/blob/main/amiga-hddlw.csv)

[Fab Package](https://github.com/schlae/amiga-hddlw/blob/main/fab/amiga-hddlw_rev1.zip)

Install a 20-pin DIP socket in position U1 just in case you have trouble
programming the PAL. All parts, with the exception of J2, J5, and J6, should
be installed on the top side of the board.

J4 should be a Molex 4-pin header with a key. In a pinch, you can sub it out
with a standard 4-pin header but it will be very easy to connect the cable
backwards, which will put 12V on the drive's 5V rail, destroying everything!

Install **either** J5 **or** J6 depending on the drive you plan to use. If you
install both, J5 will interfere with the drive select jumpers on the Sony
drive and potentially damage them.

Jumper J3 should be set to 1 for DF0 and 2 for DF1. Position 3 has not been
tested yet but may work.

## PAL programming

See the [source file](https://github.com/schlae/amiga-hddlw/blob/main/pal/amiga-hddlw.pld).

The logic equations are assembled using [galette](https://github.com/simon-frankau/galette)
and can be burned to the PAL using a number of different tools. I use the
TL-866 with the [minipro](https://gitlab.com/DavidGriffith/minipro) tool under
Linux, along with the ATF16V8B. The speed grade is not critical.

## Drive modifications

The HDDLW has been tested with the two floppy drives below. It may be possible
to use it with other drives, but you will need to investigate to figure out
the clock connection that sets the motor RPM. You will also need to figure
out how to get the high density select signal onto pin 4 of the drive.

### Sony MPF920-E

Perform the following changes

* Install JC40 (a zero ohm resistor or wire jumper), which is connected to pin
4 of the floppy connector and provides the high-density sense signal. See the
green circle in the image below.
* Cut the trace between pin 38 of IC1 and pin 16 of IC2. This is the motor
driver clock. This is indicated by the diagonal arrow in the image below.
* Connect a wire from pin 38 of IC1 to pin 14 of the floppy connector (blue
wire in the image)
* Connect a wire from pin 16 of IC2 to pin 6 of the floppy connector (red
wire in the image)

![Modifications on the Sony MPF920-E](https://github.com/schlae/amiga-hddlw/blob/main/photos/sony_modded.jpg)

### TEAC FD-235HF

Perform the following changes

* Install S5 (a zero ohm resistor or wire jumper), which is connected to pin 4
of the floppy connector and provides the high-density sense signal (circled in
the image)
* Remove 1K resistor R9 (which may be named differently on different board
revs, but one end is always tied to pin 4 of J5; pointed to by the green
arrow)
* Connect a wire from U1 pin 48 to pin 14 of the floppy connector
* Connect a wire from J5 pin 4 to pin 6 of the floppy connector

![Modifications on the TEAC FD-235HF](https://github.com/schlae/amiga-hddlw/blob/main/photos/teac_modded.jpg)

TEAC used several different board versions, so your board may not look exactly
alike. The example below uses R12 instead of R9, and S5 has been moved although
it retains the same designator.

![Alternate TEAC board](https://github.com/schlae/amiga-hddlw/blob/main/photos/teac_variation.jpg)


## Functional Explanation

The drive modifications intercept the clock signal that goes between the
floppy drive's main control IC and the motor controller, bringing them out
to two unused pins on the floppy connector. They also enable the undocumented
feature that provides a high-density detect output on pin 4.

The PAL checks for high density disks by looking at pin 4. If a high density
disk is inserted, the PAL

* Divides the motor clock by 2 to slow the RPM from 300 to 150rpm
* Returns the drive ID pattern `10101010...` to the Amiga to let it know that
this is a high-density drive/disk.

If a double-density disk is inserted, the PAL

* Passes the motor clock signal through unchanged so the disk spins at 300rpm
* Returns the drive ID pattern `11111111...`, a double-density drive/disk

And that's about it.

## License
This work is licensed under a Creative Commons Attribution-ShareAlike 4.0
International License. See [https://creativecommons.org/licenses/by-sa/4.0/](https://creativecommons.org/licenses/by-sa/4.0/).

