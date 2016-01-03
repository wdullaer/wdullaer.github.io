I recently acquired a brand new broadwell HP Spectre x360. This is one of those new Lenovo Yoga inspired convertible laptops. The hardware is really nice: great aluminium build, sharp display, good keyboard, awesome extra wide touchpad. By default it comes with Windows, which I just can't get used to anymore, so I did what I always do: format the harddisk and install Linux. Because the laptop hardware is close to the bleeding edge, this requires a little bit more fiddling than it would on another laptop. (Not even Microsoft has properly nailed the software for convertible laptops). In this blog post I'm keeping track of the tweaks I needed to do, to get the laptop working just the way I want.

## Get Linux to install
This is probably the most annoying part: due to the way HP configured UEFI and the keys that are included on the machine, you can't install another operating system without doing some tweaks in the BIOS.

1. Download and install the latest BIOS from Windows. This isn't strictly necessary, but HP doesn't offer the BIOS as a download from its website and you'll want the latest version to disable the F5 led.

2. Create a UEFI compatible bootable USB key with your favourite Linux image. [For ubuntu](http://askubuntu.com/questions/395879/how-to-create-uefi-only-bootable-usb-live-media) this actually easier than creating a standard bootable USB key.

3. Press F10 to get into the BIOS menu at boot and disable the secure boot feature. In case you are going for a dual boot scenario: disabling secure boot will not prevent you from booting into windows.

4. Press F9 at boot to get into the boot medium list. You can now select and boot your linux image.

## Stuff that works

Disabling of the keyboard when in tablet mode.
The touchscreen
The touchpad, wifi, media keys, bluetooth etc.



## Get the sound to work
Change the following line in `/etc/default/grub` from
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```
to
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_osi='!Windows 2013' acpi_osi='!Windows 2012'"
```

## Get firefox to play nice with the touchscreen
Grab and drag extension

## Get the touchpad and touchscreen to do more than the basics
See touchegg post and throw money at the people building Wayland support for you favourite DE.
See the right click post for diabling the right click zone (especially important given the size of the touchpad)

So far gnome 3 is treating this laptop very nice (especially with the good hdpi and touchscreen support)

# Remaining niggles to sort out
Hibernate (currently only resume works, so the idle time is rather short)
Disable touchpad in tablet mode (the keyboard properly disables)

## Further Reading
* [Create bootable UEFI Ubuntu USB Stick](http://askubuntu.com/questions/395879/how-to-create-uefi-only-bootable-usb-live-media)
* http://h30434.www3.hp.com/t5/Notebook-PC-Sound-and-Audio/HP-spectre-x360-on-linux/td-p/4980797
* http://blog.wdullaer.com/blog/2015/10/08/touchegg/
* right click post
