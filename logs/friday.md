# Friday Hack-a-Thon Log


## 4 PM

Just got home from work.

> First things first: back up my hard drives.

## 4:20 PM - Formatting the SD Card

Okay, I've decided to just install Raspbian on my Raspberry Pi 3 micro SD card and I will try to re-partition the hard drive after.

First, I need to format my 32GB micro SD card. To start, I need it's name:

    $  mount
    /dev/sda1
    ...
    /dev/mmcblk1p1

Before I can format it, I need to unmount it:

    $ umount /dev/mmcblk1p1

I first I thought `unmount` was not installed. But, actually it is `umount`.  Somebody trying to save themselves one keystroke 30 years ago. How convenient.


Now I just need to format it:

    $ mkdosfs -I -F32 /dev/mmcblk1p1

(Why, you may ask, did I choose the FAT32 file system? Good question. I have no idea.)

Now I will use the Gnome Disk Utility to check that the formatting worked correctly.

![screenshot of formatting success](/resources/sd_formatted.png)

Huzzah!


## 4:30 PM - Installig the Raspbian OS

I got the Raspbian OS from the official download site [here](https://www.raspberrypi.org/downloads/raspbian/).

>  WHoops. It says the download will take an hour.

Ain't nobody got time for that.  Okay, let's read ahead. To install the Raspbian OS, I will follow the offical Linux guide [here](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md).


## 5:45PM - Raspbian Finally Downloaded

Okay, Raspbian finally downloaded.  Yeesh.

I copied the Raspbian disc image over using the command:

    sudo dd bs=4M if=2017-07-05-raspbian-jessie.img of=/dev/mmcblk1 conv=fsync

This command takes a few minutes. Note that I had to copy ofer the whole drive name, not just a single partition (`mmcbblk1p1`).

![Raspbian OS Installed](/resources/raspbian_install_success.jpg)

Success!

Installation was a breeze.  Before ejecting the SD card from my laptop, I ran `sync` from the command line, just to clear ensure the write process was complete and it was safe.


## 6PM - Connect to WIFI - Wasting Time

Oh look, I can connect to WIFI because my Raspberry Pi 3 has a WIFI card. Cool.

...Luckily, I only wasted 15 minutes surfing the internet.


## 6:15PM - Re-Partition the Hard Drive

Okay, by default the Raspbian OS takes up the entire 32GB of my micro SD card. But I need to add a partition for my LFS build.

Here are a couple  guides for re-partitioning the SD card:

* [a StackOverflow answer](https://raspberrypi.stackexchange.com/questions/499/how-can-i-resize-my-root-partition)
* [AdaFruit post](https://learn.adafruit.com/resizing-raspberry-pi-boot-partition/edit-partitions)

First, I will try this with the Linux GUI program `gparted`, it seems pretty standard.  First, install it:

    $ sudo apt-get install gparted

Next, run it:

    $ gparted

And I see something like this:

![gparted started](/resources/gparted_started.png)

Okay, first things first, I want to save off the boot partition on the SD card in case I nuke it (which I think I might have to):

    $ sudo cp -r /dev/mmcblk1p1 .

It is at this point I decided I needed to unmount all the paritions on the SD card.  But... they are already unmounted.  I have no idea how that happened.  Or how I could copy from them if they are unmounted.

> Spooky

Anyway, I went ahead and resized the main 32GB Raspbian OS parition (`ext4`) into three partitions:

* 8 GB Raspbian OS partition (`ext4`)
* 22 GB partition for the LFS build (`ext4`)
* 2 GB swap partition (`linux-swap`)

![repartition with gparted](/resources/repartitioned.png)

Now, did I ruin anything? Will the Raspberry Pi still boot?

> It still booted!

## 7PM - Getting Started for Real

* [2.6 - Setting The $LFS Variable](http://www.linuxfromscratch.org/lfs/view/development/chapter02/aboutlfs.html)

Okay, now that I have a computer to build my Linux-From-Scratch (LFS) Operating System on, I have fullfilled all the early steps in the book.  Now we have to set a `LFS` environment variable in Bash. In this case, pointed to the mount point of the partition I want to build in:

    $ export LFS=/mnt/lfs

* [2.7. Mounting the New Partition](http://www.linuxfromscratch.org/lfs/view/development/chapter02/mounting.html)

Right, but that is only helpful if we actually mount the damn partition:

    $ sudo mkdir -pv $LFS
    $ sudo mount -v -t ext4 /dev/mmcblk0p3 $LFS

Okay, I just went down a long rabbit hole regaurding `nosuid` and `nodev` parameters being set on my LFS partition.  The book just says `remount` the partition to remove them, but these settings seem to be separate for the `/dev/mmcblk0p3` and `/mnt/lfs` items in the `mount` print-out.  Why? I have no idea.  But I managed to fix the mount point, just not the original partition. For now I'm going to assume that is good enough and move on.

> Fingers crossed.

Next it says I need to mount my swap partition. Fair enough.

    $ sudo /sbin/swapon -v /dev/mmcblk0p4

## 8PM - Cheap-Ass Keyboard Disaster

I bought a cheap USB keyboard on Amazon for this project.

> The # and | keys are broken.

Damn. Those are important in bash.

I am sitting here playing around with various custom key maps.  But this is not what I wanted to spend my time on.


## 8:15PM - The Big Download

* [Chapter 3. Packages and Patches](http://www.linuxfromscratch.org/lfs/view/development/chapter03/introduction.html)

Okay, now I have to download a bunch of packages worth of source code and probably some kernel stuff. I fully expect this to take a long time, over the WIFI on my Raspberry Pi.

First, we will create a directory for all the source code to live while I am working:

    $ sudo mkdir -v $LFS/sources
    $ sudo chmod 755 $LFS/sources

Now we need to grab the list of package web addresses:

    $ wget http://www.linuxfromscratch.org/lfs/view/development/wget-list

And, if all goes well, we can just use `wget` again to retrieve all the packages:

    $ wget --input-file=wget-list --continue --directory-prefix=$LFS/sources

> And now we play the waiting game...
> [The waiting game is boring, let's play Hungry, Hungry Hippos.](https://www.youtube.com/watch?v=9JVNMmsN3Co)

## 8:30 - Dinner

Nom nom nom...

## 9PM - Done for the Night

Shower and Bed

Chapters 4 and 5 tomorrow.
