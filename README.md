# How to install OpenCV 4.5.2 with CUDA 11.8 and CUDNN 8.6 in Ubuntu 20.04

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
* Install Anaconda and creat a conda env:
    ```
    $ conda create -n <name> python=3.8.2
    $ conda install numpy
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
    
    $ echo "Procced with the installation"
    $ cd opencv-4.5.2
    $ mkdir build
    $ cd build
Configure following as per your requirements. I am installing in conda env so if you want to install in virtual env follow steps in Ref 1 at the end. *Edit paths accordingly*
```
cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/home/<user-name>/anaconda3/envs/<env-name> \
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
-D OPENCV_PYTHON3_INSTALL_PATH=/home/<user-name>/anaconda3/envs/<env-name>/lib/python3.8/site-packages \
-D PYTHON_EXECUTABLE=/home/<user-name>/anaconda3/envs/<env-name>/bin/python \
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
....
--   Python 3:
--     Interpreter:                 /home/talha/anaconda3/envs/test/bin/python (ver 3.9)
--     Libraries:                   ...
--     numpy:                       ...
--     install path:                /home/talha/anaconda3/envs/test/lib/python3.9/site-packages

```

I've included below the output of my [configuration](#configuration-information).

If it is fine proceed with the compilation (Use nproc to know the number of cpu cores):
    
    $ nproc
    $ make -j8
    $ sudo make install

You'll need to update the library search path to include the lib directory of your conda environment. You can do this by adding the following lines to your ~/.bashrc file:
```bash
export LD_LIBRARY_PATH=/home/talha/anaconda3/envs/test/lib:$LD_LIBRARY_PATH
```
After adding these lines, restart your terminal or run source ~/.bashrc to update the library search path.

You can also verify if the dir is present in your env

```
ls /home/<user>/anaconda3/envs/<env-name>/lib/python3.8/site-packages | grep cv2
```
The output with be just `cv2` if nothing then it did'nt install in your conda env.
 Here's a breakdown of what each command does:

1. `make -j8`: This command compiles the OpenCV source code. The `-j8` flag tells `make` to use 8 parallel jobs, which can speed up the compilation process if you have a multi-core processor.

2. `sudo make install`: This command installs the compiled OpenCV libraries and headers to the location specified by the `CMAKE_INSTALL_PREFIX` option in the `cmake` command (in this case, `/usr/local`).

3. `sudo /bin/bash -c 'echo "/usr/local/lib" >> /etc/ld.so.conf.d/opencv.conf'`: This command adds the path `/usr/local/lib` to the file `/etc/ld.so.conf.d/opencv.conf`, which is used by the dynamic linker to find shared libraries. This step is necessary because you installed OpenCV to `/usr/local`, which may not be in the default library search path.

4. `sudo ldconfig`: This command updates the dynamic linker cache with the new library path. This step is necessary for the system to find the OpenCV libraries when you run programs that use them.

After running these commands, you should be able to use OpenCV with CUDA support in your `test` conda environment. Just make sure to activate the environment with `conda activate test` before using OpenCV in Python.
*If you have any other problem try updating the nvidia drivers.*

### Source
- [Raulqf-Virtusl Env Installtion](https://gist.github.com/raulqf/f42c718a658cddc16f9df07ecc627be7#file-install_opencv4_cuda11_cudnn8-md)
- [pyimagesearch](https://www.pyimagesearch.com/2018/08/15/how-to-install-opencv-4-on-ubuntu/)
- [learnopencv](https://www.learnopencv.com/install-opencv-4-on-ubuntu-18-04/)
- [Tzu-cheng](https://chuangtc.com/ParallelComputing/OpenCV_Nvidia_CUDA_Setup.php)
- [Medium](https://medium.com/@debugvn/installing-opencv-3-3-0-on-ubuntu-16-04-lts-7db376f93961)
- [Previous Gist](https://gist.github.com/raulqf/a3caa97db3f8760af33266a1475d0e5e)
