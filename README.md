# Getting started with the Jetson Nano
The objective is to give you clear instruction on how to install packages and cameras on the Jetson Nano.

Matt here: I've done pretty much every step on this list of installs on both jetpack 4.4.1. I do not recommend that you use JetPack 4.6 as they've heavily modified
           Gstreamer and it no longer works with any of the pre-built code for streaming it to OpenCV.
           With this set of dependencies, it was possible to run pretty much everything you could ever want on these platforms
           One other thing to consider is the system time. The jetson family is not great at keeping it's time, so if you're getting errors, fix it and try again.
           
## Some info here about getting the Raspberry Pi HQ Camera up and running
First off, the camera will need some small modifications to work properly with the Jetsons. Just follow the great tutorial by RidgeRun:
https://developer.ridgerun.com/wiki/index.php?title=Raspberry_Pi_HQ_camera_IMX477_Linux_driver_for_Jetson
If you're running JetPack 4.4.1 or 4.6, then the instructions to install the drivers for the cameras don't work. Just head down to "Installing the camera drivers"

## Installing the camera drivers
If you're using JetPack 4.6, you can just go in to the driver selection and change which camera you're set up for. Otherwise skip to installing the driver.
```
sudo /opt/nvidia/jetson-io/jetson-io.py
```
To check whether the camera is recognised with the computer you can run the following command:
```
$ ls /dev | grep video
```
If it finds video0 or video1 you're in business, but otherwise you'll need to get the driver installed.
```
cd $HOME
wget https://github.com/ArduCAM/MIPI_Camera/releases/download/v0.0.3/install_full.sh
chmod +x install_full.sh
sudo ./install_full.sh -m imx477
```
If you want to get change cameras back to the original RPi camera (IMX219), you can go into the file and change which camera you're using. 
```
sudo /opt/arducam/jetson-io/jetson-io.py
```
All this info can also be found at https://www.arducam.com/docs/camera-for-jetson-nano/native-jetson-cameras-imx219-imx477/imx477-how-to-install-the-driver/

## Dependencies Installation
Before performing any installations, you may need to install the basic dependencies first.
```
$ sudo apt install cmake python3-pip libhdf5-serial-dev hdf5-tools libhdf5-dev libblas-dev liblapack-dev libatlas-base-dev gfortran screen
$ sudo python3 -m pip install wget Cython numpy==1.19.4 scipy==1.15.4
```

## PyCUDA Installation
```
$ sudo apt-get install libboost-all-dev python-numpy build-essential python-dev python-setuptools libboost-python-dev libboost-thread-dev
```
You need to download PyCUDA from https://pypi.org/project/pycuda/#files. In the same directory of your PyCUDA download, run this terminal
```
$ tar xzvf pycuda-VERSION.tar.gz
$ cd pycuda-VERSION
```
Open configure.py and change the /usr/bin/env python into /usr/bin/env python3 (Matt here: I haven't needed to do this)
```
$ ./configure.py
$ make -j4
$ sudo python3 setup.py install
$ sudo pip3 install .
```

## LLVM Installation
Numba needs the installation of LLVM to the system. It's the back end compiler for llvmlite that is the key dependency for numba.
This will take a while (a few hours) and most things will crash in the mean time, so leave it alone.
*A suggestion* You can try also running the jetson GUI-less so that you can make this work a bunch faster and free up memory. Go to 'CuPy Installation' for details
```
$ wget https://github.com/llvm/llvm-project/releases/download/llvmorg-9.0.1/llvm-9.0.1.src.tar.xz
$ tar -xvf llvm-9.0.1.src.tar.xz
$ cd llvm-9.0.1.src
$ mkdir llvm_build_dir
$ cd llvm_build_dir/
$ cmake ../ -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD="ARM;X86;AArch64"
$ make -j4 %% Matt here again, the 4 here is telling it how many processor cores to use, for speed 4, for stability, use 3
$ sudo make install
$ cd bin/
$ echo "export LLVM_CONFIG=\""`pwd`"/llvm-config\"" >> ~/.bashrc
$ echo "alias llvm='"`pwd`"/llvm-lit'" >> ~/.bashrc
$ source ~/.bashrc
$ sudo pip3 install llvmlite==0.33.0
```
Restart the system after this as the system can be very unstable after this installation.


## TBB update
https://github.com/jefflgaol/Install-Packages-Jetson-ARM-Family/issues/2
Thank you for surefyyq for the solution regarding the TBB issue.
This needs to be updated to the most recent version as the jetsons only come with the 2019 version.
As per the LLVM compile, this will take some time and monopolise the system resources. 
```
$ git clone https://github.com/wjakob/tbb.git
$ cd tbb/build
$ cmake ..
$ make -j
$ sudo make install
```

## Numba Installation
Before proceeding to the Numba installation, you need to perform LLVM installation above first since Numba is heavily rely on LLVM installation.
```
$ sudo pip3 install numba
```

## Protobuf Installation
Protobuf is needed for ONNX, and it'll throw a hissy fit if you don't get this done first
```
sudo apt-get install protobuf-compiler libprotoc-dev
$ sudo pip3 install protobuf
```
If that doesn't work, you may build from source: (Haven't needed to do it, but potentially will need to in the future)
```
$ git clone https://github.com/protocolbuffers/protobuf.git
$ cd protobuf
$ git submodule update --init --recursive
$ ./autogen.sh
$ ./configure
$ make -j4
$ make check
$ sudo make install
$ sudo ldconfig
```

## ONNX Installation
Before proceeding to the ONNX installation, you need to perform Protobuf installation above first since ONNX heavily relies on Protobuf installation.
```
$ sudo pip3 install onnx
```

## Keras Installation
```
$ sudo apt-get install libhdf5-serial-dev hdf5-tools libhdf5-dev
$ sudo apt-get install libblas-dev liblapack-dev libatlas-base-dev gfortran
$ sudo pip3 install scipy
$ sudo pip3 install keras
```

## CuPy Installation
Installing CuPy is a bit of a pain. It will quickly crash out your memory, so you'll need to follow these instructions to the letter so it's actually possible.
```
$ sudo systemctl set-default multi-user.target
```
Then reboot your system into just the GUI-less system
```
$ cd /home/jetl/Downloads
$ git clone https://github.com/cupy/cupy.git
$ cd /cupy
$ sudo -H python3 -m pip -v install .
```
Now after 2 hours *hopefully* you'll have cupy installed and you should be good to go!
To get the GUI back you just need to put in the following command and reboot
```
sudo systemctl set-default graphical.target
reboot
```
## Installing FastMOT
FastMOT is a great little setup for multiple object tracking, and has been built for the jetson systems and x86 systems. 
For installation I'd just follow the instructions at his website: https://github.com/GeekAlexis/FastMOT
The instructions are relatively comprehensive for installing on the jetson system. For the x86 systems, the docker setup required some serious tinkering...

## My suggestions regarding retraining YOLO
The FastMOT system utilizes YOLOv4 to function currently. It works okay, but on the Jetson it's very slow. My suggestion is to go to another system that has a dedicated CUDA GPU and install Darknet and train there. Otherwise, there are many re-incarnations of YOLOv4 that can be used that have been optimised for other platforms. I would suggest this as Darknet is painfully slow for retraining YOLOv4, taking 6hrs for a 10min training session with Tensorflow/Keras.
For more information on this platform, go to https://github.com/AlexeyAB/darknet

## Tensorflow Installation
```
$ sudo apt-get install libhdf5-serial-dev hdf5-tools libhdf5-dev zlib1g-dev zip libjpeg8-dev
$ sudo pip3 install -U numpy==1.16.1 future==0.17.1 mock==3.0.5 h5py==2.9.0 keras_preprocessing==1.0.5 keras_applications==1.0.6 enum34 futures testresources setuptools protobuf
```
If you have trouble installing Protobuf, you may take a look at the Protobuf installation part above. If everything's fine, you may proceed. Now, take a look at this https://developer.download.nvidia.com/compute/redist/jp/. You may find a lot of JetPack versions and you need to choose one based on your preference. In this tutorial, I used v411 since it has compatibility with Python 2.7 and Python 3.6 installation. 
```
$ sudo pip3 install --pre --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v411 tensorflow-gpu
```
If you run into a problem when execute the code above like this:
```
Exception:
Traceback (most recent call last):
  ...
  File "/usr/share/python-wheels/requests-2.18.4-py2.py3-none-any.whl/requests/models.py", line 935, in raise_for_status
    raise HTTPError(http_error_msg, response=self)
requests.exceptions.HTTPError: 404 Client Error: Not Found for url: https://developer.download.nvidia.com/compute/redist/jp/v411/grpcio/
```
then you should download the Tensorflow installation manually. Previously, when I executed this ```sudo pip3 install --pre --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v411 tensorflow-gpu```, it already collected tensorflow-gpu and showed an output like this:
```
Collecting tensorflow-gpu
  Downloading https://developer.download.nvidia.com/compute/redist/jp/v411/tensorflow-gpu/tensorflow_gpu-1.13.0rc0+nv19.2-cp36-cp36m-linux_aarch64.whl (204.6MB)
    100% |████████████████████████████████| 204.6MB 4.6kB/s
```
So, I downloaded tensorflow_gpu-1.13.0rc0+nv19.2-cp36-cp36m-linux_aarch64.whl. From the download directory, I ran this inside the terminal:
```
$ sudo pip3 install tensorflow_gpu-1.13.0rc0+nv19.2-cp36-cp36m-linux_aarch64.whl
```

## Keeping time on the Jetson

Just copy and paste this to start installing the service. It runs when you boot up the machine and turns off.

$ sudo wget -O /etc/network/if-up.d/date-sync https://raw.githubusercontent.com/Jensen-Lab/Install-Packages-Jetson-ARM-Family/master/date-time.service
$ sudo chmod +x /etc/network/if-up.d/date-sync
$ sudo systemctl enable date-sync.service
$ sudo systemctl start date-sync.service

Every time you reboot now, if there's internet, you will refresh the time server.
Otherwise, you can always start this service and it'll fix up the time
