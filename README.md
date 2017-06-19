# CS43L22-Audio-driver

Welcome to the getting started of CS43L22 Audio Driver (RTOS Nuttx)!

### 1. Cloning GIT Repositories
```sh
$ mkdir NuttxAD; cd NuttxAD
$ git clone https://bitbucket.org/nuttx/nuttx.git nuttx
$ git clone https://bitbucket.org/nuttx/apps.git apps
$ mkdir misc; cd misc
$ git clone https://bitbucket.org/nuttx/buildroot.git buildroot
$ git clone https://bitbucket.org/nuttx/tools.git tools
$ cd ../nuttx; git submodule update --init
```

### 2. Configuring Nuttx to compile for STM32F4Discovery
To configure Nuttx you must enter the directory NuttxAD/nuttx/tool and run the configuration script:
```sh
$ cd tools/
$ ./configure.sh stm32f4discovery/nsh
$ cd ../
```

### 3. Compiling a toolchain
```sh
$ cd ../misc/tools/kconfig-frontends/
$ sudo apt-get install gperf flex bison libncurses5-dev texinfo g++
$ ./configure --enable-mconf -prefix=/usr
$ make; sudo make install
$ cd ../../../misc/buildroot/
$ make menuconfig
```

 - Target Architecture: **arm**
 - Target Architecture Variant: **Cortex-M3/M4**
 - Target ABI: **EABI**
 - Toolchain Options –>
   - Binutils Version: **binutils 2.22**
   - Build GCC cross-compiler: **true**
   - GCC compiler Version: **4.8.5** (recommended for Cortex-M4)
   - Build C++ compiler?: **true** (opitional)
   - Build gdb debugger for the Host: **true** (optional but recommended)
   - GDB debugger: **7.4.1** (recommended if building GDB)
 
```sh
$ sudo apt-get install libgmp3-dev libmpfr-dev libmpc-dev
$ make
```
Now it’s necessary to go to directory nuttx-git/nuttx and edit the file .config. The following modificatios are necessary to a succesfull compilation:
```sh
$ cd ../../nuttx
$ make menuconfig
```
 - Build Setup -> Build Host Platform: **Linux**
 - System Type ->
   - Toolchain Selection: **Buildroot**
   - STM32 Peripheral Support ->
     - DMA1: **true**
     - I2C1: **true**
     - SPI1: **false**
     - SPI3: **true**
     - I2S3: **true**
   - Exclude CCM SRAM from the heap: **true**
   - Workaround non-DMA capable memory: **true**
   - SPI Configuration -> SPI DMA: **true**
   - I2S Configuration ->
     - I2S_MCK: **true**
     - Enable I2C transmitter: **true**
 - RTOS Features ->
   - RTOS hooks -> Custom board/driver initialization: **true**
   - Work queue support -> High priority (kernel) worker thread: **true**
 - Device Drivers ->
   - I2C Driver Support: **true**
   - I2C Driver Support ->
     - Polled I2C (no interrupts): **true**
     - Support I2C reset interface method: **true**
     - I2C character driver: **true**
   - I2S Driver Support: **true**
   - USB Host Driver Support: **true**
   - USB Host Driver Support ->
     - Disable isochronous endpoint support: **true**
     - Mass Storage Class Support: **true**
 - System Type -> USB FS Host Configuration -> Enable SOF interrupts: **true**
 - File Systems ->
   - Writable file system: **true**
   - fAT file system: **true**
   - FAT file system -> FAT upper/lower names: **true**
 - Audio Support ->
   - Audio Support: **true**
   - Audio Support ->
     - Exclude Specific Audio Features -> Exclude tone (bass and treble) controls: **true**
     - Use custom device path: **true**
 - Device Drivers ->
   - Audio Device Support: **true**
   - Audio Device Support -> CS43L22 audio chip: **true**
 - Application Configuration ->
   - Examples ->
     - File system mount example: **true**
     - File system mount example -> Use block device: **true**
     - C++ Initialization: **false**
   - System Libraries and NSH Add-Ons ->
     - NxPlayer Media Player: **true**
     - Default root directory to search for media files: **/mnt/sda/music**

### 4. Compiling the RTOS Nuttx

Customizing the build environment:
```sh
$ cd "NuttxAD Folder"
$ export PATH="${PWD}/misc/buildroot/build_arm/staging_dir/bin/:${PATH}"
$ export TOPDIR="${PWD}/nuttx"
$ cd nuttx/
```
Compilation:
```sh
$ make CROSSDEV=arm-nuttx-eabi-
```
### 5. Cloning & Compiling a ST-LINK program
```sh
$ cd ../
$ git clone git://github.com/texane/stlink.git
$ make release
$ cd etc/udev/rules.d
$ sudo cp 49-stlinkv1.rules 49-stlinkv2.rules /etc/udev/rules.d
$ sudo udevadm control --reload-rules
$ sudo cp st-flash /usr/bin
```
### 6. Downloading Nuttx image
```sh
$ cd "NuttxAD/nuttx"
$ st-flash write nuttx.bin 0x08000000
```
### 7. Prepare USB flash storage
Format the USB flash storage into FAT. For example by next command
```sh
$ mkfs.vfat /dev/sdb1
```
Create folder /music
```sh
$ mkdir music
```
Copy files from "NuttxAD"/nuttx/other/audio_samples/ to /music folder of USB flash storage
```sh
$ cp "NuttxAD"/nuttx/other/audio_samples/* /mnt/media/music/
```
### 8. Example usage CS43L22 Audio driver

Power On or reset the stm32f4discovery board. We can see the Nuttx command line promt
```sh
NuttShell (NSH)
nsh>
```
Mount the usb flash device into our file system
```sh
nsh> mount -t vfat /dev/sda/ /mnt/sda
```
Start the NxPlayer program and Enter the help command to view the list of commands
```sh
nsh> nxplayer
NxPlayer version 1.04
h for commands, q to exit
nxplayer> h
NxPlayer commands
================
  balance d%      : Set balance percentage (< 50% means more left)
  device devfile  : Specify a preferred audio device
  h               : Display help for commands
  help            : Display help for commands
  mediadir path   : Change the media directory
  play filename   : Play a media file
  pause           : Pause playback
  resume          : Resume playback
  stop            : Stop playback
  tone freq secs  : Produce a pure tone
  q               : Exit NxPlayer
  quit            : Exit NxPlayer
  volume d%       : Set volume to level specified
 ```
Play the test sample track (cu44k.wav - 44100Hz, 16bit, stereo).
```sh
nxplayer> play cu44k.wav
```
Set the volume value to 50%.
```sh
nxplayer> volume 50
```
Stop the current track and play another one
```sh
nxplayer> stop
nxplayer> play hn.wav
nxplayer> volume 50
```
**Enjoy listening to music**