## Please see the [Goodix Linux Development Discord](https://discord.gg/tqxCu3986U) for more information and help if you need it.

# Goodix 521d Configuration Instructions
Tested on Arch and Manjaro. If you don't have access to the AUR, it's still possible to install this, you'll just need to build it by hand. Join the Discord if you need help.

## First Install
To download this repo: `git clone --recurse-submodules https://github.com/knauth/goodix-521d-explanation.git`

Follow this guide at your own risk. This software is experimental and not guaranteed to do anything. It might break stuff; I'm not responsible for that. Ensure you understand every command you run on your system before you execute it. **Don't use this in secure situations or as your only authentication method.**

You'll need all of the following things, which should be in this git repo:

- A simple C program to reset the reader
- The goodix-fp-dump folder provided by the good people at [Goodix Linux Development Discord](https://discord.gg/tqxCu3986U), with some modifications by [infinytum](https://github.com/infinytum)

### Part 1: Flashing the firmware
We need to flash the firmware of the sensor with an earlier version. Enter the `goodix-fp-dump` directory and run `sudo python run_521d.py` to flash the firmware. It might fail with a timeout error. In this case, go to the usbreset directory and do the following:

1. Compile the code with `gcc usbreset.c -o usbreset.out`
2. Run `lsusb | grep FingerPrint` to find the usb address of the reader. Look specifically at the "bus" and "device" fields.
3. Run `sudo ./usbreset.out /dev/bus/usb/<bus>/<device>`, substituting <bus\> and <device\> with the values from the last command, for example `sudo ./usbreset.out /dev/bus/usb/003/002`

Then run the Python code again. Eventually it should flash successfully. Now we can move on to step 2.

### Part 2: Installing fprintd
Install the latest version of fprintd by running `sudo pacman -S fprintd`.

### Part 3: Installing patched libfprint
The final step! Install the AUR package `libfprint-goodix-521d`.

Now, finally, run `systemctl restart fprintd` to restart the fingerprint service. You should be done! Use `fprintd-enroll` to enroll a new finger.

For getting this working for authentication, checkout [this](https://wiki.archlinux.org/title/Fprint) Arch Wiki page.

As of version 5.24, KDE Plasma supports fingerprint authentication natively using libfprint as a backend. This *should be* painless once libfprint itself is configured. The setup is located in Settings > Users.

Thanks to everyone who got this code working, especially Infinytum (NilaTheDragon on Discord) who did amazing work to complete the driver and helped me get it installed as well :).

## Subsequent Restarts and Dual Booting
Windows is kinda like those seagulls from Finding Nemo when it comes to hardware. It aggressively installs and updates drivers without user intervention. For this reason, you may find your reader breaking after using Windows. To fix this issue you'll need to muddle with the config a bit before you can reflash. Here's how:

1. Stop the fprintd service: `sudo systemctl stop fprintd`
2. Reset the reader (follow the instructions above)
3. Flash the firmware (instructions above)
4. Start the service again: `sudo systemctl start fprintd`

And you should be good to go.
