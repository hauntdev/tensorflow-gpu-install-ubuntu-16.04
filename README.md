# Tensorflow GPU install on ubuntu 16.04    

These instructions are intended to set up a deep learning environment for GPU-powered tensorflow.      
[See here for pytorch GPU install instructions](https://github.com/williamFalcon/pytorch-gpu-install)

After following these instructions you'll have:

1. Ubuntu 16.04. 
2. Cuda 9.0 drivers installed.
3. A conda environment with python 3.6.    
4. The latest tensorflow version with gpu support.   

---   
### Step 0: Noveau drivers     
Before you begin, you may need to disable the opensource ubuntu NVIDIA driver called [nouveau](https://nouveau.freedesktop.org/wiki/).

**Option 1: Modify modprobe file**
1. After you boot the linux system and are sitting at a login prompt, press ctrl+alt+F1 to get to a terminal screen.  Login via this terminal screen.

2. Create a file: /etc/modprobe.d/nouveau-blacklist.conf e.g. by 
```
sudo touch /etc/modprobe.d/nouveau-blacklist.conf
```
3.  Put the following in the above file...
```
blacklist nouveau
options nouveau modeset=0
```

4. Regenerate the kernel initramfs

```
sudo update-initramfs -u
```
5. reboot system   
```bash
reboot
```   
    
6. On reboot, verify that noveau drivers are not loaded   
```
lsmod | grep nouveau
```

If `nouveau` driver(s) are still loaded do not proceed with the installation guide and troubleshoot why it's still loaded.    

**Option 2: Modify Grub load command**    
From [this stackoverflow solution](https://askubuntu.com/questions/697389/blank-screen-ubuntu-15-04-update-with-nvidia-driver-nomodeset-does-not-work)    

1. When the GRUB boot menu appears : Highlight the Ubuntu menu entry and press the E key.
Add the nouveau.modeset=0 parameter to the end of the linux line ... Then press F10 to boot.   
2. When login page appears press [ctrl + ALt + F1]    
3. Enter username + password   
4. Uninstall every NVIDIA related software:   
```bash    
sudo apt-get purge nvidia*  
sudo reboot   
```   

---   
## Installation steps     


0. update apt-get   
``` bash 
sudo apt-get update
```
   
1. Install apt-get deps  
``` bash
sudo apt-get install openjdk-8-jdk git python-dev python3-dev python-numpy python3-numpy build-essential python-pip python3-pip python-virtualenv swig python-wheel libcurl3-dev curl   
```

2. install nvidia drivers 
``` bash
# The 16.04 installer works with 16.10.
# download drivers
curl -O http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_9.0.176-1_amd64.deb

# download key to allow installation
sudo apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub

# install actual package
sudo dpkg -i ./cuda-repo-ubuntu1604_9.0.176-1_amd64.deb


#  install cuda (but it'll prompt to install other deps, so we try to install twice with a dep update in between
sudo apt-get update
sudo apt-get install cuda-9-0   
```    
### Error if the OS task processing on 32bit:

1. If the system encounters a conflict, an error is displayed during the installation process as follow:

```
dpkg: error processing archive cuda-repo-ubuntu1604_9.0.176-1_amd64.deb (--install):
 package architecture (amd64) does not match system (i386)
Errors were encountered while processing:
 cuda-repo-ubuntu1604_9.0.176-1_amd64.deb
```
You can fix error the following: 

Code: `dpkg --print-architecture` will show the architecture dpkg is willing to install packages for 

If the architecture `amd64` is not listed, an amd package will be refused, even if all else seems fine. Now Can try to add it by

```
dpkg --add-architecture amd64
```

---

2. Error after finished installing Cuda 9.0:

```
The following packages have unmet dependencies:
 cuda-9-0:amd64 : Depends: cuda-toolkit-9-0:amd64 (>= 9.0.176) but it is not going to be installed
                  Depends: cuda-runtime-9-0:amd64 (>= 9.0.176) but it is not going to be installed
                  Depends: cuda-demo-suite-9-0:amd64 (>= 9.0.176) but it is not going to be installed
E: Unable to correct problems, you have held broken packages.
```

Solution: Reinstalling Ubuntu OS 16.04 (64 bit)

2a. reboot Ubuntu
```bash
sudo reboot
```    

2b. check nvidia driver install 
``` bash
nvidia-smi   

# you should see a list of gpus printed    
# if not, the previous steps failed.   
``` 

### Validation Error:
##### Code Error: `nvidia-smi: command not found`
##### Resolve: 
###### Method 1:
+ Download [Anaconda](https://www.anaconda.com/download/#linux)
+ `bash ./Anaconda3-5.1.0-Linux-x86_64.sh`
+ Add anaconda installation path to .bashrc (env variable): `export PATH="$PATH:/home/[username/anaconda3/bin]"`
+ Load in terminal: `source ~/.bashrc`
+ Run Anaconda desktop: `anaconda-navigator`
+ Create Tensorflow-GPU:
    ```
    conda create -n tensorflow python=3.6
    conda activate tensorflow
    pip install --ignore-installed --upgrade https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-1.4.0-cp36-cp36m-linux_x86_64.whl
    conda install tensorflow-gpu
    ```
+ Validation install of Tensorflow:
   
   
   ```py
   python
   >>> import tensorflow.compat.v1 as tf
   >>> tf.disable_v2_behavior
   WARNING:tensorflow:From /home/haunt/anaconda3/envs/tensorflow/lib/python3.6/site-packages/tensorflow_core/python/compat/v2_compat.py:65: disable_resource_variables (from tensorflow.python.ops.variable_scope) is deprecated and will be removed in a future version.
    Instructions for updating: 
    non-resource variables are not supported in the long term
   >>> hello  = tf.constant('Hello World!')
   >>> sess = tf.Session()
   >>> print(sess.run(hello))
   b'Hello World'
   ```
###### Method 2:

```
sudo apt-get purge nvidia-*
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update
sudo apt-cache search nvidia | grep -E “nvidia\-[0-9]{3}"
```

Output example:

```sh
nvidia-304-dev - NVIDIA binary Xorg driver development files
nvidia-331 - Transitional package for nvidia-331
nvidia-331-dev - Transitional package for nvidia-340-dev
nvidia-331-updates - Transitional package for nvidia-340
nvidia-331-updates-dev - Transitional package for nvidia-340-dev
nvidia-331-updates-uvm - Transitional package for nvidia-340
nvidia-331-uvm - Transitional package for nvidia-340
nvidia-340-dev - NVIDIA binary Xorg driver development files
nvidia-340-updates - Transitional package for nvidia-340
nvidia-340-updates-dev - Transitional package for nvidia-340-dev
nvidia-340-updates-uvm - Transitional package for nvidia-340-updates
nvidia-340-uvm - Transitional package for nvidia-340
nvidia-346 - Transitional package for nvidia-346
nvidia-346-dev - Transitional package for nvidia-352-dev
nvidia-346-updates - Transitional package for nvidia-346-updates
nvidia-346-updates-dev - Transitional package for nvidia-352-updates-dev
nvidia-352 - Transitional package for nvidia-361
nvidia-352-dev - Transitional package for nvidia-361-dev
nvidia-352-updates - Transitional package for nvidia-361
nvidia-352-updates-dev - Transitional package for nvidia-361-dev
nvidia-361-updates - Transitional package for nvidia-361
nvidia-361-updates-dev - Transitional package for nvidia-361-dev
nvidia-304-updates - Transitional package for nvidia-304
nvidia-304-updates-dev - Transitional package for nvidia-304-dev
nvidia-361 - Transitional package for nvidia-367
nvidia-361-dev - Transitional package for nvidia-367-dev
nvidia-367 - Transitional package for nvidia-375
nvidia-367-dev - Transitional package for nvidia-375-dev
nvidia-375 - Transitional package for nvidia-384
nvidia-375-dev - Transitional package for nvidia-384-dev
nvidia-384-dev - NVIDIA binary Xorg driver development files
nvidia-304 - NVIDIA legacy binary driver - version 304.137
nvidia-340 - NVIDIA binary driver - version 340.106
nvidia-384 - NVIDIA binary driver - version 384.130
nvidia-387-dev - Transitional package for nvidia-390-dev
nvidia-387 - Transitional package for nvidia-390
nvidia-390-dev - NVIDIA binary Xorg driver development files
nvidia-390 - NVIDIA binary driver - version 390.48
nvidia-396-dev - NVIDIA binary Xorg driver development files
nvidia-396 - NVIDIA binary driver - version 396.18
```

```
sudo apt-get install nvidia-390
sudo reboot
```

###### Reboot error after finished installing Nvidia driver version 390
###### Code Error: `dev-sda1: clean, xxxxx/xxxxxx files,xxxxx/xxxxx blocks`
###### Resolve: _Reinstall Ubuntu OS amd64_

~~3. Install cudnn~~

``` bash
wget https://s3.amazonaws.com/open-source-william-falcon/cudnn-9.0-linux-x64-v7.3.1.20.tgz
sudo tar -xzvf cudnn-9.0-linux-x64-v7.3.1.20.tgz
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```    

<del>4. Add these lines to end of ~/.bashrc:</del> 
``` bash
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64"
export CUDA_HOME=/usr/local/cuda
export PATH="$PATH:/usr/local/cuda/bin"
```   

~~4a. Reload bashrc~~     
``` bash 
source ~/.bashrc
```   

~~5. Install miniconda~~   
``` bash
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh   

# press s to skip terms   

# Do you approve the license terms? [yes|no]
# yes

# Miniconda3 will now be installed into this location:
# accept the location

# Do you wish the installer to prepend the Miniconda3 install location
# to PATH in your /home/ghost/.bashrc ? [yes|no]
# yes    

```   

~~5a. Reload bashrc~~     
``` bash 
source ~/.bashrc
```   

~~6. Create python 3.6 conda env to install tf~~   
``` bash
conda create -n tensorflow python=3.6

# press y a few times 
```   

~~7. Activate env~~  
``` bash
source activate tensorflow   
```

~~8. update pip (might already be up to date, but just in case...)~~
```
pip install --upgrade pip
```

~~9. Install stable tensorflow with GPU support for python 3.6~~    
``` bash
pip install --upgrade tensorflow-gpu

# If the above fails, try the part below
# pip install --ignore-installed --upgrade https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-1.2.0-cp36-cp36m-linux_x86_64.whl
```   

~~10. Test tf install~~  
``` bash
# start python shell   
python

# run test script   
import tensorflow as tf   

hello = tf.constant('Hello, TensorFlow!')

# when you run sess, you should see a bunch of lines with the word gpu in them (if install worked)
# otherwise, not running on gpu
sess = tf.Session()
print(sess.run(hello))
```  

or alternatively

```
tf.enable_eager_execution(); print(tf.reduce_sum(tf.random_normal([1000, 1000])))"
```
