# CUDA Multi-version Installation Guide

Author: Kxzuir |Rev. 04 | Last update: 2018/7/18

## System Requirements
* Operating System: Ubuntu 16.04, 64bit
* An CUDA-capable GPU
* Stable network connection
## Driver Pre-installation

It's recommended to perform the following command before everything started, to avoid further misconfiguration:

```bash
sudo apt-get update && sudo apt-get dist-upgrade
sudo apt-get autoremove && sudo apt-get autoclean
sudo reboot
```

## Install NVIDIA Driver

For detailed explanation, refer to the article [Install NVIDIA Driver and CUDA on Ubuntu / CentOS / Fedora Linux OS](https://gist.github.com/wangruohui/df039f0dc434d6486f5d4d098aa52d07#install-nvidia-graphics-driver-via-runfile). Brief steps can be summarized as the following steps.

1. Remove previous installation, in case you have installed driver via  `apt-get ` before.
   
   This step is optional for newly installed systems.
   
   ```bash
   # Note this might remove your cuda installation as well
   sudo apt-get purge nvidia*
   sudo apt-get autoremove
   ```

2. Download the Driver
   https://www.geforce.com/drivers
  
   Sometimes CUDA package may contain newer driver than the driver page. You may want to extract the driver from the CUDA package (option `--extract`. `--help` for full explanation) for the following steps.

3. Install dependencies

   ```bash
   sudo apt-get install build-essential gcc-multilib dkms g++
   sudo apt-get install freeglut3-dev libgl1-mesa-dev libx11-dev libxmu-dev libxi-dev libglu1-mesa libglu1-mesa-dev
   ```

4. Create blacklist for Nouveau driver

   Create a file at `/etc/modprobe.d/blacklist-nouveau.conf` with the following contents:

   ```
   blacklist nouveau
   options nouveau modeset=0
   ```
   
   (Optional, but **recommended**) Since we are about to performing last reboot before actual driver installation begins, it's recommended to set an environment variable for better compatibility. This can be done by appending the following line in `/etc/profile`:

   ```bash
   export IGNORE_CC_MISMATCH=1
   ```

   Then execute the following commands to apply settings, and reboot:
   
   ```bash
   sudo update-initramfs -u
   sudo reboot
   ```

5. Stop lightdm
   After the computer rebooted, press `Ctrl`+`Alt`+`F1` to enter tty1. Enter your credential to login, and execute the following command:
  
   ```bash
   sudo systemctl stop lightdm
   ```

6. Excuting the runfile to start installation process

   ```bash
   chmod +x NVIDIA-Linux-x86_64-390.42.run
   ./NVIDIA-Linux-x86_64-390.42.run --dkms
   ```
   
   **(Important)** Option `--no-opengl-files` must be added if non-NVIDIA (AMD or Intel) graphics are used for display while NVIDIA graphics are used for computation.

7. Reboot the system

   ```bash
   sudo reboot
   ```

## CUDA Pre-installation

Adding gcc repository:

```bash
sudo add-apt-repository ppa:ubuntu-toolchain-r/test; sudo apt-get update
```

## Install CUDA 9.2
1. Get latest packages
   
   https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=runfilelocal

2. Install packages
   ```bash
   # Replacing xxx to actual filenames
   sudo ./cuda_9.2.x_xxx.xx_linux.run --toolkit --toolkitpath=/opt/cuda-9.2 --samples --samplespath=/opt/cuda-9.2-samples --silent --override
   sudo ./cuda_9.2.x_xxx.xx_linux.run --installdir=/opt/cuda-9.2 --accept-eula --silent
   sudo ./cuda_9.2.x_xxx.xx_linux.run --installdir=/opt/cuda-9.2 --accept-eula --silent
   sudo ./cuda_9.2.x_xxx.xx_linux.run --installdir=/opt/cuda-9.2 --accept-eula --silent
   ```

3. Configure gcc

   ```bash
   sudo apt-get install gcc-7 g++-7
   sudo ln -s /usr/bin/gcc-7 /opt/cuda-9.2/bin/gcc
   sudo ln -s /usr/bin/g++-7 /opt/cuda-9.2/bin/g++
   ```

4. Configure environment variables

   ```bash
   printf "alias ldcuda9.2='export LD_LIBRARY_PATH=/opt/cuda-9.2/lib64\${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}; export PATH=/opt/cuda-9.2/bin\${PATH:+:\${PATH}}'" >> ~/.bashrc
   ```
5. Build samples

   ```bash
   # Restart the terminal
   ldcuda9.2
   cuda-install-samples-9.2.sh ~
   mv ~/NVIDIA_CUDA-9.2_Samples ~/cuda-9.2-samples
   cd ~/cuda-9.2-samples
   make
   ```
   
## Install CUDA 4.2
1. Get archived packages

   https://developer.nvidia.com/cuda-toolkit-42-archive

2. Inatall packages

   ```bash
   sudo ./cudatoolkit_4.2.9_linux_64_ubuntu11.04.run
   # Install path: /opt
   # Ask if remove existing version: no
   sudo mv /opt/cuda /opt/cuda-4.2
   sudo ./gpucomputingsdk_4.2.9_linux.run
   # Install path: /opt/cuda-4.2-samples
   # CUDA insatll path: /opt/cuda-4.2
   ```

3. Configure gcc

   ```bash
   sudo apt-get install gcc-4.6 g++-4.6
   sudo ln -s /usr/bin/gcc-4.6 /opt/cuda-4.2/bin/gcc
   sudo ln -s /usr/bin/g++-4.6 /opt/cuda-4.2/bin/g++
   ```

4. Configure environment variables

   ```bash
   printf "alias ldcuda4.2='export LD_LIBRARY_PATH=/opt/cuda-4.2/lib64:/opt/cuda-4.2/lib\${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}; export PATH=/opt/cuda-4.2/bin\${PATH:+:\${PATH}}; export CUDA_INSTALL_PATH=/opt/cuda-4.2'" >> ~/.bashrc
   ```

5. Repair samples

   ```bash
   cp -R /opt/cuda-4.2-samples ~
   cd ~/cuda-4.2-samples
   ```
   
   Before building samples, fix these errors which may cause linkage error:

   In `./C/common/common.mk` and `./CUDALibraries/common/common_cudalib.mk`, lines like:
   
   ```makefile
   LIB += ... ${OPENGLLIB} .... $(RENDERCHECKGLLIB) ...
   ```
   
   should have the two in reverse. The line numbers in the two files are {271, 275, 282}, {292, 296, 299}. 

   In `./CUDALibraries/src`, modify Makefile in the following places:

   `boxFilterNPP/Makefile:41`

   ```makefile
   $(CXX) $(INC) -o boxFilterNPP boxFilterNPP.cpp $(LIB) -lUtilNPP_$(LIB_ARCH) -lfreeimage$(FREEIMAGELIBARCH)
   ```

   `imageSegmentationNPP/Makefile:41`

   ```makefile
   $(CXX) $(INC) -o imageSegmentationNPP imageSegmentationNPP.cpp $(LIB) -lUtilNPP_$(LIB_ARCH) -lfreeimage$(FREEIMAGELIBARCH)
   ```

   `freeImageInteropNPP/Makefile:41`

   ```makefile
   $(CXX) $(INC) -o freeImageInteropNPP freeImageInteropNPP.cpp $(LIB) -lUtilNPP_$(LIB_ARCH) -lfreeimage$(FREEIMAGELIBARCH)
   ```

   `histEqualizationNPP/Makefile:41`

   ```makefile
   $(CXX) $(INC) -o histEqualizationNPP histEqualizationNPP.cpp $(LIB) -lUtilNPP_$(LIB_ARCH) -lfreeimage$(FREEIMAGELIBARCH)
   ```

   `randomFog/Makefile`, Add

   ```makefile
   USERENDERCHECKGL := 1
   ```

   Modify`./CUDALibraries/common_cudalib.mk:256` to

   ```makefile
   RENDERCHECKGLLIB := -L../../../C/lib -lrendercheckgl_$(LIB_ARCH)$(LIBSUFFIX)
   ```

   Finally, create OpenCL link:

   ```bash
   sudo apt-get install mesa-utils
   sudo ln -s /usr/lib/x86_64-linux-gnu/libOpenCL.so.1 /usr/lib/libOpenCL.so
   ```

6. Build samples

   ```bash
   # Restart the terminal
   ldcuda4.2
   cd ~/cuda-4.2-samples
   make
   ```
   
## Install CUDA 5.5

1. Get archived packages

   https://developer.nvidia.com/cuda-toolkit-55-archive

2. Install packages

   ```bash
   sudo ./cuda_5.5.22_linux_64.run -toolkit -toolkitpath=/opt/cuda-5.5 -samples -samplespath=/opt/cuda-5.5-samples -silent -override
   ```

3. Configure gcc

   ```bash
   sudo apt-get install gcc-4.8 g++-4.8
   sudo ln -s /usr/bin/gcc-4.8 /opt/cuda-5.5/bin/gcc
   sudo ln -s /usr/bin/g++-4.8 /opt/cuda-5.5/bin/g++
   ```

4. Configure environment variables

   ```bash
   printf "alias ldcuda5.5='export LD_LIBRARY_PATH=/opt/cuda-5.5/lib64\${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}; export PATH=/opt/cuda-5.5/bin\${PATH:+:\${PATH}}'" >> ~/.bashrc
   ```

5. Build samples

   ```bash
   # Restart the terminal
   ldcuda5.5
   /opt/cuda-5.5/bin/cuda-install-samples-5.5.sh  ~
   mv ~/NVIDIA_CUDA-5.5_Samples ~/cuda-5.5-samples
   cd ~/cuda-5.5-samples
   make
   ```

## Install CUDA 6.5

1. Get archived packages

   https://developer.nvidia.com/cuda-toolkit-65

2. Install packages

   ```bash
   sudo ./cuda_6.5.19_linux_64.run -toolkit -toolkitpath=/opt/cuda-6.5 -samples -samplespath=/opt/cuda-6.5-samples -silent -override
   ```

3. Configure gcc

   ```bash
   sudo apt-get install gcc-4.8 g++-4.8
   sudo ln -s /usr/bin/gcc-4.8 /opt/cuda-6.5/bin/gcc
   sudo ln -s /usr/bin/g++-4.8 /opt/cuda-6.5/bin/g++
   ```

4. Configure environment variables

   ```bash
   printf "alias ldcuda6.5='export LD_LIBRARY_PATH=/opt/cuda-6.5/lib64\${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}; export PATH=/opt/cuda-6.5/bin\${PATH:+:\${PATH}}'" >> ~/.bashrc
   ```

5. Repair samples

   ```bash
   /opt/cuda-6.5/bin/cuda-install-samples-6.5.sh  ~
   mv ~/NVIDIA_CUDA-6.5_Samples ~/cuda-6.5-samples
   cd ~/cuda-6.5-samples
   ```
   
   Before build samples, modify `./common/findglib.mk:57` to

   ```makefile
   UBUNTU_PKG_NAME = "nvidia-390"
   ```

   Note `390` is the currently installed driver version. Change it in case you have any newer version. 

   Create the following symbolic links:

   ```bash
   sudo ln -s /usr/lib/nvidia-390/libnvcuvid.so /opt/cuda-6.5/lib64/libnvcuvid.so
   sudo ln -s /usr/lib/nvidia-390/libnvcuvid.so.1 /opt/cuda-6.5/lib64/libnvcuvid.so.1
   ```

   And modify `./6_Advanced/shfl_scan/Makefile:84` to

   ```makefile
   NVCCFLAGS   := -D_FORCE_INLINES -m${OS_SIZE} ${ARCH_FLAGS}
   ```

6. Build samples

   ```bash
   # Restart the terminal
   ldcuda6.5
   cd ~/cuda-6.5-samples
   make
   ```
