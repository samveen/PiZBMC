## PiZBMC - A BMC using a Raspberry Pi Zero

This project is a vanity project to create a Rasberry Pi zero Based BMC to allow
for IPMI control of a Raspberry Pi 3B/4B (and maybe 3B+ if I get one to test it
on).

### Name
This name comes as a joke from my Russian friends, particularly
[Aleksei](https://www.linkedin.com/in/weaselshit/)
who taught me the exclamation [пиздят](https://en.wiktionary.org/wiki/%D0%BF%D0%B8%D0%B7%D0%B4%D1%8F%D1%82#Russian)
by using it very often (we sat next to each other), and [Misha](https://www.linkedin.com/in/michael-osipov-0a131536/)
who taught me how versatile it could be.

### Motivation

I've been working with xCAT a LOT over the last 4 years (mid 2015 onwards), and
I have come not only to like and understand it, but I've managed to become a
contributor to it.

As part of my testing with xCAT I wanted to try and see If I could manage to set
up a cluster of
[ODroid N2](https://www.hardkernel.com/shop/odroid-n2-with-4gbyte-ram/) nodes
with xCAT. As I was deciding whether I should use one of the ODroids N2s or a
Raspberry Pi 3B as the test server, I took a step back and though about how I
could manage remote power and console for the nodes. I then realised that
there wasn't any IPMI equivalent control for these nodes.

There began my effort to figure out how I would manage IPMI control for these
devices. There are already a lot of possible partial solutions for this:
- POE shields
- Network connected power switches
- USB TTL Serial console cables
- Existing Raspi BMC implementations (not IPMI, but custom interfaces)
- OpenBMC
- PYGHMI
- ConPot, IPMISIM

### Hardware Requirements

- A Raspberry Pi [3B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/) or a [4B](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/). I got my 3B as a gift from a friend, and the 4B via redirect to a [official distributor](https://www.thingbits.in/products/raspberry-pi-4-model-b-2-gb-ram?src=raspberrypi) from the Pi website. The Price was reasonable.
- A [Raspberry Pi Zero](https://www.raspberrypi.org/products/raspberry-pi-zero/). I got mine over-priced @ [USD 15](https://robu.in/product/raspberry-pi-zero-v1-3-development-board/), don't do that it's price is 5 USD.
- [A 12 Pin ENC28J60 SPI Module](https://www.google.com/search?&q=ENC28J60+12+pin) I got mine for [USD 5](https://roboticsdna.in/product/enc28j60-ethernet-module12-pin-package-ssop/) which was again expensive.
- [Jumper Cables(Thank you, Dupont)](https://www.google.com/search?&q=jumper+connectors), Female to Female
- 2.54 mm standard pitch male headers (correct terminology via [Digikey](https://www.digikey.com/catalog/en/search?filters=143632)):
  - one of 2 x 20, or
  - 2 of 1x20 (they'll come as a strip of 1x40) (Thank you for these, [Stuart](https://github.com/sturem/))
- Soldering Iron, Solder and YouTube.
- Small network switch and network patch cables. I bought a [Mercusys MS105G](https://www.mercusys.co.in/product/details/ms105g), which worked fine for my test setup. All but 1 of my network cables were pulled from the E-Waste bin that my mom maintains, broken latching tabs and all. :facepalm:

### Log of the process

#### Hardware setup process
- Collect all the Hardware requirements
- Solder the headers to the raspberry pi. I found that using a bread board or an
  ATA connector to hold the pins and tying the Zero against the pin holder using
  the end holes (which coincidentally line up perfectly with the header holes)
  allows for a well positioned set of pins. (TODO: add PIC)
- Connect the ENC28J60 module to the zero as follows:
  - Connect 7 lines of jumper cables between the ENC28J60 pins and the Pi Zero as
    follows (Legend: Label/Application (Pin No.)):

    | ENC28J60 | Pi Zero  |
    |:--------:|:--------:|
    | 5v   (1) | 5v Power (2) |
    | GND  (2) | Ground (20)  |
    | INT  (3) | GPIO 25 (22)|
    | CLK  (4) | NC |
    | SO   (5) | GPIO 9/SPI0 MISO (21)|
    | WOL  (6) | NC |
    | SCK  (7) | GPIO 11/SPI0 SCLK (23)|
    | SI   (8) | GPIO 10/SPI0 MOSI (19)|
    | RST  (9) | NC |
    | CS  (10) | GPIO 8/SPI0 CE0 (24) |
    | 3.3 (11) | NC |
    | GND (12) | NC |

   - An alternative power arrangement is to use 3.3v instead of 5v to power the
     ENC28J60(or if the ENC28J60 has no 5v to 3v3 regulator like the AMS1117):

     | ENC28J60 | Pi Zero  |
     |:--------:|:--------:|
     | 5v   (1) | NC |
     | GND  (2) | NC |
     | 3.3 (11) | 3v3 Power (17) |
     | GND (12) | Ground (20) |

- Connect the the UARTs of the 2 Raspberry Pis together (3 pins: GND and RX-TX swapped, in case you didn't figure that out).

#### Pre-boot Software setup

- Install [Ubuntu 20.04](https://ubuntu.com/download/raspberry-pi) on a Micro SD card for the Raspberry Pi 3B/4B
  - Mount the boot partition and ensure that the Serial console is enabled
  - Do whatever modification (netplan/cloudconfig) changes you need before firstboot (This is optional).
- Install [Raspberry Pi OS Lite](https://www.raspberrypi.org/downloads/raspberry-pi-os/) on a Micro SD card for the Pi Zero
  - Mount the boot partition and add configuration to enable SPI and add the ENC28J60 overlay to the config.txt (TODO: Give snippet)
  - Mount the root FS partition and enable ssh service via systemd linking (TODO: Give Details)

- Put in the respective SD cards and boot up everything. Monitor the DHCP server to ensure that the both the Pis get IP addresses.

#### Post boot setup
##### Pi Zero
 - Set the Mac address of the ENC28J60 to a fixed address as described [here](https://github.com/raspberrypi/linux/issues/795). (Caveat emptor: use `which ip` to find full path of the `ip` command which differs across distributions :facepalm:


### Development steps
- Install `python3` and the `pyghmi` python library (use your favourite source (I cloned the official repo)
- Run the fakebmc.py script that is packaged with the distribution (TODO: Pic)
- Actually develop something out of the FakeBMC (TODO: Do something)

### Advantages
The Raspberry Pi platform has been around long enough that most problems that I thought of and faced
had already been faced by someone earlier and solved. I didn't have to do any hard solutions for
anything, just the process of fitting together and then stitching all the pieces already in place.

### Alternatives not examined:

- [tiny core Linux](http://www.tinycorelinux.net/11.x/armv6/releases/RPi/)
- Wiznet5500 based ethernet modules

## License

Everything here is released under the HIRE ME/PAY ME License (a modified 2 Clause BSD License). Please see the LICENSE file for details.

Exceptions:
- `fakebmc.py` is straight out of the pyghmi distribution, so their license applies.

## References:

* [Google Fu(A BIG THANKS to the Google folks and Siva(HOD CSE IITB 2004))](https://www.google.com/)
* [A Raspberry Pi BMC](https://github.com/joshuaboniface/rpibmc)
* [Interactive Raspberry Pi Pinout](https://pinout.xyz/)
* [Attaching Ethernet on Pi Zero](http://raspi.tv/2015/ethernet-on-pi-zero-how-to-put-an-ethernet-port-on-your-pi)
* [Raspberry Pi Zero Headless Quick Start](https://learn.adafruit.com/raspberry-pi-zero-creation/overview)
* [Arduino Ethernet Module](https://www.google.com/search?q=arduino+ethernet+module) -> [ENC28J60](https://www.google.com/search?q=ENC28J60)

* [IPMI 2.0 Technical Resources](https://www.intel.com/content/www/us/en/servers/ipmi/ipmi-technical-resources.html)

* [Using screen to Connect to the Serial Console](https://docs.fedoraproject.org/en-US/Fedora/22/html/System_Administrators_Guide/sec-Using_screen_to_Connect_to_the_Serial_Console.html)
* [RPi Serial Connection](https://elinux.org/RPi_Serial_Connection)
* [Priviliged ports opening without user `root`](https://superuser.com/a/892391/246855)
