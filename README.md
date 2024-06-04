# NFCheckRem
Patch for Nintendo consoles to remove the read-only check on amiibos and allow for rewritable Ntag215 NFC tags

# Installation
- Nintendo 3DS : Ensure you have the latest [Luma3DS](https://github.com/LumaTeam/Luma3DS/) version, then go into the folder corresponding to your console and download the .ips file. 
Put this file into your SD card in the following folder : `/luma/sysmodules/` then ensure you have "Enable loading external FIRMs and modules" and "Enable game patching" enabled in the Luma3DS settings (hold SELECT on boot)
- Wii U : (not yet implemented)
- Switch : (not yet implemented)

### Note for 3DS users
The patch will do nothing if wumiibo is enabled, ensure wumiibo is disabled before attempting to scan any Amiibo or NFC tag

# Why ?
When writing an Amiibo to a blank Ntag215 NFC tag with an app such as [TagMo](https://github.com/HiddenRamblings/TagMo), the tag will become read-only on some parts of the data.

This data includes the Amiibo game character id, variant, figure type, model number and series.

This means that if it's read-only, you cannot change the figure stored on the NFC tag, which therefore mean having to buy multiple tags for every Amiibo you want.

# What does this do ?
This modifies the NFC system module to disable the checks that are made on those areas, yes the console checks if the tag is read-only.

By disabling these checks, this means you can have write-enabled tags and they would still work on consoles with the patch installed

And thus, you can reuse your tag forever without being constrained to have it as one specific Amiibo (you still have to rewrite it every time you want to change it)

# Technical details
TL;DR : This removes all "if" constraints that prevent an unlocked tag from being read, by replacing them with an instruction that does nothing or an instruction that "always validate the check" depending on the need

## How does it work ?
Documentation for the IPS file format : https://zerosoft.zophar.net/ips.php

For example, the N3DS patch file is made like this :
- 5 bytes : `PATCH` (`0x50 0x41 0x54 0x43 0x48`)
- 3 bytes `0x02 0x23 0x74`, 2 bytes `0x00 0x06`, 6 bytes `0x00 0x46 0x00 0x2D 0x00 0x46`
- 3 bytes `0x02 0x23 0x84`, 2 bytes `0x00 0x02`, 2 bytes `0x00 0x46`
- 3 bytes `0x02 0x23 0x8D`, 2 bytes `0x00 0x01`, 1 byte `0xE0`
- 3 bytes `0x02 0x23 0xB4`, 2 bytes `0x00 0x02`, 2 bytes `0x00 0x46`
- 3 bytes `0x02 0x23 0xC6`, 2 bytes `0x00 0x02`, 2 bytes `0x00 0x46`
- 3 bytes `0x02 0x23 0xE4`, 2 bytes `0x00 0x02`, 2 bytes `0x00 0x46`
- 3 bytes `0x02 0x23 0xF7`, 2 bytes `0x00 0x01`, 1 byte `0xE0`
- 3 bytes `EOF` (`0x45 0x4F 0x46`)

## Why those values ?
The files names are the applications ID for the NFC service.
- For the Nintendo 3DS : https://www.3dbrew.org/wiki/Title_list#00040130_-_System_Modules
- For the Wii U : (not yet implemented)
- For the Switch : (not yet implemented)

The offsets are found by decompiling the code using Ghidra and using a bit of logic.

Let's take the New Nintendo 3DS patch file again for example :
  - Open GodMode9, press the HOME button -> Title manager -> NAND / TWL
  - Naviguate to 0004013000004002 or 0004013020004002 depending on your console model (Old or New)
  - Press (A) -> Open title folder
  - Select the .app file, press (A) -> NCCH image options... -> Extract .code
  - You now have in your SD card in `/gm9/out/` either a 0004013000004002.dec.code or 0004013020004002.dec.code depending on your console model (Old or New)

Next take this file and store it on your computer. Download Ghidra from https://github.com/NationalSecurityAgency/ghidra and open it.

Make a new project or use an existing one as you will, the most important is that you open the file you extracted from your 3DS with these settings : ARM processor, v6 variant, size 32, little endian

These settings are set to match the 3DS CPU architecture, so that the code actually makes sense once decompiled

Execute all decompilation methods, then once it has done doing its shenanigans, go into the top bar -> Search -> Memory...

Here, input the following values : `00 00 00 04 5F 00 00 00` then press "Search All". You will get multiple matches, select the second one and close the search windows
  - The values correspond to the last two "Must match" values found here : https://www.3dbrew.org/wiki/Amiibo#Page_layout
  - The selected portion is big enough to reduce potential false positives and small enough to not be annoying to write. Yes i am lazy

Still in the listing tab, if you scroll to the right of the selected data, you will see this :

![image](https://github.com/Golem642/NFCheckRem/assets/65229557/687abebe-c074-4413-84e3-33092f2518e5)

These 2 "XREF" are indications that those values are used somewhere in the code, specifically in the FUN_00022364 function. Double click on that name and you made it inside the read-only check function, scroll back up a bit to find the actual start of the function

![image](https://github.com/Golem642/NFCheckRem/assets/65229557/3c73212e-e669-45ff-a8a5-b630c165c8e3)

If you're wondering why we didn't choose the first search result, it is because the function it is linked to doesn't seem to have any references to it. If you look above, you can see an XREF[1] at the end of the FUN_00022364 line, whereas if you go to the function of the other search result there will be none, therefore meaning this function is seemingly unused.

Now it's pure logic and following the flow of the code. Everytime you see a reference to a LAB_000XXXXX, it means it's a conditional jump. And everytime that happens, you need to check by double clicking on that LAB name whether this jump goes to continue the function's code or to exit it.

The goal is to execute as much as possible of the code you can see in the decompiled tab on the right. You can select a portion of the values on the left to see what does it translates to on the right (i usually select everything until i meet another LAB)

So with that all in mind, here are all of the final replaced values :
- Address `0x022374` : `0x00 0x46 0x00 0x2D 0x00 0x46` -> Replaces the first and second conditional jumps by an empty instruction
  - Here `0x00 0x2D` is an already existing instruction in the original code, so i put it back instead of doing another record which would use more bytes in the ips file
- Address `0x022384` : `0x00 0x46` -> Replaces a conditional jump by an empty instruction
- Address `0x02238D` : `0xE0` -> Replaces the "conditional jump" instruction by a "forced jump" instruction
- Address `0x0223B4` : `0x00 0x46` -> Replaces a conditional jump by an empty instruction
- Address `0x0223C6` : `0x00 0x46` -> Replaces a conditional jump by an empty instruction
- Address `0x0223E4` : `0x00 0x46` -> Replaces a conditional jump by an empty instruction
- Address `0x0223F7` : `0xE0` -> Replaces the "conditional jump" instruction by a "forced jump" instruction

Here the empty instruction i chose is `MOV r0,r0` which basically means you set the register 0 to itself, and converted into ARM this is equal to `0x00 0x46` (Thanks to this website : https://armconverter.com/)

The "forced jump" can be deduced from the code by looking at the conditional jumps instructions name : You can see BEQ, BNE, BCC, and just B which is what matters to us. All of the rest are conditional jumps, while the B instruction will always jump forward to the said amount of bytes. So we just have to replace whatever conditional jump instruction there is with the B instruction to do the trick

You may notice i stopped at the LAB_000223fc, that's because at first it seemed like i had already passed all of the checks that prevents writable tags from being read. Although i wasn't sure, i tested with what i already had and sure enough it was working so i kept it like that, no need to do more.

Thanks to all of the listed links, as well as the [NFC tools](https://play.google.com/store/apps/details?id=com.wakdev.wdnfc) app and [AbandonedCart](https://github.com/HiddenRamblings/TagMo/issues/729) who was willing to make changes to TagMo to allow writing for those special tags
