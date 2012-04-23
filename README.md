freebsd-beaglebone
==================

Automate building a bootable FreeBSD image for BeagleBone.

This basically automates Damjan Marion's instructions at
  http://people.freebsd.org/~dmarion/beaglebone/creating_bootable_sd_card/

============================================================

What is a BeagleBone?
--------------------

The BeagleBone is an ARM-based development/test board from the
BeagleBoard project.  It features a modern ARMv7 processor
and a wealth of on-chip peripheral support.  Details can be found at:
   http://beagleboard.org/bone

The BeagleBone is an easy way to learn about ARM development: it is
inexpensive (about $85) and does not require any special skills or
hardware to use.  You connect it to your desktop with a Mini USB cable
(which both powers the board and provides serial console access) and
boot it from a MicroSDHC card with the system image.

Once you have the BeagleBone working, you can expand it by connecting
it to Ethernet, adding USB peripherals (such as external disk drives
or wireless network interfaces) or attaching new hardware to the
connectors on the board.

============================================================

How to Build a Disk Image
-------------------------

The beaglebsd.sh script can build a complete bootable FreeBSD image
ready to be copied to a MicroSDHC card.  The script runs on
FreeBSD-CURRENT (March 2012 or later).

Using the script to build an image consists of three steps:

1. EDIT beaglebsd-config.sh

You need to set the image size to match the MicroSDHC card you'll be using
and verify that certain directories have enough free space.  There are
lots of comments in this file to help you.

2. RUN beaglebsd.sh as root

   $ sudo /bin/sh beaglebsd.sh

The script will first check that you have all the needed sources.  If
you don't, the script will tell you exactly how to obtain the missing
pieces.  Follow the instructions and re-run the script until you have
everything.

As soon as it finds all the required pieces, the script will then
compile everything and build the disk image.  This part of the process
can take several hours.

3. COPY the image to a MicroSDHC card.

The script will print the command you need to do this.

To verify the image, try mounting the card on your FreeBSD system.  It
should show up with two slices: The first slice is FAT formatted and
contains the necessary boot machinery, including the U-Boot loader.
The second slice is a standard FreeBSD UFS partition with the actual
system installed.

============================================================

How to boot the BeagleBone
--------------------------

1. CONNECT the board to your FreeBSD system using a Mini-USB cable.
The Mini-USB connector is on the bottom of the BeagleBone.

2. ACCESS the serial console on the board from your desktop:
   $ sudo cu -l /dev/ttyU1 -s 115200
(If this doesn't work, you may need to load the uftdi driver into your kernel.)

3. INSERT the MicroSDHC card into your BeagleBone

4. REBOOT the BeagleBone by depressing the small switch next to the
Ethernet port.

============================================================

Anatomy of a BeagleBone Boot Image
----------------------------------

The AM3358 SoC ROM code expects the MicroSDHC card to use
MBR partitioning and to have a bootable FAT partition.

The ROM code loads a file called "MLO" from the FAT filesystem into
on-chip RAM.  The MLO program is responsible for initializing the
external RAM and loading the next-stage boot.  The on-chip RAM is 128k
and some is used for ROM execution, so the MLO must be less than 110k.

In our case, MLO is a stripped-down version of the U-Boot boot loader.
It initializes the external RAM and then loads the full U-Boot program.

U-Boot is a highly modular boot loader system that can be used on a
wide variety of systems.  The loader is configured with built-in
script commands that can be partially overridden by additional
commands in the uEnv.txt file.

The uEnv.txt file supplied here directs U-Boot to load a file called
kernel.bin and execute it at address 0x80200000.

TODO: I think it would be better overall to have U-Boot load ubldr (a
version of loader(8) that uses U-Boot for system device access) and
ubldr load the kernel.  In particular, ubldr can load the kernel
directly from UFS, which would allow us to use installkernel to
install the kernel.  Unfortunately, ubldr does not yet support MBR
partitions.

============================================================