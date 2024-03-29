# NFCheckRem
Patch for Nintendo consoles to remove the read-only check on amiibos and allow for rewritable Ntag215 NFC tags

# Installation
- Nintendo 3DS : Ensure you have the latest Luma3DS version, then go into the folder corresponding to your console and download the .ips file. 
Put this file into your SD card in the following folder : `/luma/sysmodules/`  then 
ensure you have "Enable loading external FIRMs and modules" and "Enable game patching" enabled in the Luma3DS settings (hold SELECT on boot)
- Wii U : (not yet implemented)
- Switch : (not yet implemented)

# Why ?
When writing an Amiibo to a blank Ntag215 NFC tag with an app such as TagMo, the tag will become read-only on some parts of the data.

This data includes the Amiibo game character id, variant, figure type, model number and series.

This means that if it's read-only, you cannot change the figure stored on the NFC tag, which therefore mean having to buy multiple tags for every Amiibo you want.

# What does this do ?
This modifies the NFC system module to disable the checks that are made on those areas, yes the console checks if the tag is read-only.

By disabling these checks, this means you can have write-enabled tags and they would still work on consoles with the patch installed

And thus, you can reuse your tag forever without being constrained to have it as one specific Amiibo (you still have to rewrite it every time you want to change it)

# Technical details
## How does it work ?
Documentation for the IPS file format : https://zerosoft.zophar.net/ips.php

For example, the N3DS file is made like this :
5 bytes : `PATCH`, 3 bytes `0x02 0x24 0xB0`, 2 bytes `0x00 0x0D`, 13 bytes filled with `0x00`, 3 bytes `0x02 0x26 0x08`, 2 bytes `0x00 0x0D`, 13 bytes filled with `0x00`, 3 bytes `EOF`

## Why those values ?
The files names are the applications ID for NFC scanning.
- For the Nintendo 3DS : https://www.3dbrew.org/wiki/Title_list#00040130_-_System_Modules
- For the Wii U : (not yet implemented)
- For the Switch : (not yet implemented)

The offsets are found by doing a search on the code files of the NFC apps.
- For the Nintendo 3DS :
  - Open GodMode9, press the HOME button -> Title manager -> NAND / TWL
  - Naviguate to 0004013000004002 or 0004013020004002 depending on your console model (Old or New)
  - Press (A) -> Open title folder
  - Select the .app file, press (A) -> NCCH image options... -> Extract .code
  - You now have in your SD card in `/gm9/out/` either a 0004013000004002.dec.code or 0004013020004002.dec.code depending on your console model
  - You can open it with the hexadecimal editor of your choice or use the builtin one in GM9
  - Search for the following values : `0x00 0x04 0x5F 0x00 0x00 0x00`, you will get multiple matches
    - The values correspond to a portion of the last two "Must match" values found here : https://www.3dbrew.org/wiki/Amiibo#Page_layout
    - The selected portion is big enough to reduce potential false positives and small enough to not be annoying to write. Yes i am lazy
  - You will see all of the "lock bytes" (including the ones in NFC page 0x0 and 0x82) and "must match" bytes are all placed conveniently next to eachother
  - Note the offsets of `000224B0` and `00022608` (for the second match), which is where the full "string" starts (with the `0x0F 0xE0` raw binary match requirement from NFC page 0x0)
  - Note as well that the full data is composed of 16 bytes, but since the last 3 bytes are already 0x0 we can ignore those and we get a size of 13 bytes
  - Replacing the rest of the bytes with zeros will basically empty the byte check constraints and remove the read-only check protection
  - There you go, you got all of the values needed to compose the patch

- For the Wii U : (not yet implemented)

- For the Switch : (not yet implemented)
