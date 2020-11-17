# RaspberryPi Gotchas

URL: https://github.com/iDoka/raspberrypi-gotchas

## ⚠️ Note

Here my collection of **practical** RPi's recipes.
I checked or created this recipes by my hands.

## TOC

* [RTC setup](RTC.md)
* [CAN bus](CAN.md)
* [SDR (Software Defined Radio)](SDR.md)
* [Speed up booting](fastboot.md)
* [Display related stuff](display.md)
* [Video related stuff](video.md)
* [Audio related stuff](audio.md)


## Before install

Burn image to uSD card:
```
sudo dd if=2020-08-20-raspios-buster-armhf-lite.img bs=4M of=/dev/sdX conv=fsync
```
WARNING: instread of `/dev/sdX` use your uSD card device!!! (please double check with this cmd!!!)


## First steps after Raspbian install

Update all:
```
sudo apt-get update -y
sudo apt-get upgrade -y
sudo reboot
```

Install neccessary packet:
```
sudo apt install git python2.7 python-pip python3 python3-pip cython cython3 ipython ipython3 python-dev python3-dev 
sudo apt install python-serial python3-serial python3-serial-asyncio python-libusb1 python3-libusb1
sudo apt install python-numpy python3-numpy python-yaml python3-yaml python-setuptools python3-setuptools
sudo apt install virtualenv python-virtualenv python3-virtualenv pex python-pex python3-pex dh-virtualenv virtualenvwrapper python3-venv

sudo apt install libusb-1.0-0 libusb-1.0-0-dev libusb-dev libusbhid-common libhidapi-libusb0

sudo apt install libv4l-dev v4l-utils
```





