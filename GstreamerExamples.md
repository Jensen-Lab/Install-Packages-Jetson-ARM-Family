# Working with GStreamer on the Nano
Working with GStreamer on the Jetson nano can be really painful at times, but due to the very nice work by Nvidia to utilize the graphics acceleration provided by their tegra chip, really incredible things can be achieved such as viewing and encoding 4K video at 30FPS.

A great place to start learning about using the pipeline setup that Gstreamer has available can be found at:
https://developer.nvidia.com/embedded/learn/tutorials/first-picture-csi-usb-camera

## Running some more complex pipelines with GStreamer
All the things I have put together come from this documentation: 
https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/accelerated_gstreamer.html#wwpID0E06P0HA

Jetsons can do some impressive things, but they're highly limited in terms of their connectivity to the SD card and other peripherals so streaming video can be a challenge. My suggestion is to use the highly optimised encoding pipelines to their fullest to make projects possible.

# Saving 4K video from RPi HQ cam
A pipeline to save encoded 4K video to the Jetson from the RPi camera:
```
gst-launch-1.0 nvarguscamerasrc ! 'video/x-raw(memory:NVMM), format=(string)NV12, width=(int)4032, height=(int)3040, framerate=(fraction)30/1' ! nvvidconv !   'video/x-raw(memory:NVMM), format=(string)I420' ! omxh264enc !   'video/x-h264, stream-format=(string)byte-stream' ! h264parse ! qtmux ! filesink location=test2.mp4 -e
```
This can be further modified if you'd like to crop the video to 1920x1080 by just changing the width and height in the first part of the stream
This video can also be scaled down for other purposes by adding in a scaling line before the encoding as follows:
```
gst-launch-1.0 nvarguscamerasrc ! 'video/x-raw(memory:NVMM), format=(string)NV12, width=(int)4032, height=(int)3040, framerate=(fraction)30/1' ! nvvidconv !  'video/x-raw(memory:NVMM),width=(int)1280, height=(int)960, format=(string)NV12' ! omxh264enc !   'video/x-h264, stream-format=(string)byte-stream' ! h264parse ! qtmux ! filesink location=test2.mp4 -e
```

Modified from: https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/accelerated_gstreamer.html#wwpID0E0YP0HA

# Opening 4K video from RPi HQ cam
A pipeline to open encoded video on the Jetson that runs at full frame:
```
gst-launch-1.0 filesrc location=test2.mp4 ! qtdemux name=demux ! h264parse ! omxh264dec ! nvoverlaysink -e
```
Modified from:
https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/accelerated_gstreamer.html#wwpID0E0YP0HA

