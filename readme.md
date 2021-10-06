# Goodix 521d Configuration Instructions
Tested on Arch and Manjaro. 

To download this repo: `git clone --recurse-submodules https://github.com/knauth/goodix-521d-explanation.git`

Follow this guide at your own risk. This software is experimental and not guaranteed to do anything. It might break stuff; I'm not responsible for that. Ensure you understand every command you run on your system before you execute it.

You'll need all of the following things, which should be in this git repo:

- A PKGBUILD for a lower version of fprintd-libfprint2
- A simple C program to reset the reader
- The goodix-fp-dump folder provided by the good people at [Goodix Linux Development Discord](https://discord.gg/tqxCu3986U), with some modifications
- The patched version of libfprint2 created by [infinytum](https://github.com/infinytum)

## Part 1: Flashing the firmware
We need to flash the firmware of the sensor with an earlier version. Enter the `goodix-fp-dump` directory and run `sudo python run_521d.py` to flash the firmware. It might fail with a timeout error. In this case, go to the usbreset directory and do the following:

1. Compile the code with `gcc usbreset.c -o usbreset.out`
2. Run `lsusb | grep FingerPrint` to find the usb address of the reader. Look specifically at the "bus" and "device" fields.
3. Run `sudo ./usbreset.out /dev/bus/usb/<bus>/<device>`, substituting <bus\> and <device\> with the values from the last command, for example `sudo ./usbreset.out /dev/bus/usb/003/002`

Then run the Python code again. Eventually it should flash successfully. Now we can move on to step 2.

## Part 2: Installing older fprintd
First, make sure you have no other versions of libfprint or fprintd. You should be able to remove them with `sudo pacman -Rc libfprint fprintd-libfprint2`. This is important since the following steps will fail if these programs are already installed.

Next we'll install the older version of fprintd-libfprint2. Go to the `fprintd-libfprint` directory, which contains a modified PKGBUILD for the AUR. Make sure you have the following dependencies installed:

`git gtk-doc meson pam pam_wrapper python-cairo python-dbus python-dbusmock python-gobject`

All of these should be installable using pacman, please let me know if not.

Run `makepkg -si` to install the program.

## Part 3: Installing patched libfprint
The final step! Go to the `libfprint` directory. Run `git branch` and make sure the only option is `feat-5110-images` (press q to quit the viewer). If not, you can run this command in the main directory to get the correct branch:

`git clone --branch feat-5110-images https://github.com/infinytum/libfprint.git`

Next run the following commands:

`sudo meson -Dgtk-examples=true -Ddrivers=all -Dprefix=/usr . _build`

`sudo ninja -C _build install`

These commands build and install the patched code.

Now, finally, run `systemctl restart fprintd` to restart the fingerprint service. You should be done! Just use `fprintd-enroll` to enroll a new finger. A word of caution- the driver doesn't wait for you to place your finger down, so be quick.

Thanks to everyone who got this code working, especially Infinytum (NilaTheDragon on Discord) who did amazing work to complete the driver and helped me get it installed as well :).
