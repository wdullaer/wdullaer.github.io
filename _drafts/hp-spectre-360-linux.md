## Stuff that works
Disabling of touchpad and keyboard when in tablet mode.
The touchscreen
The touchpad, wifi, media keys, bluetooth etc.

## Get Ubuntu to install
Deactivate Secure Boot in the BIOS.
press F10 to get into the BIOS menu.
Press F9 to be able to select the boot device.
While you're at it, update to the latest BIOS version and disable the F5 led.

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

So far gnome 3 is treating this laptop very nice (especially with the good hdpi and touchscreen support)

## Further Reading
* http://h30434.www3.hp.com/t5/Notebook-PC-Sound-and-Audio/HP-spectre-x360-on-linux/td-p/4980797
* http://blog.wdullaer.com/blog/2015/10/08/touchegg/
