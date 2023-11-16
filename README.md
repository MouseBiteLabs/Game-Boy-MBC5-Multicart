# Game Boy MBC5 Multicart

This is my design of a flashable MBC5-based multicart for the Game Boy. You can make a 2-in-1 or 4-in-1 cartridge, that changes games based on a button press or power cycling the Game Boy. With this board you can also make a single game, but with the addition of a pressable reset button on the cartridge.

The features are as follows:

- Able to make a cartridge with either 2x 2 MB games or 4x 1 MB games
- Optionally adds a pressable button to the cartridge (no shell cutting required)
  - The button can be configured to cycle games on the cart, reset the console, or both
- Configurable to have separate save data for each game, or to share the same save data across multiple games
- Fully compatible with the <a href="https://www.gbxcart.com/">GBxCart RW</a> so you can transfer games and save files to and from the board

Some examples of fun cartridges you can make with this board are:
- A single Pokemon game that has a reset button
- Both Pokemon Red and Blue with separate saves
- Both Pokemon Red and Blue with *the same* save (save files are compatible between versions!)
- Both Pokemon Red and Blue with *the same* save that can switch between versions *during gameplay without resetting* (hotswap!)
- A single copy of Powerpuff Girls with two separate save files

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC5-Multicart/assets/97127539/75a34d8d-a28f-4cf6-b867-b6cfd8e9bbdb)

All gerbers and source files can be found in this repo, as this project is fully open source. Technical documentation of the board can be found in the Technical folder.

## Important Things Before You Start

1) To use this board, you need to have an original Game Boy game that uses an MBC5 mapper chip. <a href="https://catskull.net/gb-rom-database/">You can find a list of games and their mappers here</a>. Use the search function.
2) You will need to remove the MBC5 from your donor cartridge for use on this board. This will require a hot air rework station or a hot plate. There's a list below of other parts you can re-use from the donor cartridge.
3) When soldering parts on, it's a good idea to put kapton tape or otherwise cover the bottom cartridge edge. You do not want to get solder on the cartridge contacts.
4) I am not responsible for any damage you do to your self or your property. I do not guarantee design compatibility. You may encounter issues with certain games! Attempt this project at your own risk.
5) If you are using this board to make games other than for personal use, you must have permission from the originator to use and distribute any ROM images or other related material. You are responsible for making sure you adhere to any license requirements.

DO NOT use my circuit boards for profiting from stolen work - this especially includes homebrew content, ROM hacks, and using fan-made labels without permission from the originator. **Support original creators!**

## Board Characteristics and Order Information

The zipped folder contains all the gerber files for this board. The following options **must** be chosen when ordering boards for yourself.

- Thickness: 0.8mm
- Surface Finish: ENIG
- Gold Fingers: Yes, 30° chamfer

**Add this note to orders:** "The milling.gbr layer indicates where an unplated slot should be added to the board. Please add an unplated slot as indicated."

**Currently not selling on Etsy, but will in the future. Stay tuned.**

<a href="https://www.etsy.com/shop/MouseBiteLabs"><img src="https://github-production-user-asset-6210df.s3.amazonaws.com/97127539/239718536-5c9aefe3-0628-4434-b8d8-55ff80ac3bbc.png" alt="PCB from Etsy" /></a> 

You can use the zipped folder at any board fabricator you like. You may also buy the board from PCBWay using this link (disclosure: I receive 10% of the sale value to go twoards future PCB orders of my own):

[link]

<a href="">The board is also listed on OSH Park as well.</a> **Be sure to get them in 0.8mm thickness if you order from here.**

## Required Equipment

The EEPROMs on the board needs to be programmed somehow. I recommend using the GBxCart, as mentioned. These boards are fully compatible with it, and it makes reflashing games extremely easy using <a href="https://github.com/lesserkuma/FlashGBX">lesserkuma's FlashGBX software</a>.

Alternatively, you can buy an EEPROM programmer with a TSOP adapter. The downside to this method is that you have to desolder the chip every time you want to program it. The <a href="https://flashcatusb.com/">FlashcatUSB</a> is one popular option in retro spaces.

## Battery Safety

When assembling a board, I recommend populating all the parts *except* the battery, and getting it to run initially without it. This is to make it easier to fix any solder connections that might need fixing, without having to worry about getting the battery hot. And if you need to rework anything near the battery after you've put it on the board, be safe and remove it before putting a hot soldering iron next to it.

And this should go without saying, but if you're assembling these boards with a hot plate or hot air, *do not* solder the battery on this way. You should use an iron, and keep heat off of the battery as much as possible. 

(Also check polarity!)

## Board Configurations

There's a lot to cover here. There are four separate switches to configure, and two different sizes of SRAM to pick from. You also can add a real-time clock to the cartridge for games that needed it.

*Note that you can simply solder bridges from the middle pads of the switches to the left or right to the "ON" or "OFF" positions, instead of installing an actual switch.*

### Real-Time Clock (RTC)

If your game uses an RTC, populate C2, C3, R2, and X1 as indicated in the BOM.

If your game does *not* use the RTC function, you can omit these parts. But you need to bridge the pads of C3.

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC3-Cartridge/assets/97127539/d46ea632-6318-4e5d-bf40-c4e9a7f52aa1)

### SRAM Size Selection

There are two sizes of SRAM you can use, and the type you use will dictate how your save data is stored on the cartridge and if it's shared between games or not.

- If you use a 1 Mbit SRAM (like the AS6C1008), then each game you put on the cartridge will have its own separate save files. If you program multiple copies of the same game, then you can effectively have multiple save files on the same cart. Easy example is Pokemon games - they only let you have one save file, but if you put Pokemon Red on the multicart four times, then that gives you four separate save files!
- If you use a 256 Kbit SRAM (like the AS6C62256) then all the games you put on the cartridge will *share* the same RAM space.  This feature is probably only useful for Pokemon games. As an example: this means that if you made a cart that has Pokemon Red and Blue on it, you would play both versions with the same save data, as save data between the two games are compatible with each other.

### Game Cycling Mode Switch (SW2)

SW2, split into two separate switches SW2A (top half) and SW2B (bottom half), controls how you change games on the cartridge. The following table describes the different settings:

| Mode | SW2A (Top) | SW2B (Bottom) | Game Change Method | Resets the Console? |
| ---- | ---------- | ------------- | ------------------ | ------------------- |
| 1    | OFF        | ON            | Cycle power        | Yes                 |
| 2    | ON         | ON            | Press button       | Yes                 |
| 3    | OFF        | OFF           | Press button       | No                  |
| 4    | ON         | OFF           | No change          | Yes                 |

*Note: Mode 4 will effectively act as a single-game cartridge with an added reset button. This mode does not require populating SW1, U5, and U7. But it requires you to place SW3 into mode C, and restricts your game size to 2 MB and RAM size to 256 Kbit.*

### Game and Save Data Configuration Switch (SW3)

SW3, split into two separate switches SW3A (bottom half) and SW3B (top half), controls the order in which the ROM banks (the games) and the RAM banks (the save data) cycle. This table assumes you are using a 1 Mbit SRAM chip, as a 256 Kbit SRAM will share save data across all ROMs.

| Mode | SW3A | SW3B | ROM Banks  | RAM Banks   | Game 1     | Game 2     | Game 3     | Game 4     |
| ---- | ---- | ---- | ---------- | ----------- | ---------- | ---------- | ---------- | ---------- |
| A    | ON   | OFF  | 2x 2 MB    | 4x 256 Kbit | ROM1, RAM1 | ROM1, RAM2 | ROM2, RAM3 | ROM2, RAM4 |
| B    | OFF  | OFF  | 4x 1 MB    | 4x 256 Kbit | ROM1, RAM1 | ROM2, RAM2 | ROM3, RAM3 | ROM4, RAM4 |
| C    | ON   | ON   | 2x 2 MB    | 2x 256 Kbit | ROM1, RAM1 | ROM2, RAM2 |            |            |
| D    | OFF  | ON   | 2x 1 MB    | 2x 256 Kbit | ROM1, RAM1 | ROM2, RAM2 |            |            |

*Note: Mode D is useless. Just use Mode C instead.*

### Game Mode Examples

Here's a list of example cartridges you can make with these settings:
1) Pokemon Red, Blue, Yellow, and Green on one cartridge with separate save files that changes via powering the Game Boy off and on again: **Mode 1B** with 1 Mbit SRAM
2) Pokemon Gold and Silver with the same save file that hotswaps when you press the button on the cartridge (changing games during gameplay): **Mode 3C** with 256 Kbit SRAM
3) Pokemon Crystal with a reset button: **Mode 4C** with 256 Kbit SRAM

## How to Program Games

When using the GBxCart to program multiple games to the cartridge, it's recommended to turn SW2A and SW2B to the OFF position. To change ROM/RAM banks to program, simply press the SW1 tactile button between programming steps. Using FlashGBX's "Refresh" button should show that the game information on the left of the screen changes after pressing the button.

If you're programming the flash chip separate from the board, you'll need to concatenate all the ROM files together into one large 4 MB file to program the chip with.

## Test Points and Final Checkout

On the back of the board are five test points. Here's where they are connected:

- TP1: SRAM supply voltage
- TP2: Battery voltage (after R1)
- TP3: Battery voltage (positive terminal of battery)
- TP4: Ground
- TP5: VCC input voltage

After you assemble your game, you should measure the current out of the battery. But first, you should program it with the GBxCart, or if you programmed the EEPROM separately, put it into a Game Boy and cycle power once. Then, flip the PCB upside down on a non-conductive surface (not your leg), and set your multimeter in DC millivolts (or volts). Put the positive probe on TP3 and the negative probe on TP2. If you used a 10kΩ for R1, as indicated in the BOM, you should read a voltage in the single or tens of millivolts for non-RTC games, or up to 60 mV for games using a real-time clock. If you have something much higher, especially voltages above 60mV, then you likely have an issue or short circuit on the board somewhere.

### Current Draw Measurements

The revision of MBC3 chip you are using will influence how much current draw you get, and thus how long your battery life will last. For the test set up, I am replacing the battery with a regulated DC power supply set for 3 VDC for consistency, on my regular MBC3 cart board with an MM1134 chip for U4, and brand new AS6C62256 SRAM.

These numbers are for reference only - to help you decide which MBC3 revision to use for your game, if you have a choice. The current required for retaining save data on the multicart will be slightly higher due to the extra components compared to a regular MBC3 cartridge.

| Rev   | P/N      | Current draw (no RTC) | Current draw (with RTC) |
| ----- | -------- | --------------------- | ----------------------- |
| MBC3  | P-1      |           ?            |            ?             |
| MBC3  | LR385364 |          ?             |            ?             |
| MBC3  | BU3631K  |          ?             |            ?             |
| MBC3A | LR38536B |          0.1 uA             |            1.5 uA             |
| MBC3A | BU3632K  |          0.5 uA             |            1.5 uA             |
| MBC3A | P-2      |          0.5 uA             |            3.9 uA             |
| MBC3B | BU3634K  |          ?             |             ?            |
| MBC3B | P-2      |          0.4 uA            |              3.7 uA           |

*If you have one of the revisions of MBC3 chips that either have a question mark in the table, or is missing from the table, please contact me!*

## Bill of Materials (BOM)

| Reference Designators | Value/Part Number      | Package        | Description        | Source                                           |
| --------------------- | ---------------------- | -------------- | ------------------ | ------------------------------------------------ |
| B1                    | CR2032, CR2025, CR2016 | CR2032         | Backup Battery     | [https://mou.sr/3SeAzfT](https://mou.sr/3SeAzfT) |
| C1                    | 0.1uF                  | 0603           | Capacitor (MLCC)   | [https://mou.sr/3ENc15O](https://mou.sr/3ENc15O) |
| C2                    | 15 pF                  | 0603           | Capacitor (MLCC)   | [https://mou.sr/3PPorjO](https://mou.sr/3PPorjO) |
| C3                    | 15 pF                  | 0603           | Capacitor (MLCC)   | [https://mou.sr/3PPorjO](https://mou.sr/3PPorjO) |
| C4                    | 0.1uF                  | 0603           | Capacitor (MLCC)   | [https://mou.sr/3ENc15O](https://mou.sr/3ENc15O) |
| C5                    | 0.1uF                  | 0603           | Capacitor (MLCC)   | [https://mou.sr/3ENc15O](https://mou.sr/3ENc15O) |
| C6                    | 10uF                   | 0603           | Capacitor (MLCC)   | [https://mou.sr/3mZtSkF](https://mou.sr/3mZtSkF) |
| C7                    | 0.1uF                  | 0603           | Capacitor (MLCC)   | [https://mou.sr/3ENc15O](https://mou.sr/3ENc15O) |
| C8                    | 0.1uF                  | 0603           | Capacitor (MLCC)   | [https://mou.sr/3ENc15O](https://mou.sr/3ENc15O) |
| Q1                    | 2N7002                 | SOT-23         | N-Channel FET      | [https://mou.sr/3rgfh6J](https://mou.sr/3rgfh6J) |
| R1                    | 10k                    | 0603           | Resistor           | [https://mou.sr/3riR7IH](https://mou.sr/3riR7IH) |
| R2                    | 330k                   | 0603           | Resistor           | [https://mou.sr/3PZ2pvj](https://mou.sr/3PZ2pvj) |
| R5                    | 10k                    | 0603           | Resistor           | [https://mou.sr/3riR7IH](https://mou.sr/3riR7IH) |
| R8                    | 10k                    | 0603           | Resistor           | [https://mou.sr/3riR7IH](https://mou.sr/3riR7IH) |
| R9                    | 130k                   | 0603           | Resistor           | [https://mou.sr/3MjXliy](https://mou.sr/3MjXliy) |
| R10                   | 49.9k                  | 0603           | Resistor           | [https://mou.sr/3Q3NRZO](https://mou.sr/3Q3NRZO) |
| SW1                   | See note               | 5.2 x 5.2mm    | Tactile Switch     | [https://mou.sr/3uipCQz](https://mou.sr/3uipCQz) OR see note |
| SW2                   | CAS-D20TA              | J Form Lead    | Dual SPDT          | [https://mou.sr/46gGqF1](https://mou.sr/46gGqF1) |
| SW3                   | CAS-D20TA              | J Form Lead    | Dual SPDT          | [https://mou.sr/46gGqF1](https://mou.sr/46gGqF1) |
| U1                    | 29F032, 29F033         | TSOP-40        | Flash EEPROM       | AliExpress or eBay                               |
| U2                    | MBC5         | QFP-32         | MBC5 Mapper        | Donor MBC5 Game Boy cartridge                    |
| U3                    | AS6C62256, AS6C1008    | SOP-28, SOP-32 | SRAM               | [https://mou.sr/3sFegFF](https://mou.sr/3sFegFF) |
| U4                    | TPS3613                | MSOP-10        | Battery Management | [https://mou.sr/45Ir2kh](https://mou.sr/45Ir2kh) |
| U5                    | SN74HCS74              | TSSOP-14       | Dual Flip Flop     | [https://mou.sr/3QYGEuT](https://mou.sr/3QYGEuT) |
| U6                    | SN74AHC1G08            | SOT-23-5       | 2-input AND Gate   | [https://mou.sr/3Bku2qj](https://mou.sr/3Bku2qj) |
| U7                    | SN74AHC1G08            | SOT-23-5       | 2-input AND Gate   | [https://mou.sr/3Bku2qj](https://mou.sr/3Bku2qj) |
| X1                    | 32.768 kHz             | Radial         | Crystal Oscillator | [https://mou.sr/3ZteKuy](https://mou.sr/3ZteKuy) |

### Note about SW1

SW1 can be either short or long. If the button is long enough, it will sit inside the shell in a way that lets you press it by lightly pressing on the cartridge shell. This way, you can activate it without removing it from the console. This is helpful if you want to change games via a button press.

- If you want to make the button pressable when the cartridge is fully assembled, you need to find a switch that has an extended stem on AliExpress, eBay, or Amazon (**if you find a compatible switch on Mouser or Digikey, please let me know**). The measurements will be 5.2 mm x 5.2 mm, with a height of 3.5 mm. Sometimes these parts are listed as "4 mm x 4 mm" or "5 mm x 5 mm" instead. Check the datasheet, if available, to see if it'll fit the footprint. They should have listing pictures similar to the ones seen here:

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC3-Multicart/assets/97127539/57f93dff-8af2-446d-9d30-69dc870ae9df)

- If you don't care about pressing the button while it's in the shell, like if you are making a multicart that changes games by cycling the power where you wouldn't need it, then you can simply use the TS18-5-25-SL-260-SMT-TR (https://mou.sr/3uipCQz).

### Usable Donor Cartridge Parts

You can use a few parts from the donor cart on the new board to save some money. Note that you will generally get better reliability with new parts as opposed to old ones. For example: I have seen failed RAM chips from donors in the past.

1) **C1, C2, R2, X1: RTC Components** - You can move over these parts if you're using the real-time clock function. Note that C1 and C2 are designated as C2 and C3 on my board. They're the same value, so they're interchangeable.
2) **U2: MBC5** - This one is required
3) **U3: SRAM** - You can use this part *only if* the sum of the RAM space for the games you're using is the same or less amount of RAM that the donor cartridge has. If you plan to have separate save data for every game, you'll probably need to buy an AS6C1008.

You could probably transfer over most of the 0.1uF capacitors but they're pretty cheap anyway, so I generally just recommend buying new resistors and capacitors.

## Things to Remember

- The 29F032 and 29F033 have been known to occasionally be defective upon arrival. They're either used, or new old stock, and usually only available from AliExpress.
- The footprint for the battery can fit a CR2032, CR2025, or CR2016 with solder tabs. The only difference is the mAh capacity (larger number = longer life). If you get Panasonic tabbed batteries, you may have to trim the battery tabs to make them fit on the footprint.
  - For untabbed coin cells, you can find battery retainer adapters online, <a href="https://retrogamerepairshop.com/products/hdr-game-boy-game-battery-retainer?variant=40511013290156">like this one.</a>
- Generally, ROM sizes are conveyed in terms of kilobytes (KB) and megabytes (KB, MB). RAM size is usually conveyed in terms of kilobits or megabits (Kbit, Mbit). You can convert Kbit and Mbit to KB and MB by dividing Kbit or Mbit by 8. For example, 256 Kbit = 32 KB.

## Revision History

### v1.1
- Fix schematic errors
- Extended cart edge down by 0.25 mm for better fitment
- Added OSHW logo and "SUPPORT ORIGINAL CREATORS!"

### v1.0
- Prototype revision

## Resources and Acknowledgements

- <a href="http://www.devrs.com/gb/files/hardware.html">Jeff Frohwein's GameBoy Tech Page</a>
- <a href="https://gbhwdb.gekkio.fi/">Game Boy Hardware Database</a>
- <a href="https://catskull.net/gb-rom-database/">Nintendo Gameboy Game List</a>
- <a href="https://www.gbxcart.com/">insideGadgets discord server for GBxCart RW compatibility requirements</a>
- <a href="https://github.com/lesserkuma/FlashGBX">Lesserkuma's FlashGBX software</a>
- <a href="https://www.ti.com/lit/ds/symlink/tps3613-01.pdf?HQS=dis-mous-null-mousermode-dsf-pf-null-wwe&ts=1698238885366&ref_url=https%253A%252F%252Feu.mouser.com%252F">TPS3613 Datasheet</a>
- Board outline modified from <a href="https://tinkerer.us/projects/homebrew-gameboy-cartridge.html">Dillon Nichols's Homebrew Gameboy Cartridge project</a>
- Some components from <a href="https://github.com/adafruit/Adafruit-Eagle-Library">Adafruit's Eagle parts library</a>
- Some components from <a href="https://www.snapeda.com/">SnapMagic</a>
- Thank you to <a href="https://github.com/Gekkio">gekkio</a> for their deep Game Boy knowledge resources, and for collaboration in demystifying some of the design choices on Game Boy cartridges
- Thanks to the awesome members of the <a href="https://moddedgameboy.club/">Modded Gameboy Club</a> for their feedback and support during the entire project development

## License

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/80x15.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>. You are able to copy and redistribute the material in any medium or format, as well as remix, transform, or build upon the material for any purpose (even commercial) - but you **must** give appropriate credit, provide a link to the license, and indicate if any changes were made.

©MouseBiteLabs 2023
