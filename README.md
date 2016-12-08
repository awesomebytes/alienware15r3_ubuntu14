# How to install Ubuntu 14.04.05 on Alienware 15 R3

Tiny guide to install Ubuntu 14.04.05 on a brand new Alienware 15 R3.

## Let windows 10 install
Just next, next, next filling up your data.

You should get a BIOS update alert from the Alienware Update widget. If not,
right click on the Down arrow icon in the bottom right extra icons `^` thing and 
right click, then click `Check for Updates`.

Install it. It will reboot your computer, try to not touch anything up until
you are back to Windows.

## Shrink the disk
Go to the disk manager (right click on Windows icon > Disk management) and
Shrink the OS partition (right click on it, `Shrink Volume...`). It offered
me shrinking by 115XXX MB. Just shrink it 110000 MB. Shrinking it more won't work.

## Get Windows to boot on AHCI mode (SATA options)
For Ubuntu to see the NVME disk it needs to boot on AHCI not on RAID mode (the default).
As you don't want to go to the BIOS to change the SATA mode every time you want to boot
in one or another OS, we need to force Windows to be able to boot in AHCI mode.

For that follow the instructions (kindly taken from [here](http://www.tenforums.com/drivers-hardware/15006-attn-ssd-owners-enabling-ahci-mode-after-windows-10-installation.html)):

1. Run Command Prompt as Admin.
2. Invoke a Safe Mode boot with the command: `bcdedit /set {current} safeboot minimal`.
3. Restart the PC and enter your BIOS during bootup (F2 key).
4. In tab `Advanced` change option `SATA Operation` from `RAID on` to `AHCI` mode then go to `Exit` tab and use `Save Changes and Reset`.
5. Windows 10 will launch in Safe Mode.
6. Right click the Window icon and select to run the Command Prompt in Admin mode from among the various options.
7. Cancel Safe Mode booting with the command: `bcdedit /deletevalue {current} safeboot`.
8. Restart your PC once more and this time it will boot up normally but with AHCI mode activated.
9. Enjoy your awesomeness.


## Disable Secure boot
You need to disable secure boot in order to boot any other OS.

Enter your BIOS (F2 key on boot). Go to `Boot` tab and change `Secure Boot` option to `Disabled`.

Note that `Boot List Option` should be `UEFI` (it's the default).

## Get the latest Ubuntu 14.04 and write it to a bootable pendrive
I got my image from the [official Ubuntu releases link](http://releases.ubuntu.com/14.04/) scrolling down to find [ubuntu-14.04.5-desktop-amd64.iso](http://releases.ubuntu.com/14.04/ubuntu-14.04.5-desktop-amd64.iso). I used Firefox + DownThemAll! addon to download it faster.


I use [UNetbootin](https://unetbootin.github.io/) for writing my images.

I use [Gparted](http://gparted.org/download.php) to format my pendrive to FAT32.

## Add to the bootable pendrive the latest Gparted version (0.24)
Download from [ubuntuupdates.org gparted](http://www.ubuntuupdates.org/package/getdeb_apps/trusty/apps/getdeb/gparted). Scroll down to `Download "gparted"` and click on 64-bit deb package. The direct link is [here](http://archive.getdeb.net/ubuntu/pool/apps/g/gparted/gparted_0.24.0-1~getdeb1_amd64.deb) which may or not work.

Copy `gparted_0.24.0-1~getdeb1_amd64.deb` to the pendrive root into a new folder, I created one called
`gparted_deb`. You'll need it.


## Boot from the live linux pendrive
Press F12 while booting and choose under `UEFI OPTIONS` to boot from your pendrive, for me it was
`USB1 - UEFI OS( USB DISK 3.0 PMAP)`.


## Install Gparted 0.24
In order for the installation wizard to be able to deal with your NVME disk (the SSD) you need the
newest Gparted.

To install it, open a terminal (Control+Alt+T) and:
```
cd /cdrom
cd gparted_deb # Or whatever you called the folder
sudo dpkg -i gparted_0.24.0-1~getdeb1_amd64.deb
```

## Use the install Wizard
Just double click the `Install Ubuntu 14.04.05 LTS` desktop icon.

Configure as you like BUT DON'T ENABLE DOWNLOAD UPDATES WHILE INSTALLING NOR INSTALL THIRD PARTY SOFTWARE. It will freeze your installation. If you don't believe me, just try and enjoy your reboot.


Click on `Something else`.

Now you should see some partitions like `/dev/nvme0n1`. If you don't, you missed some step.

Now choose the free space partition that corresponds to the shrinked space we made before. For me it's
115343 MB. I'll just make a partition for `/` and another `swap` one.

In order to be able to hibernate in Ubuntu you'll need at least your amount of RAM as swap. I doubt very much it will actually work, but hey, you need to try.

I have 16GB of RAM so I'll do 115343 - 17 * 1024 = 97884 MB partition. (Yeah that's a 17, I'm a bit
lazy to check for how much exactly it should be).

Click on that `free space` to be selected and click on the `+` symbol. Put your amount
of MB for it (97884) in Size. Choose `Logical` as Type. Leave Location as `Beginning of this space`. Use as `Ext4`. Mount point as `/`.

Then on the left free space, repeat the process but make it of type `swap`.

**IMPORTANT** now you need to change the `Device for boot loader installation` to `/dev/nvme0n1`.

Now you can click `Install Now`.

In a few minutes you should be good to go!

## Install Wifi and Ethernet
For the Wifi card QCA6174 you need a newer binary of the firmware (based on [this askubuntu post](http://askubuntu.com/questions/607707/ath10k-installation)).

Just copy the folder of this repo [QCA6174](QCA6174) to `/lib/firmware/ath10k`. Note that you'll be
overwriting what is already there.

You can download it either doing:

```
git clone https://github.com/awesomebytes/alienware15r3_ubuntu14
```

Or clicking in `Clone or download` > `Download ZIP`.

Once done, reboot. Wifi and Ethernet will be working after.

## Getting Nvidia and CUDA to work
I found no way of using the Nvidia 100% of the time, but using the Intel HD 530 card
for normal use and executing programs with `optirun` worked.

In all my other approachs when trying to use CUDA demos I'd get (I hope this helps someone
googling):

```
 CUDA Device Query (Runtime API) version (CUDART static linking)

cudaGetDeviceCount returned 35
-> CUDA driver version is insufficient for CUDA runtime version
Result = FAIL
```

Instructions next:

## Installing CUDA (and nvidia driver) and forcing usage of Intel card
Download from Nvidia [Cuda 8.0](https://developer.nvidia.com/cuda-downloads):

```bash
wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_8.0.44-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1604_8.0.44-1_amd64.deb
sudo apt-get update
sudo apt-get install cuda
```

**Before rebooting do** (if you reboot without doing this you'll break your X):
```bash
sudo prime-select intel
```

## Installing bumblebee
```bash
sudo apt-get install bumblebee bumblebee-nvidia primus linux-headers-generic
```

**Now reboot**.

## Testing installation
Put a copy of the CUDA demos in your home:
```
/usr/local/cuda-8.0/bin/cuda-install-samples-8.0.sh ~
```

Then you can compile a couple of demos:

```bash
cd ~/NVIDIA_CUDA-8.0_Samples/1_Utilities/deviceQuery
make
optirun ./deviceQuery
```

I got as output:
```
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce GTX 1070"
  CUDA Driver Version / Runtime Version          8.0 / 8.0
  CUDA Capability Major/Minor version number:    6.1
  Total amount of global memory:                 8113 MBytes (8507555840 bytes)
  (16) Multiprocessors, (128) CUDA Cores/MP:     2048 CUDA Cores
  GPU Max Clock rate:                            1645 MHz (1.64 GHz)
  Memory Clock rate:                             4004 Mhz
  Memory Bus Width:                              256-bit
  L2 Cache Size:                                 2097152 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)
  Maximum Layered 1D Texture Size, (num) layers  1D=(32768), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(32768, 32768), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 2 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 8.0, CUDA Runtime Version = 8.0, NumDevs = 1, Device0 = GeForce GTX 1070
Result = PASS
```

Another demo:

```bash
cd ~/NVIDIA_CUDA-8.0_Samples/1_Utilities/bandwidthTest
make
optirun ./bandwidthTest
```

For other demos you may need to install:
```
sudo apt-get install freeglut3-dev mesa-common-dev
```


Furthermore, check usage of the Nvidia 1070 GTX with `glxheads`:

```
sudo apt-get install mesa-utils
```

```
sam@alien:~$ glxheads 
glxheads: exercise multiple GLX connections (any key = exit)
Usage:
  glxheads xdisplayname ...
Example:
  glxheads :0 mars:0 venus:1
Name: :0
  Display:     0xe81010
  Window:      0x5000002
  Context:     0xe90200
  GL_VERSION:  3.0 Mesa 11.2.0
  GL_VENDOR:   Intel Open Source Technology Center
  GL_RENDERER: Mesa DRI Intel(R) HD Graphics 530 (Skylake GT2)
```

And:
```
sam@alien:~$ optirun glxheads
glxheads: exercise multiple GLX connections (any key = exit)
Usage:
  glxheads xdisplayname ...
Example:
  glxheads :0 mars:0 venus:1
Name: :0
  Display:     0x1f324e0
  Window:      0x5200002
  Context:     0x2276610
  GL_VERSION:  4.5.0 NVIDIA 367.57
  GL_VENDOR:   NVIDIA Corporation
  GL_RENDERER: GeForce GTX 1070/PCIe/SSE2
```


## Further notes
Checking `dmesg` it complained about not having `nss-myhostname`.

    sudo apt-get install libnss-myhostname

Solves it.

There is a tool to change the colors of the LEDs for Alienware 17 that should be easy to make it work on this laptop:
https://github.com/acyed/qtFx

## Install Tensorflow

TODO
