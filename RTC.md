# RTC Raspberry Pi tips & tricks

## DS3231

add to `/boot/config.txt`:
```
## I2C
dtparam=i2c_arm=on
dtparam=i2c_arm_baudrate=10000

## RTC
dtoverlay=i2c-rtc,ds3231
```


then run:
```
sudo apt-get install python-smbus python3-smbus i2c-tools ntpsec-ntpdate -y
sudo i2cdetect -y 1
```
Note: `ntpdate` packet is deprecated (Nov 2020).


You should see `0x68` address, now you can reboot:
```
sudo reboot
```

After reboot
```
sudo i2cdetect -y 1
```
should print UU instread 0x68. It mean driver loaded (device in use).


Now disable fake `hwclock`:
```
sudo apt-get -y remove fake-hwclock
sudo update-rc.d -f fake-hwclock remove
sudo systemctl disable fake-hwclock
```

If needed setup right timezone: 
```
sudo dpkg-reconfigure tzdata
```

Make DS3231 as main RTC. In file `/lib/udev/hwclock-set` comment:
```
#if [-e/run/systemd/system];then
#exit 0
#if
```
and:
```
#/sbin/hwclock --rtc=$dev --systz --badyear
#/sbin/hwclock --rtc=$dev --systz

```

Now final check:
```
sudo hwclock --verbose -r

```


For date check use:
```
date
```

For date setup use:
```
date --date="Nov 11 2020 13:12:10"
```

We can also setup RPi time based on RTC data:
```
sudo hwclock --systohc
```

Sync RPi time with NTP server:
```
sudo ntpdate 0.ru.pool.ntp.org
```
or (depend on you geolocation):
```
sudo ntpdate 0.de.pool.ntp.org
```

Write updated time to RTC:
```
sudo hwclock -w
```

??? We can also setup RPi time based on RTC data (indeed alias for `hwclock -w`):
```
sudo hwclock --systohc
```


We can also direct setup RPi time and date in RTC HW:
```
sudo hwclock --set --date "Sat Nov 11 08:31:24 PDT 2020"
```
or
```
sudo hwclock --set --date "11/11/2020 23:10:45"
```





