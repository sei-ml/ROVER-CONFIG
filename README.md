# ROVER-CONFIG
### Resize Partition with gparted
```
sudo apt install gparted
sudo gparted
```
### Adjust PATH
* Add Cuda to PATH (append to ~/.bashrc)

```
export CUDA_HOME=/usr/local/cuda
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64
export PATH=$PATH:$CUDA_HOME/bin
```

* Required for USB issues on Nvidia Jetson Xavier NX (append to ~/.bashrc)

```
export OPENBLAS_CORETYPE=ARMV8
```

source: https://stackoverflow.com/questions/65631801/illegal-instructioncore-dumped-error-on-jetson-nano

### Upgrade Cmake

* check cmake version

```
cmake --version
```

* upgrade cmake to version 3.20

```
#!/bin/bash

version=3.20
build=1
mkdir ~/temp
cd ~/temp
wget https://cmake.org/files/v$version/cmake-$version.$build.tar.gz
tar -xzvf cmake-$version.$build.tar.gz
cd cmake-$version.$build/
```

* install and extract source by running

```
./bootstrap
make -j$(nproc)
sudo make install
```

```
cmake --version
```

source: https://askubuntu.com/questions/355565/how-do-i-install-the-latest-version-of-cmake-from-the-command-line

* Build and Install Open3D from source. 
* Tested on Nvidia Jetson Xavier NX supporting Azure Kinect. 
* * Open3D (0.15.1)
* * Azure Kinect SDK (1.4.1)
* * cmake (3.24.1)
* * JetPack (4.4)
* * Ubuntu (18.04)
* * Python3 (3.6.9)

```
git clone --recursive https://github.com/isl-org/Open3D.git
git checkout v0.15.1
```

* install dependencies

```
./util/install_deps_ubuntu.sh
sudo apt-get install -y libc++-7-dev libc++abi-7-dev clang-7 python-pip3 ccache gfortran
sudo -H pip3 install --upgrade pip==20.3
pip3 install matplotlib
```

* build
```
cd Open3D && mkdir build && cd build
cmake -DBUILD_AZURE_KINECT=ON -DBUILD_CUDA_MODULE=ON -DBUILD_GUI=ON ..

### Alternative Method 1
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DBUILD_CUDA_MODULE=ON -DBUILD_GUI=ON -DBUILD_TENSORFLOW_OPS=OFF -DBUILD_PYTORCH_OPS=OFF -DBUILD_UNIT_TEST=ON -DCMAKE_INSTALL_PREFIX=~/open3d_install -DBUILD_AZURE_KINECT=ON -DUSE_SYSTEM_LIBREALSENSE=ON -DGLIBCXX_USE_CXX11_ABI=ON -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ -DBUILD_FILAMENT_FROM_SOURCE=ON -DPYTHON_EXECUTABLE=$(which python3) -CMAKE_CUDA_COMPILER=/usr/local/cuda/bin/nvcc ..

### Alternative Method 2
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DBUILD_CUDA_MODULE=ON -DBUILD_GUI=ON -DBUILD_TENSORFLOW_OPS=OFF -DBUILD_PYTORCH_OPS=OFF -DBUILD_UNIT_TEST=ON -DCMAKE_INSTALL_PREFIX=~/open3d_install -DBUILD_AZURE_KINECT=ON -DUSE_SYSTEM_LIBREALSENSE=ON -DGLIBCXX_USE_CXX11_ABI=ON -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ -DBUILD_FILAMENT_FROM_SOURCE=ON -DPYTHON_EXECUTABLE=$(which python3) ..

make -j$(nproc)
sudo make install
make install-pip-package -j$(nproc)
```
source: http://www.open3d.org/docs/release/arm.html#install-dependencies

* Run Open3D GUI
```
./bin/Open3D/Open3D
```

* 3D Reconstruction with Azure Kinect and Open3D

```
python3 ~/Open3D/examples/python/reconstruction_system/sensors/azure_kinect_recorder.py --output frames.mkv

python3 ~/Open3D/examples/python/reconstruction_system/sensors/azure_kinect_mkv_reader.py --input frames.mkv --output frames

python3 ~/Open3D/examples/python/reconstruction_system/run_system.py --config ./frames/config.json --make

python3 ~/Open3D/examples/python/reconstruction_system/run_system.py --config ./frames/config.json --register

python3 ~/Open3D/examples/python/reconstruction_system/run_system.py --config ./frames/config.json --refine

python3 ~/Open3D/examples/python/reconstruction_system/run_system.py --config ./frames/config.json --integrate
```

### Enable screen recording and streaming with gstreamer
```
sudo apt-get install libgstrtspserver-1.0 libgstreamer1.0-dev
wget https://gstreamer.freedesktop.org/src/gst-rtsp/gst-rtsp-server-1.14.1.tar.xz
tar -xvf gst-rtsp-server-1.14.1.tar.xz
cd  gst-rtsp-server-1.14.1
cd examples
gcc test-launch.c -o test-launch $(pkg-config --cflags --libs gstreamer-1.0 gstreamer-rtsp-server-1.0)
```

* Launch RTSP Server 
```
./test-launch "ximagesrc use-damage=0 ! nvvidconv ! omxh264enc ! video/x-h264, profile=baseline ! h264parse ! video/x-h264, stream-format=byte-stream ! rtph264pay name=pay0 pt=96 "
```

source: https://forums.developer.nvidia.com/t/streaming-desktop-with-rtsp-gstreamer-server/143765/19?page=2

* Aliases
```
alias stream='/home/doogi/gst-rtsp-server-1.14.1/examples/test-launch "ximagesrc use-damage=0 ! nvvidconv ! omxh264enc ! video/x-h264, profile=baseline ! h264parse ! video/x-h264, stream-format=byte-stream ! rtph264pay name=pay0 pt=96 "'
alias azure='python3 /home/doogi/Open3D/examples/python/reconstruction_system/sensors/azure_kinect_recorder.py --output mini.mkv'
```

### Setup VNC server
```
mkdir -p ~/.config/autostart
cp /usr/share/applications/vino-server.desktop ~/.config/autostart/.
gsettings set org.gnome.Vino prompt-enabled false
gsettings set org.gnome.Vino require-encryption false
# Replace thepassword with your desired password
gsettings set org.gnome.Vino authentication-methods "['vnc']"
gsettings set org.gnome.Vino vnc-password $(echo -n 'thepassword'|base64)
sudo reboot
```



