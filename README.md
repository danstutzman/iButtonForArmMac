# How to download iButton temperature data onto ARM-based Mac 

The iButton is a small hardware device to take temperature readings every five minutes and store them internally.  To read the temperatures, you plug it into the DS9490 adapter, then plug the adapter's USB cable into your computer.

Maxim Integrated provides an easy driver download for Windows.  For my MacBook Air M1 (which has an ARM chip), the setup process was more complicated.  For example, I couldn't use Homebrew `brew install libusb` because it was for the wrong architecture.  So here's what I did to get it working.

Hardware that you'll need:
- Mac with ARM-based chip
- iButton DS1925EVKIT which includes:
  - DS1925 (button-sized temperature logger)
  - DS1402 (cable that converts logger to network cable)
  - DS9490 (adapter to convert network cable to USB-A)
- USB-C to USB-A hub

## Install Java

Download and install a Java JDK
  - I used `1.8.0_202`
  - When I run `file /usr/bin/java` I get the following output:
```
/usr/bin/java: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e:Mach-O 64-bit executable arm64e]
/usr/bin/java (for architecture x86_64):	Mach-O 64-bit executable x86_64
/usr/bin/java (for architecture arm64e):	Mach-O 64-bit executable arm64e
```
  - I added the following line to my `~/.zshrc`: `export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home`

## Create Terminal running Rosetta

1. Go to Finder, enter Utilities
1. Right-click Terminal, click Copy
1. Type Cmd-V to paste another copy of Terminal
1. Rename newly created copy of Terminal to `Terminal Rosetta`
1. Right-click Terminal Rosetta, click Get Info
1. Make sure `Open using Rosetta` is checked
1. close the Get Info window

## Build libraries

1. Run Terminal Rosetta in Applications/Utilities
1. cd to your `iButtonForArmMac` directory
1. `tar xvzf libusb-1.0.24.tar.bz2`
1. `cd libusb-1.0.24`
1. `make clean`
1. `./configure`
1. `make`
1. `cd ..`
1. `tar xvzf libusb-compat-0.1.7.tar.gz`
1. `cd libusb-compat-0.1.7`
1. `LIBUSB_1_0_CFLAGS="-I$PWD/../libusb-1.0.24/libusb -static" LIBUSB_1_0_LIBS="-L$PWD/../libusb-1.0.24/libusb/.libs -lusb-1.0" ./configure`
1. `make clean`
1. `make`
1. `cd ..`
1. Download https://www.maximintegrated.com/design/tools/appnotes/5917/OneWireViewer-Linux.zip into `iButtonForArmMac` directory
1. `unzip OneWireViewer-Linux.zip`
1. `cd OneWireViewer-Linux`
1. `cp Makefile.new OneWireViewer-Linux/PDKAdapterUSB/Makefile`
1. `cd OneWireViewer-Linux/PDKAdapterUSB`
1. `make clean`
1. `make`
1. `sudo cp -v libonewireUSB.jnilib /Library/Java/Extensions`
1. Connect your USB-A to USB-C hub to your Mac.
1. Connect the DS1402 cable to the DS9490 adapter.
1. Connect the DS9490 adapter to the USB hub.
1. When I run `ioreg -p IOUSB -w 0` I get the following output:
```
+-o Root  <class IORegistryEntry, id 0x100000100, retain 26>
  +-o AppleT8103USBXHCI@01000000  <class AppleT8103USBXHCI, id 0x1000002ad, registered, matched, active, busy 0 (1840 ms), retain 110>
  | +-o USB3.0 Hub@01200000  <class IOUSBHostDevice, id 0x100001d3a, registered, matched, active, busy 0 (16 ms), retain 37>
  | +-o USB2.0 Hub@01100000  <class IOUSBHostDevice, id 0x100001d4f, registered, matched, active, busy 0 (33 ms), retain 38>
  |   +-o IOUSBHostDevice@01130000  <class IOUSBHostDevice, id 0x100001d63, registered, matched, active, busy 0 (9 ms), retain 23>
  +-o AppleT8103USBXHCI@00000000  <class AppleT8103USBXHCI, id 0x1000002a9, registered, matched, active, busy 0 (44 ms), retain 46>
```
1. Run `make test`.  I get the following output:
```
javac -d . -classpath .:OneWireAPI.jar java/test/TestMain.java
java -classpath .:OneWireAPI.jar TestMain
PDKAdapter.OpenPort_native called
opening DS2490-1
select port returned: true
adapterDetected returned: true
reset returned: 1
datablock returned: 3301D4210D00000065
reset returned: 1
getByte returned: 01
getByte returned: D4
reset returned: 1
getBit returned: 80
find first: true
address: F70000003DA1D681
address: 650000000F25F453
powerdelivery: true
powerdelivery after bit: true
powerdelivery after byte: true
calling owRelease
```

## Run OneWire Viewer
1. Download OneWire Viewer MAC from https://www.maximintegrated.com/en/products/ibutton-one-wire/one-wire/software-tools/viewer-mac.html
1. Extract the .zip file
1. Move the newly created OneWireViewerMac directory to your `iButtonForArmMac` folder
1. Run Terminal Rosetta in Applications/Utilities (if you aren't already)
1. cd to your `iButtonForArmMac` directory
1. `java -jar OneWireViewerMac/OneWireViewerMac/OneWireViewer.jar`
1. See OneWireViewer instructions at https://www.maximintegrated.com/en/products/ibutton-one-wire/one-wire/software-tools/viewer.html
