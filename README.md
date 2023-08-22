# How to install OpenCV 4.5.2 with CUDA 11.2 and CUDNN 8.2 in Ubuntu 20.04

First of all install update and upgrade your system:
    
        $ sudo apt update
        $ sudo apt upgrade
   
    
Then, install required libraries:

* Generic tools:

        $ sudo apt install build-essential cmake pkg-config unzip yasm git checkinstall
    
* Image I/O libs
    ``` 
    $ sudo apt install libjpeg-dev libpng-dev libtiff-dev
    ``` 
* Video/Audio Libs - FFMPEG, GSTREAMER, x264 and so on.
    ```
    $ sudo apt install libavcodec-dev libavformat-dev libswscale-dev libavresample-dev
    $ sudo apt install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
    $ sudo apt install libxvidcore-dev x264 libx264-dev libfaac-dev libmp3lame-dev libtheora-dev 
    $ sudo apt install libfaac-dev libmp3lame-dev libvorbis-dev
    ```
* OpenCore - Adaptive Multi Rate Narrow Band (AMRNB) and Wide Band (AMRWB) speech codec
    ```
    $ sudo apt install libopencore-amrnb-dev libopencore-amrwb-dev
    ```
    
* Cameras programming interface libs
    ```
    $ sudo apt-get install libdc1394-22 libdc1394-22-dev libxine2-dev libv4l-dev v4l-utils
    $ cd /usr/include/linux
    $ sudo ln -s -f ../libv4l1-videodev.h videodev.h
    $ cd ~
    ```

* GTK lib for the graphical user functionalites coming from OpenCV highghui module 
    ```
    $ sudo apt-get install libgtk-3-dev
    ```
* Python libraries for python3:
    ```
    $ sudo apt-get install python3-dev python3-pip
    $ sudo -H pip3 install -U pip numpy
    $ sudo apt install python3-testresources
    ```
* Parallelism library C++ for CPU
    ```
    $ sudo apt-get install libtbb-dev
    ```
* Optimization libraries for OpenCV
    ```
    $ sudo apt-get install libatlas-base-dev gfortran
    ```
* Optional libraries:
    ```
    $ sudo apt-get install libprotobuf-dev protobuf-compiler
    $ sudo apt-get install libgoogle-glog-dev libgflags-dev
    $ sudo apt-get install libgphoto2-dev libeigen3-dev libhdf5-dev doxygen
    ```

We will now proceed with the installation (see the Qt flag that is disabled to do not have conflicts with Qt5.0).

    $ cd ~/Downloads
    $ wget -O opencv.zip https://github.com/opencv/opencv/archive/refs/tags/4.5.2.zip
    $ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/refs/tags/4.5.2.zip
    $ unzip opencv.zip
    $ unzip opencv_contrib.zip
    
    $ echo "Create a virtual environtment for the python binding module (OPTIONAL)"
    $ sudo pip install virtualenv virtualenvwrapper
    $ sudo rm -rf ~/.cache/pip
    $ echo "Edit ~/.bashrc"
    $ export WORKON_HOME=$HOME/.virtualenvs
    $ export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
    $ source /usr/local/bin/virtualenvwrapper.sh
    $ mkvirtualenv cv -p python3
    $ pip install numpy
    
    $ echo "Procced with the installation"
    $ cd opencv-4.5.2
    $ mkdir build
    $ cd build
Configure following as per your requirements. I am installing in conda env so if you want to install in virtual env follow steps in Ref 1 at the end.
```
cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D WITH_TBB=ON \
-D ENABLE_FAST_MATH=1 \
-D CUDA_FAST_MATH=1 \
-D WITH_CUBLAS=1 \
-D WITH_CUDA=ON \
-D BUILD_opencv_cudacodec=OFF \
-D WITH_CUDNN=ON \
-D OPENCV_DNN_CUDA=ON \
-D CUDA_ARCH_BIN=7.5 \
-D WITH_V4L=ON \
-D WITH_QT=OFF \
-D WITH_OPENGL=ON \
-D WITH_GSTREAMER=ON \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D OPENCV_PC_FILE_NAME=opencv.pc \
-D OPENCV_ENABLE_NONFREE=ON \
-D OPENCV_PYTHON3_INSTALL_PATH=/home/talha/anaconda3/envs/test/lib/python3.9/site-packages \
-D PYTHON_EXECUTABLE=/home/talha/anaconda3/envs/test/bin/python \
-D OPENCV_EXTRA_MODULES_PATH=~/Downloads/opencv_contrib-4.5.2/modules \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D INSTALL_C_EXAMPLES=OFF \
-D BUILD_EXAMPLES=OFF ..
```

    
If you want also to use CUDNN you must include those flags (to set the correct value of CUDA_ARCH_BIN you must visit https://developer.nvidia.com/cuda-gpus and find the Compute Capability CC of your graphic card). If you have problems with the setting up of CUDDN check the *List of documented problems*:

	-D WITH_CUDNN=ON \
	-D OPENCV_DNN_CUDA=ON \
	-D CUDA_ARCH_BIN=7.5 \

Before the compilation you must check that CUDA has been enabled in the configuration summary printed on the screen. (If you have problems with the CUDA Architecture go to the end of the document).

```
--   NVIDIA CUDA:                   YES (ver 11.8, CUFFT CUBLAS FAST_MATH)
--     NVIDIA GPU arch:             75
--     NVIDIA PTX archs:
-- 
--   cuDNN:                         YES (ver 8.6.0)

```

I've included below the output of my [configuration](#configuration-information).

If it is fine proceed with the compilation (Use nproc to know the number of cpu cores):
    
    $ nproc
    $ make -j8
    $ sudo make install

Include the libs in your environment    
    
    $ sudo /bin/bash -c 'echo "/usr/local/lib" >> /etc/ld.so.conf.d/opencv.conf'
    $ sudo ldconfig

*If you have any other problem try updating the nvidia drivers.*

### Source
- [Raulqf](https://gist.github.com/raulqf/f42c718a658cddc16f9df07ecc627be7#file-install_opencv4_cuda11_cudnn8-md)
- [pyimagesearch](https://www.pyimagesearch.com/2018/08/15/how-to-install-opencv-4-on-ubuntu/)
- [learnopencv](https://www.learnopencv.com/install-opencv-4-on-ubuntu-18-04/)
- [Tzu-cheng](https://chuangtc.com/ParallelComputing/OpenCV_Nvidia_CUDA_Setup.php)
- [Medium](https://medium.com/@debugvn/installing-opencv-3-3-0-on-ubuntu-16-04-lts-7db376f93961)
- [Previous Gist](https://gist.github.com/raulqf/a3caa97db3f8760af33266a1475d0e5e)
