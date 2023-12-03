# Technical Design Document

This write-up will serve as an attempt of explaining how this cartridge functions. I'm not going to explain the majority of the cart, as you can find most of the explanation in <a href="https://github.com/MouseBiteLabs/Game-Boy-MBC5-Cartridge/tree/main/Technical">my MBC5 repo.</a> Therefore, this write-up will mostly focus on how the multicart portion works and the necessary components to safely implement it.

Please correct me on any information I may have gotten wrong.

## Schematic

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC5-Multicart/assets/97127539/e4664f93-bc41-41bf-80c3-2c70be1f19c4)

## Multicart Basics

A byte is eight bits - corresponding to the eight digital pins D0 to D7. The address pins A0 to A21 determine how many bytes the EEPROM can hold in memory. There are 22 address pins on the 29F033, and they can be either 0 or 1 - therefore the EEPROM can hold 2^22, or 4,194,304, bytes total. Each byte is located at a different "address". Note that the ROM chip 29F033 is a 4 MB EEPROM - see where the 4 comes from? So, this should mean we can hold one 4 MB ROM file on the chip. But, we could also store two 2 MB ROMs, or four 1 MB ROMs, because those add up to 4 MB.

If we want to hold 2 MB of data we only need 21 address pins, A0 through A20, because 2^21 is 2,097,152 - half of the total bytes with 22 address pins. **So omitting one address pin halves the amount of bytes the EEPROM can hold.** But we still have that extra address pin A21; the total space of the chip is *still* 4 MB. If we set A21 at 0 while programming the first game using address pins A0 through A20, then we can set it to 1 to use the rest of the available space on the chip for a second game. Then we simply toggle the A21 pin to 0 or 1 to switch games. These are typically referred to as "memory banks".

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC3-Multicart/assets/97127539/a069322a-3be1-4b8f-8a22-4e24da3ded10)

If we wanted to do four games, we just take the next highest address pin, A20, and toggle that between 0 and 1 as well. Then you have two address pins that can be either 0 or 1, giving you four combinations, and thus four banks of memory.

## D Flip-Flop

A flip-flop is a basic electronic device that can store memory. The output Q will be set to the input D whenever the clock pin is pulsed (normally on the rising edge of the clock signal). If the D pin changes value at all, the output of Q (and /Q, which is the inverse of Q) will not be affected, until the clock signal rises again. The output therefore only updates to whatever is seen on the input when the clock signal rises from low to high.

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC3-Multicart/assets/97127539/a25d2741-df0a-470f-8939-a234603ec142)

The /PRE and /CLR pins are used for setting the Q and /Q outputs to a specific value, ignoring the D input and clock pin. If you set them both high, then the flip-flop can operate normally.

So if we connect the D input to the /Q output, then we can essentially make a clock divider circuit. See the timing diagram below:

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC3-Multicart/assets/97127539/bdfe9fd4-d0a2-4aac-a864-b7ab8bc5d0f4)

When the clock pin rises, the data from the D pin is loaded onto the Q output. Because D is connected to /Q, Q will become whatever /Q was when the clock rises. And thus, /Q will become the opposite of Q. As you can see, this effectively divides the clock frequency in half, switching between 1 and 0 every time the clock pulses.

Now, we can connect the clock pin to a button, with a pull-up resistor to VCC. Whenever we press the button, it shorts the clock pin to GND, and when we release, that gives the clock pin a rising edge as the pull-up resistor pulls the voltage back to VCC. So with every button press, the output of the flip-flop toggles from 0 to 1 or from 1 to 0. Connect that output pin to the upper address pin on the EEPROM, and you now have a way to switch between the two ROMs with a button. Or, if you connect the clock pin to the /RESET pin 30 on the cart edge, which goes low when power is turned off, you now will switch ROMs every time the /RESET pin rises as the game is turned on.

### Two D Flip-Flops

But why stop there? What if we add another flip-flop with the D and /Q pins tied together, but instead connect the output Q of the first flip-flop to the clock input of the second flip-flop?

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC3-Multicart/assets/97127539/060d0502-842d-4363-9a57-4e060e3f168b)

Now we get *another* clock divider. The output Q2 is now half the frequency of Q1. We have just made a binary counter - every button press counts up on the output pins - 00, 01, 10, 11, then back to 00. If we take this *second* output and connect it to our second highest address pin, we can now switch bewteen four separate ROM files.

*The same outputs can be connected to the RAM address pins for the same end effect, so you have separate RAM banks to match the ROM banks.*

### Schmitt Trigger

What's that funky symbol next to the /PRE, CLK, /CLR, and D pins? This indicates these input pins have a Schmitt trigger. Using a Schmitt trigger is a way of cleaning up an input signal, kind of like debouncing. If your input is noisy, or slow, a Schmitt trigger will prevent the input signal from flipping the logic  between 0 and 1 rapidly. It's best seen in this table from the SN74HCS74 datasheet (you can ignore the "Low Power" column for our purposes).

![image](https://github.com/MouseBiteLabs/Game-Boy-MBC3-Multicart/assets/97127539/2eda8b9f-8868-47b5-b597-92edd83af59e)

What's really beneficial here is the "Supports Slow Inputs" column. The /RESET signal (cart edge pin 30) is *very* slow, relative to what logic chips are usually expecting. The rise time is in the milliseconds; on discrete logic chips without Schmitt trigger inputs, the changes are measured in *nano*seconds. If you have a signal too slow, it can cause the bouncing seen in the Chaotic Neutral square. So for the purposes of a /RESET or button triggered multicart, the Schmitt trigger is a *must*.

### Preserving Game Selection

In order to keep the state of the flip-flop persistent across resets/power cycles, so it "remembers" which game you were playing before toggling the clock pin, it must be operational on battery power via the battery management IC, TPS3613. This is somewhat unfortunate - the more devices you run on battery power, the shorter battery life will become. Luckily, the SN74HCS74 has a typical supply current requirement of 0.1 uA (max of 2 uA).

## Tri-State Buffers

U6, U7, and U8 are what's known as tri-state buffers. The output pin Y will be whatever the input pin A is, as long as the OE pin is pulled HIGH. When the OE pin is LOW, then the output of Y is high impedance, or "floating" - not HIGH *or* LOW. U6's and U7's OE pins are connected to /RESGATE, which is HIGH whenever power is supplied to the cartridge from the Game Boy (i.e. when the game is not running on battery power) and LOW whenever the Game Boy is off. U8's OE pin is tied directly to VCC, the cartridge input voltage.

The reason the U6 and U7 buffers are important is the EEPROM is powered by VCC, and *not* by the battery. If we connected the outputs of the flip-flops directly to the address pins, because the dual flip-flop is powered on battery, the states of the address pins could feasibly be at 3 V. The EEPROM's datasheet specifies the voltage on the input pins be no larger than 0.5 V above VCC. Since the VCC pin will be 0 V, we'd be violating the datasheet requirements and potentially damaging the chip.

The buffers that have been chosen *can* handle a voltage on their input pins if VCC is 0 V. There is a small leakage current through the input pins if either or both of the flip-flop outputs are high, however it's only 0.1 uA *max* at ambient temperature, so again a negligible impact.

The reason U8 is important is if it's not included and the /MR pin of U4 is connected directly to the button, then the RC circuit on the button would discharge the battery through the /MR pin, which is internally pulled up to VCC.

## TPS3613 Master Reset Pin

The TPS3613 is already explained in more detail in the MBC5 technical design document, but one minor difference in implementation here is that the master reset pin (/MR) is connected to SW1 (through U8) when SW2A is set to the OFF position. As mentioned, /MR is pulled up internally in the TPS3613 - and when it's grounded with a press of SW1, the /RESET output is pulled low and the positive RESET output is pulled high. This will reset the MBC5 and the Game Boy via the cart edge /RST pin 30, until you release the button. 

## Resources

- <a href="http://www.devrs.com/gb/files/hardware.html">Jeff Frohwein's GameBoy Tech Page</a>
- <a href="https://gbhwdb.gekkio.fi/">Game Boy Hardware Database</a>
- <a href="https://catskull.net/gb-rom-database/">Nintendo Gameboy Game List</a>
- <a href="https://www.ti.com/lit/ds/symlink/tps3613-01.pdf?HQS=dis-mous-null-mousermode-dsf-pf-null-wwe&ts=1698238885366&ref_url=https%253A%252F%252Feu.mouser.com%252F">TPS3613 Datasheet</a>
- <a href="https://www.ti.com/lit/ds/symlink/sn74hcs74-q1.pdf">SN74HCS74 Datasheet</a>
- <a href="https://www.ti.com/lit/ds/symlink/sn74ahc1g08.pdf">SN74AHC1G08 Datasheet</a>
- <a href="https://github.com/Gekkio/gb-schematics/blob/main/DMG-CPU-06/schematic/DMG-CPU-06.pdf">Gekkio's Game Boy Schematic Resources</a>

## License
<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/80x15.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>. You are able to copy and redistribute the material in any medium or format, as well as remix, transform, or build upon the material for any purpose (even commercial) - but you **must** give appropriate credit, provide a link to the license, and indicate if any changes were made.
