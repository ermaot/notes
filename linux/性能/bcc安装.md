
```
# Pre-requisites:
# 1) Configure SUDO
# 2) Set up http_proxy and https_proxy for build user and for root (via .bashrc)
#    if there is no direct internet connection

# Install GIT
sudo yum install git

# Create build folder:
cd
mkdir build
cd build

# Detect number of threads
export THREADS=$(grep -c ^processor /proc/cpuinfo)

# Install kernel 4.9.71 from source
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.9.71.tar.xz
tar -xf linux-4.9.71.tar.xz
cd linux-4.9.71

# MANUAL copy .config 
# you can download recent kernel from Elrepo, take config from there and use "make olddefconfig"
# however Elrepo 4.14 kernel doesn't appear to work with BCC on CentOS 6
# kernel 4.9 works just fine

# build kernel

make -j $THREADS
sudo make modules_install
sudo make install
sudo dracut --force /boot/initramfs-4.9.71.img 4.9.71
cd ..

# Install GCC
sudo yum install gcc-c++

# Install cmake
wget https://cmake.org/files/v3.7/cmake-3.7.0.tar.gz
tar xf cmake-3.7.0.tar.gz
cd cmake-3.7.0
./configure
make -j $THREADS
sudo make install
cd ..

# Install Python 2.7
wget https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tar.xz
tar xf Python-2.7.12.tar.xz
cd Python-2.7.12
./configure
make -j $THREADS
sudo make altinstall
cd ..

# Install bison
curl -OL https://ftp.gnu.org/gnu/bison/bison-3.0.tar.xz
tar -xf bison-3.0.tar.xz
cd bison-3.0
./configure
make -j $THREADS
sudo make install
cd ..

# Install GCC 6
sudo yum install https://rpmfind.net/linux/centos/6.9/extras/x86_64/Packages/centos-release-scl-rh-2-3.el6.centos.noarch.rpm --nogpgcheck
sudo yum install devtoolset-6
export CC=/opt/rh/devtoolset-6/root/usr/bin/gcc
export CXX=/opt/rh/devtoolset-6/root/usr/bin/g++

# Build CLANG
curl -LO http://releases.llvm.org/3.9.1/cfe-3.9.1.src.tar.xz
curl -LO http://releases.llvm.org/3.9.1/llvm-3.9.1.src.tar.xz
tar -xf cfe-3.9.1.src.tar.xz
tar -xf llvm-3.9.1.src.tar.xz
mkdir clang-build
mkdir llvm-build

cd llvm-build
cmake -G "Unix Makefiles" -DLLVM_TARGETS_TO_BUILD="BPF;X86" \
  -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr ../llvm-3.9.1.src
make -j $THREADS
sudo make install

cd ../clang-build
cmake -G "Unix Makefiles" -DLLVM_TARGETS_TO_BUILD="BPF;X86" \
  -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr ../cfe-3.9.1.src
make -j $THREADS
sudo make install
cd ..

# Install packages needed for BCC build
sudo yum install -y elfutils-libelf-devel flex

# Download BCC
git clone https://github.com/iovisor/bcc.git

# MANUAL! Apply patch bcc_rhel6.patch
# Not needed if CentOS 6 support pull request is already merged

# Build BCC
export CFLAGS=-I${HOME}/build/linux-4.9.71/usr/include
mkdir bcc-build
cd bcc-build
cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=/usr ../bcc
make -j $THREADS
sudo make install

# Copy bcc package for python 2.7
sudo cp -r /usr/lib/python2.6/site-packages/bcc /usr/local/lib/python2.7/site-packages/bcc

# MANUAL! Mount DEBUGFS in fstab:
#debugfs			/sys/kernel/debug	debugfs	defaults	0 0
# or
mount -t debugfs debugfs /sys/kernel/debug

# You can start tools under root using command line like this:
#/usr/local/bin/python2.7 /usr/share/bcc/tools/offcputime <args>
```
