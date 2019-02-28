# CUDA Multi-version Installation Guide

Author: Kxzuir | Rev. 10 | Last update: 2019/02/28

## System Requirements
* Operating System: Ubuntu 16.04, 64bit
* A CUDA-capable GPU (https://developer.nvidia.com/cuda-gpus)
* Stable network connection
## Driver Pre-installation

It's recommended to perform the following command before everything started, to avoid further misconfiguration:

```bash
sudo apt-get update && sudo apt-get dist-upgrade
sudo apt-get autoremove && sudo apt-get autoclean
sudo reboot
```

## Install NVIDIA Driver

For detailed explanation, refer to the article [Install NVIDIA Driver and CUDA on Ubuntu / CentOS / Fedora Linux OS](https://gist.github.com/wangruohui/df039f0dc434d6486f5d4d098aa52d07#install-nvidia-graphics-driver-via-runfile). We just extract what we need here.

1. Remove previous installation, in case you have installed driver via `apt-get ` before.
   
   This step is not necessary for newly installed systems.
   
   ```bash
   # Note this might remove your cuda installation as well
   sudo apt-get purge nvidia*
   sudo apt-get autoremove
   ```

2. Download the driver

   https://www.nvidia.com/object/unix.html
  
   Sometimes CUDA package may carry newer driver than the driver page. You may want to extract the driver from CUDA package (option `--extract`. `--help` for full explanation) for the following steps.
   
   At the time this tutorial was updated, the newest driver version is `418.53` and the driver package is `NVIDIA-Linux-x86_64-418.43.run`. If a newer driver releases, the package you see may vary.

3. Install dependencies

   ```bash
   sudo apt-get install build-essential gcc-multilib dkms gcc g++
   sudo apt-get install freeglut3-dev libgl1-mesa-dev libx11-dev libxmu-dev libxi-dev libglu1-mesa libglu1-mesa-dev
   ```
   
   **(Important)** Since we are about to add a new repository carrying various `gcc`/`g++` versions, it's necessary to pin `gcc-5` and `g++-5` to version `5.4.0-6ubuntu1~16.04.10`, or a newer version `5.5.0-12ubuntu1~16.04` will be installed. This version is higher than the complier version used to build the kernel, and could prevent driver installation when `--dkms` option is on. So we need to install `gcc-5` and `g++-5` first, and pin their version to what we need.
   
   ```bash
   sudo apt-get install gcc-5 g++-5
   sudo apt-mark hold gcc-5 g++-5
   ```
   
4. Create blacklist for Nouveau driver

   Create a file at `/etc/modprobe.d/blacklist-nouveau.conf` with the following contents:

   ```
   blacklist nouveau
   options nouveau modeset=0
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

6. Execute the runfile to start the installation process

   ```bash
   chmod +x NVIDIA-Linux-x86_64-418.43.run
   sudo ./NVIDIA-Linux-x86_64-418.43.run --dkms
   ```
   
   **(Important)** Option `--no-opengl-files` must be added (i.e., `sudo ./NVIDIA-Linux-x86_64-418.43.run --dkms --no-opengl-files`)if non-NVIDIA (AMD or Intel) graphics are used for display while NVIDIA graphics are used for computation.

7. Restart lightdm, or simply reboot to make everything retuned

   ```bash
   sudo systemctl start lightdm
   ```
   
   or
   
   ```bash
   sudo reboot
   ```

8. Update the driver

   If a new driver version release later, re-perform step `2`, `5`, `6` and `7`. Previous old driver will be automatically uninstalled in step `6`, with a prompt for confirmation.
   
## Version Check

1. Version for gcc

   The `gcc` compiler is required for development using the CUDA Toolkit. Moreove, each CUDA version has its own `gcc` version requirements. The constraint is implemented by putting version check code in `${CUDA_INSTALL_PATH}/include/crt/host_config.h` or `${CUDA_INSTALL_PATH}/include/host_config.h`. You may modify the file to adapt your needs, but in some cases it would cause unexpected behavior.

   This tables shows the officially announced and maximum version of `gcc` each CUDA version supported. In this tutorial, we prefer to use the highest version of `gcc` CUDA allowed, shown as "Recommended" column.

   | CUDA Version | Announced | Maximum | Recommended     |
   |--------------|-----------|---------|-----------------|
   | 4.2          | 4.5.2     | 4.6.x   | 4.6.4 (gcc-4.6) |
   | 5.5          | 4.7.2     | 4.8.x   | 4.8.5 (gcc-4.8) |
   | 6.5          | 4.8.2     | 4.8.x   | 4.8.5 (gcc-4.8) |
   | 7.5          | 4.9.2     | 4.9.x   | 4.9.4 (gcc-4.9) |
   | 8.0 GA2      | 5.3.1     | -       | -               |
   | 9.0          | 5.3.1     | 6.x     | 6.5.0 (gcc-6)   |
   | 9.2          | 5.4.0     | 7.x     | 7.4.0 (gcc-7)   |
   | 10.0         | 5.4.0     | 7.x     | 7.4.0 (gcc-7)   |
   | 10.1         | 5.4.0     | 8.x     | 8.1.0 (gcc-8)   |

   To have access to each `gcc` version, we need to add a repository:

   ```bash
   sudo add-apt-repository ppa:ubuntu-toolchain-r/test; sudo apt-get update
   ```

2. Version for CUDA

   You do not need to install all CUDA versions listed below, just what you need. Here we demonstrate the installation process of CUDA-10.1, CUDA-9.2, CUDA-4.2, CUDA-5.5 and CUDA-6.5.
   
   Installation order can be arbitary.

   CUDA Toolkit Archive: https://developer.nvidia.com/cuda-toolkit-archive. This link holds all CUDA versions NVIDIA had ever released.

## Install CUDA 10.1
1. Get latest package
   
   https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=runfilelocal
   
   Note this link always leads to the newest CUDA version. If a newer CUDA version releases, the content you see may vary.
   
   Or just simply perform:
   
   ```bash
   cd ~
   wget https://developer.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.105_418.39_linux.run
   ```
   
   This will directly download the package into your home dictionary.
   
2. Install package

   ```bash
   cd ~
   chmod +x cuda_10.1.105_418.39_linux.run
   sudo ./cuda_10.1.105_418.39_linux.run --toolkit --toolkitpath=/opt/cuda-10.1 --silent --override
   ```

3. Configure gcc

   Note the gcc/g++ version must satisfy the constraint listed above. In most cases, we can just pick up the highest version for most optimization. Here in CUDA-10.1, gcc-8/g++-8 is the best choice.

   ```bash
   sudo apt-get install gcc-8 g++-8
   sudo ln -sf /usr/bin/gcc-8 /opt/cuda-10.1/bin/gcc
   sudo ln -sf /usr/bin/g++-8 /opt/cuda-10.1/bin/g++
   ```

4. Configure environment variables

   ```bash
   printf "\nalias ldcuda10.1='export LD_LIBRARY_PATH=/opt/cuda-10.1/lib64\${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}; export PATH=/opt/cuda-10.1/bin\${PATH:+:\${PATH}}'\n" >> ~/.bashrc
   ```
5. Build samples

   ```bash
   cd ~
   . ~/.bashrc
   ldcuda10.1
   
   # Verify CUDA version. You should see "release 10.1" in the output.
   nvcc -V
   
   # Copy CUDA samples to your home dictionary and build there
   cuda-install-samples-10.1.sh ~
   mv ~/NVIDIA_CUDA-10.1_Samples ~/cuda-10.1-samples
   cd ~/cuda-10.0-samples
   make
   
   # You should see "Finished building CUDA samples" in the last.
   ```

## Install CUDA 9.2

The process is similar to the installation of CUDA 10.0. We just put the bash script.
Release page: https://developer.nvidia.com/cuda-92-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=runfilelocal

```bash
cd ~
wget https://developer.nvidia.com/compute/cuda/9.2/Prod2/local_installers/cuda_9.2.148_396.37_linux
wget https://developer.nvidia.com/compute/cuda/9.2/Prod2/patches/1/cuda_9.2.148.1_linux
sudo ./cuda_9.2.148_396.37_linux.run --toolkit --toolkitpath=/opt/cuda-9.2 --silent --override
sudo ./cuda_9.2.148.1_linux.run --installdir=/opt/cuda-9.2 --silent --accept-eula

sudo apt-get install gcc-7 g++-7
sudo ln -sf /usr/bin/gcc-7 /opt/cuda-9.2/bin/gcc
sudo ln -sf /usr/bin/g++-7 /opt/cuda-9.2/bin/g++

printf "\nalias ldcuda9.2='export LD_LIBRARY_PATH=/opt/cuda-9.2/lib64\${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}; export PATH=/opt/cuda-9.2/bin\${PATH:+:\${PATH}}'\n" >> ~/.bashrc

. ~/.bashrc
ldcuda9.2
cuda-install-samples-9.2.sh ~
mv ~/NVIDIA_CUDA-9.2_Samples ~/cuda-9.2-samples
cd ~/cuda-9.2-samples
make
```

## Install CUDA 4.2
1. Get archived packages

   https://developer.nvidia.com/cuda-toolkit-42-archive

2. Install packages

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
   sudo ln -sf /usr/bin/gcc-4.6 /opt/cuda-4.2/bin/gcc
   sudo ln -sf /usr/bin/g++-4.6 /opt/cuda-4.2/bin/g++
   ```

4. Configure environment variables

   ```bash
   printf "\nalias ldcuda4.2='export LD_LIBRARY_PATH=/opt/cuda-4.2/lib64:/opt/cuda-4.2/lib\${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}; export PATH=/opt/cuda-4.2/bin\${PATH:+:\${PATH}}; export CUDA_INSTALL_PATH=/opt/cuda-4.2'\n" >> ~/.bashrc
   ```

5. Repair samples

   ```bash
   cp -R /opt/cuda-4.2-samples ~
   cd ~/cuda-4.2-samples
   ```
   
   Before building samples, fix these errors which may cause linking error:

   In `./C/common/common.mk` and `./CUDALibraries/common/common_cudalib.mk`, lines like:
   
   ```makefile
   LIB += ... ${OPENGLLIB} .... $(RENDERCHECKGLLIB) ...
   ```
   
   should have the two in reverse. The line numbers in the two files are {271, 275, 282}, {292, 296, 299}. 

   In `./CUDALibraries/src`, modify Makefile in the following places:

   `boxFilterNPP/Makefile:41` to

   ```makefile
   $(CXX) $(INC) -o boxFilterNPP boxFilterNPP.cpp $(LIB) -lUtilNPP_$(LIB_ARCH) -lfreeimage$(FREEIMAGELIBARCH)
   ```

   `imageSegmentationNPP/Makefile:41` to

   ```makefile
   $(CXX) $(INC) -o imageSegmentationNPP imageSegmentationNPP.cpp $(LIB) -lUtilNPP_$(LIB_ARCH) -lfreeimage$(FREEIMAGELIBARCH)
   ```

   `freeImageInteropNPP/Makefile:41` to

   ```makefile
   $(CXX) $(INC) -o freeImageInteropNPP freeImageInteropNPP.cpp $(LIB) -lUtilNPP_$(LIB_ARCH) -lfreeimage$(FREEIMAGELIBARCH)
   ```

   `histEqualizationNPP/Makefile:41` to

   ```makefile
   $(CXX) $(INC) -o histEqualizationNPP histEqualizationNPP.cpp $(LIB) -lUtilNPP_$(LIB_ARCH) -lfreeimage$(FREEIMAGELIBARCH)
   ```

   `randomFog/Makefile`, add

   ```makefile
   USERENDERCHECKGL := 1
   ```

   Modify `./CUDALibraries/common_cudalib.mk:256` to

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
   . ~/.bashrc
   ldcuda4.2
   cd ~/cuda-4.2-samples
   make
   ```
   
## Install CUDA 5.5

1. Get archived package

   https://developer.nvidia.com/cuda-toolkit-55-archive

2. Install packages

   ```bash
   sudo ./cuda_5.5.22_linux_64.run -toolkit -toolkitpath=/opt/cuda-5.5 -silent -override
   ```

3. Configure gcc

   ```bash
   sudo apt-get install gcc-4.8 g++-4.8
   sudo ln -sf /usr/bin/gcc-4.8 /opt/cuda-5.5/bin/gcc
   sudo ln -sf /usr/bin/g++-4.8 /opt/cuda-5.5/bin/g++
   ```

4. Configure environment variables

   ```bash
   printf "\nalias ldcuda5.5='export LD_LIBRARY_PATH=/opt/cuda-5.5/lib64\${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}; export PATH=/opt/cuda-5.5/bin\${PATH:+:\${PATH}}'\n" >> ~/.bashrc
   ```

5. Build samples

   ```bash
   . ~/.bashrc
   ldcuda5.5
   cuda-install-samples-5.5.sh ~
   mv ~/NVIDIA_CUDA-5.5_Samples ~/cuda-5.5-samples
   cd ~/cuda-5.5-samples
   make
   ```

## Install CUDA 6.5

1. Get archived package

   https://developer.nvidia.com/cuda-toolkit-65

2. Install packages

   ```bash
   sudo ./cuda_6.5.19_linux_64.run -toolkit -toolkitpath=/opt/cuda-6.5 -silent -override
   ```

3. Configure gcc

   ```bash
   sudo apt-get install gcc-4.8 g++-4.8
   sudo ln -sf /usr/bin/gcc-4.8 /opt/cuda-6.5/bin/gcc
   sudo ln -sf /usr/bin/g++-4.8 /opt/cuda-6.5/bin/g++
   ```

4. Configure environment variables

   ```bash
   printf "\nalias ldcuda6.5='export LD_LIBRARY_PATH=/opt/cuda-6.5/lib64\${LD_LIBRARY_PATH:+:\${LD_LIBRARY_PATH}}; export PATH=/opt/cuda-6.5/bin\${PATH:+:\${PATH}}'\n" >> ~/.bashrc
   ```

5. Repair samples

   ```bash
   /opt/cuda-6.5/bin/cuda-install-samples-6.5.sh ~
   mv ~/NVIDIA_CUDA-6.5_Samples ~/cuda-6.5-samples
   cd ~/cuda-6.5-samples
   ```
   
   Before build samples, modify `./common/findglib.mk:57` to

   ```makefile
   UBUNTU_PKG_NAME = "nvidia-418"
   ```

   Note `418` is the currently installed driver version. Change it in case you have any newer version. 

   Create the following symbolic links:

   ```bash
   sudo ln -s /usr/lib/nvidia-418/libnvcuvid.so /opt/cuda-6.5/lib64/libnvcuvid.so
   sudo ln -s /usr/lib/nvidia-418/libnvcuvid.so.1 /opt/cuda-6.5/lib64/libnvcuvid.so.1
   ```

   And modify `./6_Advanced/shfl_scan/Makefile:84` to

   ```makefile
   NVCCFLAGS   := -D_FORCE_INLINES -m${OS_SIZE} ${ARCH_FLAGS}
   ```

6. Build samples

   ```bash
   . ~/.bashrc
   ldcuda6.5
   cd ~/cuda-6.5-samples
   make
   ```
   
## Switch Between CUDA Versions

You may notice a new command like `ldcudaX.X`(`ldcuda4.2`, `ldcuda6.5`, `ldcuda9.2`, `ldcuda10.0`,etc.), and that's the switch for different CUDA versions. By default, the system won't load any CUDA version; You need to manually enable specific CUDA version by enter corresponding command, just like each "Build samples" does. Your command will only affect the session it stays, so you can open as many terminals as you want, with each under its own CUDA environment, including `nvcc`, `gcc`, `g++` and related libs.

Of course, if you would like to set a default version, just append a `ldcudaX.X` command in `~/.bashrc`. Later entered command will override default settings.

## Remove CUDA Versions

Since CUDA installation just involves copying files to desired directories, uninstall CUDA is quite easy. For example, if you want to uninstall CUDA-10.1, just remove the dictionary `/opt/cuda-10.1` (also, `~/cuda-10.1-samples`, where you tested your complier; and for CUDA-4.2, also `/opt/cuda-4.2-samples`). Other versions won't be affected.
