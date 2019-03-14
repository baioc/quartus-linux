Installing and Running Altera's Quartus II on Linux
====


## 1) Download & Install

Go to [Intel Download Center](https://www.intel.com/content/www/us/en/programmable/downloads/download-center.html).

**Select by Device** -> Choose Device Family -> Choose your Device

Click on the most recent **Web Edition**.

Select the rigth OS and click on **Individual Files**.

Download only what you need.

Give the downloaded script execution permission: `chmod +x QuartusSetup.run`

Run the installation script: `/path/to/script/QuartusSetup.run`


## 2) Add Path dependencies

Create the following script file to be run at startup, one way to accomplish this is to append `source /path/to/script/quartus.sh` to your shell configuration file.

```
export QUARTUS_64BIT=1					# Remove this if running on 32 bit
export ALTERA_ROOT="$HOME/Applications/altera"		# Change this to the path you've installed Altera Quartus at

export QUARTUS_ROOTDIR_OVERRIDE="$ALTERA_ROOT/quartus"
export QSYS_ROOTDIR="$QUARTUS_ROOTDIR_OVERRIDE/sopc_builder/bin"
export QUARTUS_LIBRARY_PATHS="$QUARTUS_ROOTDIR_OVERRIDE/linux/:/lib/x86_64-linux-gnu/"
export SOPC_KIT_NIOS2="$ALTERA_ROOT/nios2eds"
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$QUARTUS_LIBRARY_PATHS"
export PATH="$PATH:$ALTERA_ROOT/quartus/bin"
```


## 3) Install 32-bit compatibility packages

A few 32-bit packages will probably need to be installed on 64-bit systems in order for some tools to work.

Try opening and using any tool directly fom the Quartus GUI. If it doesn't work, run it from the terminal and there should be some complaint about a missing packge or library. Look up on what package you may find this library and install it.

#### The eclipse-nios2 tool requires installing `libgtk2.0-0:i386`

It may also require starting it through the terminal with:

```$SOPC_KIT_NIOS2/bin/eclipse-nios2 -configuration $HOME/.nios2-ide-6.1 $WORKSPACE_ARGS "$@"```

Other packages I've needed to install were gcc­-multilib, lib32ncurses5, libx11-6, libfreetype6, libpng12, libc6, libxtst6, zlib1g, libssl1.0.0 and libssl-dev; generally the i386 versions. You may also need Java 8 or greater.


## 4) USB-Blaster configuration

### Taken from [here](http://www.fpga-dev.com/altera-usb-blaster-with-ubuntu/).

At first, connect the cable and make sure the USB device is recognized:

```
$ dmesg | tail
[...]
[16059.962298] usb 2-2: New USB device found, idVendor=09fb, idProduct=6010
[16059.962301] usb 2-2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[16059.962303] usb 2-2: Product: CV SoCKit
[16059.962305] usb 2-2: Manufacturer: Altera
[16059.962307] usb 2-2: SerialNumber: ARCVSC-123-457

$ lsusb | grep Altera
Bus 002 Device 007: ID 09fb:6010 Altera
```
Take note of the Product ID listed - 6010 in the above example.

The Quartus software will use the Linux built-in usb_device drivers. By default, only root has access to these so we must make sure the user is allowed to access them as well.

jtagd, part of the Quartus tools, is a deamon that provides the interface between the Altera tool accessing the JTAG chain and the USB driver. If not already running, jtagd will be startetd automatically when the Quartus software or jtagconfig is run. You'll usually run these as a user, which means jtagd will also run as a user. That is why edited permission for the usb_device is necessary.

Create a file /etc/udev/rules.d/51-usbblaster.rules, make sure it has read permissions for root, and fill it with this content:

```
SUBSYSTEM=="usb",\
ENV{DEVTYPE}=="usb_device",\
ATTR{idVendor}=="09fb",\
ATTR{idProduct}=="6010",\
MODE="0666",\
NAME="bus/usb/$env{BUSNUM}/$env{DEVNUM}",\
RUN+="/bin/chmod 0666 %c"
```

Edit the value for ATTR{idProduct} to match the Product ID determined before.

If you have more than one Product ID you want this to work for, simply repeat the above lines in the same file and use the other Product IDs for ATTR{idProduct}.

For the changes to take effect, reboot the machine.

Make sure jtagd has access to the list of devices:

```
$ sudo mkdir /etc/jtagd
$ sudo cp $QUARTUS_ROOTDIR_OVERRIDE/linux64/pgm_parts.txt /etc/jtagd/jtagd.pgm_parts
```

Also make sure this file has read access for the user.

The cable should now be recognized as a valid hardware by the Quartus tools. From Quartus, select Tools, Programmer, Hardware Setup... and then select the board from the drop-down list. Now, the Programmer, JTAG Chain Debugger and System console should all recognize and use the USB-Blaster device.

