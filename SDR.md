# SDR Raspberry Pi tips & tricks


## Common

build tools:
```
sudo apt-get install libtool libusb-1.0-0-dev build-essential autoconf cmake pkg-config -y
```

SoapySDR:
```
sudo apt-get install soapysdr-tools libsoapysdr-dev libsoapysdr0.6 python-soapysdr python3-soapysdr soapysdr-module-all -y
```

## RTLSDR

```
sudo apt-get install rtl-sdr librtlsdr-dev soapysdr-module-rtlsdr soapysdr0.6-module-rtlsdr -y
```

also useful to install udev rules for access to your SDR HW under unpriveleged user:
```
sudo cp 28-rtlsdr.rules /etc/udev/rules.d/28-rtlsdr.rules
sudo udevadm control --reload-rules
```

### usage

Write RF @ 433.92M & 250kSamples/s:

```
rtl_sdr  -f 433920000 -s 250000 dump.cu8
```

### rtl_433

```
git clone --depth=1 https://github.com/merbanan/rtl_433
cd rtl_433
rm -rf build
mkdir -p build
cd build
cmake  ..
make -j4 && sudo make install
```



## HackRF

```
sudo apt-get install hackrf libhackrf-dev libhackrf0 soapysdr-module-hackrf soapysdr0.6-module-hackrf -y
```

### usage


Write RF @ 433.92M & 250kSamples/s:
```
hackrf_transfer -f 433920000 -l 16 -s 250000 -r dump.cs8
```

Playback RF @ 433.92M & 250kSamples/s:
```
hackrf_transfer -f 433920000 -x 10 -s 250000 -R -t dump.cs8
```




