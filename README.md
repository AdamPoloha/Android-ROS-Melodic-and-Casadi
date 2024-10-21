# Android-ROS-Melodic-and-Casadi

Install Termux and Termux-Api
Must have root and BusyBox (Does not have to be Magisk)

https://github.com/LinuxDroidMaster/Termux-Desktops/blob/main/Documentation/chroot/ubuntu_chroot.md

In Termux: (You can use ssh)

pkg update
pkg upgrade
pkg install tur-repo
pkg install pulseaudio
pkg install proot-distro
pkg install wget
pkg install gitrosdep init
rosdep update
pkg install x11-repo
pkg install root-repo
pkg install termux-x11-nightly
pkg update
pkg install tsu
pkg install pulseaudio
pkg install mc
pkg install curl

mkdir /data/local/tmp/chrootubuntu
cd /data/local/tmp/chrootubuntu

curl https://cdimage.ubuntu.com/ubuntu-base/releases/18.04.5/release/ubuntu-base-18.04.5-base-arm64.tar.gz --output ubuntu.tar.gz

tar xpvf ubuntu.tar.gz --numeric-owner

mkdir sdcard
mkdir dev/shm

cd ../
nano ./start.sh

Then copy in:
#!/bin/sh

# The path of Ubuntu rootfs
UBUNTUPATH="/data/local/tmp/chrootubuntu"

# Fix setuid issue
busybox mount -o remount,dev,suid /data
rosdep init
rosdep update
busybox mount --bind /dev $UBUNTUPATH/dev
busybox mount --bind /sys $UBUNTUPATH/sys
busybox mount --bind /proc $UBUNTUPATH/proc
busybox mount -t devpts devpts $UBUNTUPATH/dev/pts

# /dev/shm for Electron apps
busybox mount -t tmpfs -o size=256M tmpfs $UBUNTUPATH/dev/shm

# Mount sdcard
busybox mount --bind /sdcard $UBUNTUPATH/sdcard

# chroot into Ubuntu
busybox chroot $UBUNTUPATH /bin/su - root

Then press CTRL+S and CTRL+X and continue:

chmod +x ./start.sh
sh ./start.sh

Then in the ubuntu shell:

echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "127.0.0.1 localhost" > /etc/hosts

groupadd -g 3003 aid_inet
groupadd -g 3004 aid_net_raw
groupadd -g 1003 aid_graphics
usermod -g 3003 -G 3003,3004 -a _apt
usermod -G 3003 -a root

apt update
apt upgrade

apt install nano vim net-tools sudo git mc wget curl dialog apt-utils

ln -sf /usr/share/zoneinfo/[Region]/[City] /etc/localtime
#ln -sf /usr/share/zoneinfo/Pacific/Auckland /etc/localtime

apt install locales
locale-gen en_US.UTF-8
locale-gen en_GB.UTF-8
#locale-gen en_AU.UTF-8
#locale-gen en_NZ.UTF-8

apt install xubuntu-desktop

apt-get autopurge snapd

cat <<EOF | tee /etc/apt/preferences.d/nosnap.pref
# To prevent repository packages from triggering the installation of Snap,
# this file forbids snapd from being installed by APT.
# For more information: https://linuxmint-user-guide.readthedocs.io/en/latest/snap.html
Package: snapd
Pin: release a=*
Pin-Priority: -10
EOF

Install ROS:

http://wiki.ros.org/melodic/Installation/Ubuntu

sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'

curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -
apt update

apt install ros-melodic-desktop

echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
source ~/.bashrc

source /opt/ros/melodic/setup.bash

apt install python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential python-numpy python-pip

rosdep init
rosdep update

Casadi for python 2.7:

https://gmsanchez.github.io/2019/12/04/installing-casadi-on-a-raspberry-pi/

apt-get install gcc g++ gfortran git cmake libclang-dev llvm-dev libblas3 libblas-dev liblapack3 liblapack-dev ocl-icd-opencl-dev pkg-config --install-recommends

apt-get install swig ipython python-dev python-numpy python-scipy python-matplotlib --install-recommends

apt install coinor-libipopt-dev

git clone https://github.com/casadi/casadi.git -b 3.5.5
cd casadi
mkdir build
cd build

cmake -DPYTHON_EXECUTABLE=/usr/bin/python2 -DPYTHON_INCLUDE_DIR=/usr/include/python2.7 -DPYTHON_LIBRARY=/usr/lib/aarch64-linux-gnu/libpython2.7.so -DNUMPY_PATH=/usr/include/python2.7/numpy -DWITH_PYTHON=ON -DWITH_QPOASES=ON -DWITH_LAPACK=ON -DWITH_IPOPT=ON -DWITH_HSL=OFF -DWITH_CLANG=OFF -DWITH_OPENCL=ON -DWITH_DL=ON -DWITH_BLASFEO=OFF -DWITH_BUILD_BLASFEO=OFF -DWITH_MUMPS=ON ..

make -j4

make install

Setup vnc:

cd ~/
