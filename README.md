# RaspberryPi Gotchas

## video

* **Libx264** is a software codec so will never be hardware accelerated.
* **h264_omx** is the hardware accelerated codec in ffmpeg, so I'm assuming they've enabled the relevant build options for that.

* https://things.bleu255.com/moddingfridays/Hardware_Accelerated_Raspberry_Pi_on_Raspbian
* [command line tool for capturing video with the camera module](https://www.raspberrypi.org/documentation/usage/camera/raspicam/raspivid.md)


It says Using OMX.broadcom.video_encode, but %CPU goes to 99% anyway, when a comparable gstreamer command uses around 30 %CPU...
`ffmpeg -f mjpeg -i "http://user:pass@xxx.xxx.x.xx:xxx/mjpeg.cgi" -c:v h264_omx -an output.mp4`

### cam 

* [Camination::1 Stop Motion, Timelapse and Animation using WebCams](http://www.sparkslabs.com/camination/)
* [Camination::2 Stop Motion, Timelapse and Animation using Webcams](https://github.com/sparkslabs/camination)


Even for video conversion, this is millions times faster than using -codec:v libx264 or -codec:v libx265:
`ffmpeg -f v4l2 -video_size 1280x800 -i /dev/video0 -codec:v h264_omx -b:v 2048k webcam.mkv`

Struggling some hours with VLC, I successfully found a way to tell VLC to stream using ffmpeg and mpeg4 codec:
`cvlc v4l2:///dev/video0:width=1280:height=800 --sout '#transcode{vcodec=mp4v,scale=Auto,acodec=none,scodec=none}:http{mux=ffmpeg{mux=avi},dst=:8080/}' -vvv`
It uses ffmpeg and because of "vcodec=mp4v" it seems to use the equivalent of "-codec:v mpeg4" into ffmpeg.


Final pipeline:
`ffmpeg -f v4l2 -video_size 1280x800 -i /dev/video0 -codec:v h264_omx -b:v 2048k -f matroska - | cvlc - --sout '#http{mux=ffmpeg{mux=flv},dst=:8080/}' -vvv`


Here is my entire memo, too (i'll keep it safe into my linux/things recipes):
```
#Listing available ffmpeg codecs (encoding and decoding)
ffmpeg -decoders | grep mmal
ffmpeg -encoders | grep omx

#Listing available formats and resolutions for the capture input :
ffmpeg -f v4l2 -list_formats all -i /dev/video0

#Doing a raw "capture and place to file" with input resolution specified (still using mux depending on file extension)
ffmpeg -f v4l2 -video_size 1280x800 -i /dev/video0 -codec copy output.avi
```

VLC Only stuff :
```
#Raw capture :
cvlc v4l2:///dev/video0:width=1280:height=800 --sout '#file{dst=capture.avi}' -vvv

#Streaming with differnt encoding (not hardware, unfortunately) :
cvlc v4l2:///dev/video0:width=1280:height=800 --sout '#transcode{vcodec=h264,scale=Auto,acodec=none,scodec=none}:http{mux=ffmpeg{mux=flv},dst=:8080/}' -vvv
cvlc v4l2:///dev/video0:width=1280:height=800 --sout '#transcode{vcodec=mp2v,scale=Auto,acodec=none,scodec=none}:http{mux=ffmpeg{mux=avi},dst=:8080/}' -vvv
cvlc v4l2:///dev/video0:width=1280:height=800 --sout '#transcode{vcodec=mp4v,scale=Auto,acodec=none,scodec=none}:http{mux=ffmpeg{mux=avi},dst=:8080/}' -vvv

#With some parameters about videobitrate (vb 800 means 800k) and framerate (5 fps)
cvlc v4l2:///dev/video0:width=1280:height=800 --sout '#transcode{vcodec=mp4v,vb=512,scale=Auto,acodec=none,scodec=none,fps=5}:http{mux=ffmpeg{mux=avi},dst=:8080/}' -vvv
```


### ffmpeg HW acceleration


You could use gstreamer. Hardware decoding and encoding is supported by the gstreamer1.0 omx-plugin, which is provided by the Foundation repository.

* `--enable-omx-rpi`
* `--enable-decoder=h264_mmal`



* https://trac.ffmpeg.org/wiki/HWAccelIntro
* [(RPi) Compile FFmpeg with the OpenMAX H.264 GPU acceleration](https://github.com/legotheboss/YouTube-files/wiki/(RPi)-Compile-FFmpeg-with-the-OpenMAX-H.264-GPU-acceleration)

I'm trying to livestream one of my cameras to Youtube, which was working well previously on Raspbian:
`ffmpeg -nostdin -r 14.5 -i http://192.168.1.250:8080/html/cam_pic_new.php?pDelay=66666 -f lavfi -i anullsrc -b:v 2M -b:a 128k -c:v h264_omx -ar 22050 -f flv rtmp://a.rtmp.youtube.com/live2/<CODE>`

build a custom version of FFmpeg to use the hardware encoder on Raspberry Pi. Make sure the `--enable-omx` and `--enable-omx-rpi` flag is enabled when configuring the FFmpeg build. This flag indicates that build FFmpeg with Raspberry Pi-specific OpenMAX encoder, which is silghtly different from the normal version of the OpenMAX encoder (e.g. they dependend on different .so library).

```
sudo apt-get install libomxil-bellagio-dev -y
git clone https://github.com/FFmpeg/FFmpeg.git
cd FFmpeg
./configure --arch=armel --target-os=linux --enable-gpl --enable-omx --enable-omx-rpi --enable-nonfree
make -j4
sudo make install
```



Finally recompiled ffmpeg successfully this way:
```
    sudo apt-get install libtool-bin
    mkdir /home/pi/Desktop/sources
    mkdir /home/pi/Desktop/sources/arm

    cd /home/pi/Desktop/sources
    wget http://tipok.org.ua/downloads/media/aac ... 0.2.tar.gz
    tar -xzf libaacplus-2.0.2.tar.gz
    cd libaacplus-2.0.2
    #./autogen.sh --with-parameter-expansion-string-replace-capable-shell=/bin/bash --enable-static --host=arm-unknown-linux-gnueabi --prefix=/home/pi/Desktop/sources/arm
    ./autogen.sh --with-parameter-expansion-string-replace-capable-shell=/bin/bash --enable-static --prefix=/home/pi/Desktop/sources/arm
    make
    sudo make install

    cd /home/pi/Desktop/sources
    git clone git://git.videolan.org/x264
    cd x264
    ./configure --enable-static --enable-shared
    make
    sudo make install

    sudo apt-get install git libasound2-dev libav-tools
    sudo apt-get install yasm libvpx. libx264. libxcb.
    sudo apt-get install libmp3lame-dev libmp3lame0

    cd /usr/src
    sudo git clone https://git.ffmpeg.org/ffmpeg.git

    cd ffmpeg
    ./configure --enable-gpl --enable-libx264 --enable-mmal --enable-omx-rpi --enable-omx --enable-libxcb --enable-libmp3lame --enable-nonfree
    make
    ffmpeg --list-hwaccels
    #sudo make install
    #ldconfig
```



Steps to build FFmpeg for RPi with the h264_omx encoder

* http://www.redhenlab.org/home/the-cognitive-core-research-topics-in-red-hen/the-barnyard/hardware-encoding-with-the-raspberry-pi



#### RPi 4

```
sudo gst-launch-1.0 -v v4l2src device=/dev/video0 do-timestamp=true ! tee name=tee ! capsfilter caps="video/x-raw,width=1920,height=1080,framerate=30/1,bitrate=40000" ! queue ! videoflip method=rotate-180 ! videoconvert ! videorate ! queue ! omxh264enc ! queue ! avimux ! queue ! filesink location = test.h264 qos=true --gst-debug=GST_QOS:5 ! queue ! videoscale method=1 ! videoconvert ! capsfilter caps="video/x-raw,width=256,height=144,framerate=30/1" ! queue ! videoflip method = rotate-180 ! queue ! omxh264enc ! queue ! h264parse ! mpegtsmux ! rtpmp2tpay ! multiudpsink clients=192.168.5.255:1234 ttl=1 auto-multicast=true
```

* [Raspberry Pi 4 - how to enable hardware-accelerated (gpu) h.264 decoding in ffmpeg?](https://www.raspberrypi.org/forums/viewtopic.php?t=265764)

`ffmpeg -c:v h264_mmal -i "rtsp://192.168.1.87/stream0:554?username=admin&password=admin" -vf fps=1/60 img%03d.jpg`

`raspivid -md 7 -w 640 -h 480 -n -o - -t 0 -fps 50 | gst-launch-1.0 fdsrc ! h264parse ! omxh264dec ! videoconvert ! fbdevsink`

* https://freedesktop.org/wiki/GstOpenMAX/


### pipe

* [A frontend for ffmpeg using only pipes](https://github.com/kanryu/pipeffmpeg)
* https://github.com/kanryu/pipeffmpeg/blob/master/pipeffmpeg.py





## audio


## display



### Screen recording on Pi

`ffmpeg -f x11grab -r 30 -s cif -i :0.0 /tmp/out.mpg`


* [Utility to take a snapshot of the Raspberry Pi screen and save it as a PNG file](https://github.com/AndrewFromMelbourne/raspi2png)



