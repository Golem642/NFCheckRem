# NFCheckRem
Patch for Nintendo consoles to remove the read-only check on amiibos and allow for rewritable Ntag215 NFC tags

# Installation
- Nintendo 3DS : Ensure you have the latest Luma3DS version, then go into the folder corresponding to the console and download the .ips file
Put this file into your SD card in the following folder : SD:/luma/sysmodules/
Ensure you have "Enable loading external FIRMs and modules" and "Enable game patching" enabled in the Luma3DS settings (hold SELECT on boot)

# Why ?
When writing an Amiibo to a blank Ntag215 NFC tag with an app such as TagMo, the tag will become read-only on some parts of the data.
This data includes the Amiibo game character id, variant, figure type, model number and series.
This means that if it's read-only, you cannot change the figure stored on the NFC tag, which therefore mean having to buy multiple tags for every Amiibo you want.

# What does this do ?
This modifies the NFC system module to disable the checks that are made on those areas, yes the console checks if the tag is read-only.
By disabling these checks, this means you can have write-enabled tags and they would still work on consoles with the patch installed
And thus, you can reuse your tag forever without being constrained to have it as one specific Amiibo (you still have to rewrite it every time you want to change it)

# Technical details

hold on i'm finishing writing this, i gotta save that before
