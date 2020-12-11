# CAN bus on Raspberry Pi tips & tricks


## Install

### MCP2515 based solution

I describe my dual CAN setup with MCP2515 on two separated SPI interfaces.

Please use supplied in `overlays` folder `mcp2515-can1-overlay.dts`:

```
dtc -I dts -O dtb -o mcp2515-can1.dtbo mcp2515-can1-overlay.dts
dtc  -@ -Hepapr -I dts -O dtb -o mcp2515-can1.dtbo mcp2515-can1-overlay.dts

sudo cp  /boot/overlays/mcp2515-can1.dtbo /boot/overlays/mcp2515-can1.dtbo.original
sudo mv mcp2515-can1.dtbo  /boot/overlays
```

Now edit hardware config:
```
sudo nano /boot/config.txt

## SPI CAN bus
dtparam=spi=on
dtoverlay=mcp2515-can0,oscillator=16000000,interrupt=22
dtoverlay=mcp2515-can1,oscillator=16000000,interrupt=23
dtoverlay=spi-bcm2835-overlay
dtoverlay=spi1-1cs

```

And reboot RPi:
```
sudo reboot
```

Some sources mention of mandatory add parameter `spimaxfrequency` (after `interrupt` param) to limit SPI interface speed:
```
spimaxfrequency=1000000
```

I don't do this, my setup works fine without `spimaxfrequency` additions.

<!--
dtparam=spi=on
dtoverlay=mcp2515-can0,oscillator=16000000,interrupt=25,spimaxfrequency=1000000
dtoverlay=spi0-hw-cs
-->


After reboot try to bring up interface:
```
sudo /sbin/ip link set can0 up type can bitrate 500000
sudo ifconfig can0 txqueuelen 1000
```


Now check CAN interface status:
```
ip a
```

#### Auto bring up interfaces

Make two files (for two can HW instances):
```
cat /etc/network/interfaces.d/can0

auto can0
iface can0 can static
  bitrate 125000
```

```
cat /etc/network/interfaces.d/can1

auto can1
iface can1 can static
  bitrate 500000
```
Then reboot.

More detail:
* [Bring up CAN interface](https://wiki.rdu.im/_pages/Application-Notes/Software/can-bus-in-linux.html)
* [Good detailed explanation of /etc/network/interfaces syntax](https://unix.stackexchange.com/questions/128439/good-detailed-explanation-of-etc-network-interfaces-syntax)






### CAN Utils

Don't do this:
```
sudo apt-get install can-utils -y
```

I waste too much time with obsolete version can-utils.
It seems `candump` & `cansniffer` doesn't work with 29bit CAN bus IDs.
Can't apply with my DUT :(

```
sudo apt-get remove can-utils -y
  Removing can-utils (2018.02.0-1) ...
```


Install lastest version from git:
```
git clone --depth=1 https://github.com/linux-can/can-utils
cd can-utils
make && sudo make install
cd ..
rm -rf can-utils
```

List of available utils:

#### Basic tools to display, record, generate and replay CAN traffic

* **candump** : display, filter and log CAN data to files
* **canplayer** : replay CAN logfiles
* **cansend** : send a single frame
* **cangen** : generate (random) CAN traffic
* **cansequence** : send and check sequence of CAN frames with incrementing payload
* **cansniffer** : display CAN data content differences

#### CAN access via IP sockets

* **canlogserver** : log CAN frames from a remote/local host
* **bcmserver** : interactive BCM configuration (remote/local)
* [**socketcand**](https://github.com/linux-can/socketcand) : use RAW/BCM/ISO-TP sockets via TCP/IP sockets
* [c**annelloni**](https://github.com/mguentner/cannelloni) : UDP/SCTP based SocketCAN tunnel

#### CAN in-kernel gateway configuration

* **cangw** : CAN gateway userspace tool for netlink configuration

#### CAN bus measurement and testing

* **canbusload** : calculate and display the CAN busload
* **can-calc-bit-timing** : userspace version of in-kernel bitrate calculation
* **canfdtest** : Full-duplex test program (DUT and host part)

#### ISO-TP tools

* **isotpsend** : send a single ISO-TP PDU
* **isotprecv** : receive ISO-TP PDU(s)
* **isotpsniffer** : 'wiretap' ISO-TP PDU(s)
* **isotpdump** : 'wiretap' and interpret CAN messages (CAN_RAW)
* **isotpserver** : IP server for simple TCP/IP <-> ISO 15765-2 bridging (ASCII HEX)
* **isotpperf** : ISO15765-2 protocol performance visualisation
* **isotptun** : create a bi-directional IP tunnel on CAN via ISO-TP

#### J1939/ISOBus tools

* **j1939acd** : address claim daemon
* **j1939cat** : take a file and send and receive it over CAN
* **j1939spy** : spy on J1939 messages using SOC_J1939
* **j1939sr** : send/recv from stdin or to stdout
* **testj1939** : send/receive test packet

#### Log file converters

* **asc2log** : convert ASC logfile to compact CAN frame logfile
* **log2asc** : convert compact CAN frame logfile to ASC logfile
* **log2long** : convert compact CAN frame representation into user readable







## Usage


### Common CAN things

The CAN bus statistics:
```
cat /proc/net/can/stats
```

Summary stats for all interfaces:
```
cat /proc/net/dev
```

Detailed interface status:
```
ip -details link show can0
```


Reboot can0 interface:
```
sudo ip link set can0 down && sudo ip link set can0 up
```


### How to handle Virtual CAN

Just for playground stuff or complex scenarios.




### Hardware CAN




## Links

* todo













