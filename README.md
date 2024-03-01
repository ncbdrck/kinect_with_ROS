# Working with the Kinect V2

We need a Kinect bridge to connect Kinect to the PC. We can buy one from Amazon.
```
# Xbox Kinect Adapter for Xbox One S/Xbox One X Windows 8/8.1/10 Power AC Adapter PC Development Kit with UK Plug

https://www.amazon.co.uk/gp/product/B07HT4224B
```

**Install Guidelines**

This is a guide on installing it in Ubuntu and working with Python. So, the packages for working with ROS are missing in the below version. 
```
https://www.notaboutmy.life/posts/run-kinect-2-on-ubuntu-20-lts/
```
**Note:** Instruction mentioned in this guide is also included below. So you can directly skip to the next step.

## Installation Steps

**Install Build Tools and Libraries**

```
sudo apt-get install build-essential cmake pkg-config  -y  # Build tools
sudo apt-get install libusb-1.0-0-dev          # libusb
sudo apt-get install libturbojpeg0-dev -y      # TurboJPEG
sudo apt-get install libglfw3-dev -y           # OpenGL
sudo apt-get install libva-dev libjpeg-dev -y  # VAAPI for Intel only
sudo apt-get install libopenni2-dev -y         # OpenNI2
```

**CUDA** (only if you have a Nvidia GPU)
```
https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#ubuntu-installation
```

At the moment (16/05/2022) following are the steps to install cuda
```
sudo apt update
sudo apt install nvidia-cuda-toolkit

wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/3bf863cc.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
sudo apt-get update
sudo apt-get -y install cuda

sudo apt-get install linux-headers-$(uname -r)
sudo apt-get update
sudo apt-get install cuda
sudo reboot

echo 'export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}' >> ~/.bashrc
```
Next, install the CUDA code samples. These are required by the `Libfreenect` installation. The CUDA Samples are bundled with the `CUDA toolkit` installation bundle on their website. Since we utilized apt to install the toolkit, the samples are missing. We can fix this by running the CUDA installation runfile and selecting to install **ONLY** the samples.

We can do this by first downloading the runfile.
```
wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run
sudo sh cuda_10.1.243_418.87.00_linux.run --override
```
A terminal-based GUI will appear. It may state that CUDA drivers are already installed. Just proceed, we are not installing drivers. Using the arrow keys to navigate, perform the following:

  1. Type `accept` to accept the EULA
  2. Uncheck Driver, CUDA Toolkit 10.1, CUDA Demo Suite 10.1. CUDA Documentation 10.1 (only CUDA Samples 10.1 should be checked)
  3. Hover over `Install` and hit your `Enter` key

By default, the samples should be installed in your home directory.

Once the samples are installed, we need to update our `CPATH`, so `Libfreenect` knows where to look for them. So for `~/.bashrc`:
```
echo 'export CPATH=$CPATH:$HOME/NVIDIA_CUDA-10.1_Samples/common/inc' >> ~/.bashrc
source $HOME/.bashrc
```

**Build Libfreenect 2**
```
cd
git clone https://github.com/OpenKinect/libfreenect2.git
cd libfreenect2
mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=$HOME/freenect2
make
make install
sudo cp ../platform/linux/udev/90-kinect2.rules /etc/udev/rules.d/
```

Finally, replug the Kinect and run the test program. The test program can be found in the compiled project’s bin folder of the repository.
```
cd ~/libfreenect2/build
./bin/Protonect
```

**IAI Kinect2 for OpenCV 4**

We can find the installation instructions on the GitHub page. However, There's an error in the instructions. Replace ```git clone https://github.com/code-iai/iai_kinect2.git``` with ```git clone https://github.com/paul-shuvo/iai_kinect2_opencv4.git```

```
cd ~/catkin_ws/src/
git clone https://github.com/paul-shuvo/iai_kinect2_opencv4.git
cd iai_kinect2_opencv4
rosdep install -r --from-paths .
cd ~/catkin_ws
catkin_make -DCMAKE_BUILD_TYPE="Release"

# If you get the following error pcl_conversions build error: PCL requires C++14 or above, 
# Go to your CMakeLists.txt in the src folder and add the line set( CMAKE_CXX_STANDARD 14) on the very top, and rebuild.

source devel/setup.bash
```

Now, we can start the service by
```
roslaunch kinect2_bridge kinect2_bridge.launch
```

Now we can check if it is working by
```
# normal
rosrun kinect2_viewer kinect2_viewer

# For the superimposed images: 
rosrun kinect2_viewer kinect2_viewer hd image
    
# For the point cloud images: 
rosrun kinect2_viewer kinect2_viewer hd cloud

# For the superimposed and point cloud images: 
rosrun kinect2_viewer kinect2_viewer hd both
```

## Error Handling
If you get an error. We need to do the following **unknown libva error**

```
# https://github.com/OpenKinect/libfreenect2/issues/1123
echo "export LIBVA_DRIVER_NAME=i965" >> ~/.bashrc
source ~/.bashrc
```
