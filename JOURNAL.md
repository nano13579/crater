# Guitar Pedal

a custom guitar pedal PCB - Crater is reference to the Comet Distortion modification of  the BOSS DS-1 distortion pedal

[Docs](Reference)
[Github](https://github.com/nano13579/crater)

## General Vocab

**Impedance**: opposes AC and changes depending on signal frequency, measured in Ohms

**Low Pass Filter**: allows only certain low frequency under a certain threshold to pass; for audio it is used to reduce sharp high frequency noises

**Analog Buffer**: 

**Source**: device/unit sending an electrical signal

**Load**: device/unit receiving or measuring an electrical signal

## Day 1

There are a lot of designs for single effect units (i.e. only overdrive etc.) so I will likely make a digital multi effects unit that has controls for fuzz, distortion, and overdrive. Initially thinking about using a single effect pedal, I was leaning towards a modular analog design for its warm tones, specifically including PT2399 Echo Chip IC and the JRC4558 op amp in my original plans. However, since the pedal will control multiple effects, I'll likely use a DSP for controlling many effects at once/separately. Between Daisy Seed and Teensy - microcontroller based DSPs - the Daisy Seed seems to have more coherent docs and greater support and a DAC, which the Teensy lacks and will likely give me a headache later. Yet I’m not sure I enjoy the prepackaged feel or the high cost of the Daisy microcontroller so I’ll likely mimic the DSP using an STM32H7. Things to add: audio coder-decoder, clock, buffer and protection, power management, and I/O.

Some General Things
- LDO (Low-Dropout Regulator) - is used to dissipate input voltage into heat for steady output voltage
- Step-Down Buck Converter - a high input voltage is turned on and off via Pulse Width Modulation (PWM), then an inductor and capacitor smooth out the current. A feedback loop controls the frequency of the PWM to account for fluctuations in input voltage. Overall a higher DC voltage is smoothly converted to a lower DC voltage.
- Codec - has both DAC and ADC converters for further processing and playback

After Looking at the Daisy Seed Schematic
- Since LDOs are not as efficient when the change in voltage is relatively large, the Daisy Seed uses an LP2985 as a Linear LDO Regulator to reduce noise and minimize the instability w/in the output voltage, resulting in a high Power Supply Rejection Ratio (PSRR)
- The Daisy Seed uses the TPS6217x as a step-down buck converter 
- Uses TI's PCM3060 Asynchronous Stereo Audio Codec. The master clock within the PCM3060 is not synchronized with the STM's internal clock (hence the 'asynchronous'), in which the codec's clock and the MCU's clock can be set as controller/worker and vice versa. The audio sample rate is determined by the codec's internal clock. Although the default clock speed for the PCM3060 is 48 kHz, some ppl on reddit have recommended a) a lower frequency (for a more rough sound) or b) a high frequency (to reduce noise and/or make the sound more clean) - I'll probably figure out which way I'll go later

For Next Time
- As much as I was trying to avoid the high price of the Daisy, the STM32H7 is actually relatively expensive compared to other MCUs in the STM32 series. Maybe I'll look for some alternatives or something.
- The Daisy Seed was last updated early in 2022, so some components may be outdated and/or have better alternatives - look for these too.

## Day 2

I set up the basic schematic adding the peripherals I had mentioned earlier. I'm also adding a Micro-B USB and debug and I/O and some buttons + LEDs. And a crystal. Schematic is still in planning stage in which I've currently just placed various components I plan on using. However, I think I want to shift my focus to the passive components/ICs that I'll be using for the effects that I plan to have. 

## Day 3

![alt text](<excalidraw.png>)

I made a general wiring diagram to better visual connections between components. Op-amps will be more detailed as I further explore possible options. Input and output jacks will both be 1/4 inch mono. This will, unfortunately, have to be a 4 layer pcb. I have opted to use the STM32H750IBT6, over the K6 used in the most recent version of the Daisy Seed since the K6 uses a Ball Grid Array - something which cannot easily be hand soldered and would require more expensive pcb manufacturing. The Daisy Seed uses 64MB Synchronous DRAM, so I plan on using the IS42S16400J-xT, with 54 pins an a TSOP Type II package. I also found IS42S16400J-xC as a near identical symbol/footprint compared to that of the xT, so I chose on a whim. Looking at KiCad's default symbol library, I could not find 8MB QSPI Flash, however, I did find the IS25WP256D-xM, which has a capacity of 256MBit (more than what is needed/referenced in the Daisy but that is fine hardware wise). Since I have yet to plan the details for specific effects and controls, I have added all the necessary components to the schematic save user interactives, op-amps, and filtration peripherals, which will be added at a later date. 

For Next Time
- Finish wiring SDRAM and Flash to the STM
- Create more concrete plans for operational amplifiers etc.

## Day 4

I learned that the xT in the IS42 SDRAM represented solderable pins over the difficult to handle BGA, so I ended up having chose the right package. I set up the basic power pins for the STM, placing 100nF capacitors per VDD pin and a single 10uF bulk capacitor. Since the audio is handled using an external ADC and DAC in the form of the PCM codec, I can connect VDDA to both VSSA and VREF+. I [found](https://electronics.stackexchange.com/questions/589927/purpose-of-vref-in-mcus-adcs) that VREF+ marks an absolute reference voltage (the difference in potential energy between a point and a defined, or absolute, value) which the STM32H7 can compare input analog signals to. In simpler chips VREF+ is preconnected to VDDA, however this STM provides the ability to use a lower reference voltage for higher precision. However, since I do not plan on using the onboard ADC (much? - will disconnect the pin later if need be), I will not need this extra precision, so the VCC will be used as a reference instead. I found general guidelines for power supply decoupling in the [STM32H750IB datasheet](https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://www.st.com/resource/en/datasheet/stm32h750ib.pdf&ved=2ahUKEwjZ9bLWxIiVAxWVKEQIHahbDHcQFnoECCEQAQ&usg=AOvVaw3bOUsi3I8JUORLRKVO5P65). 

For Next Time
- Same goals as last time

## Day 5

Added a ferrite bead to bridge DC to the noisy AC rail. The bead provides resistance against the alternating current but allows DC current to flow through, powering the analog components. I chose 120R impedance at 100MHz, only because I've seen examples of this value bead having been used in other similar chips. I'm not sure if it will work for my board, will likely have to find and check against a possible bead's ZRX graph. I also realized that I forgot the external clock as a part of the DSP to my wiring diagram. Some general notes that I took when wiring the [IS42 SDRAM](https://www.issi.com/WW/pdf/42-45S16400J.pdf): This is a 16 bit wide chip and requires data to be transferred in its full width, so the data mask pins (LDQM and UDQM) are used to write data in the form of two 8 bit chunks. FMC_NBL0 handles the lower byte and FMC_NBL1 handles the upper byte, the STM32 can choose to write to one or to the other or both, whichever is not in use is driven high, telling the IS42 to ignore the byte. FMC itself stands for Flexible Memory Controller and allows the STM32 to integrate external memory sources and map these external IC's - processes will albeit be a little slower than on-board hardware. DQ0 to DQ16 on the IS42 are I/O. The 12 addressable pins A0 through A11 specify data storage/retrieval location in terms of row and column addresses. Both I/O and and addressable pins are bidirectional to allow for both reading and writing. Control lines are wired appropriately with pins that have predetermined FMC functions (found in the Pin Description section of the SMT32 datasheet). I'm not entirely sure that I wired the VDD and GND pins right, will check this later.

For Next Time
- Same as last time, but implement external crystal oscillator

## Day 6

Finished setting up the external clock, I still have to choose a specific crystal that is rated for the 18pF caps, but will likely do this later. Also completed wiring the LP2985 1.8V LDO, which I actually misplaced in my initial wiring diagram. 9V is first stepped down to 3.3V using the TPS6. This 3.3V is then brought to 1.8V to power flash and codec etc. I had initially wired the TPS62172DSG according the "Typical Application Schematic" in the documentation for the chip, only to realize that the diagram was for a different version of the TPS6. The chip in the documentation had a fixed 1.8V output while I am using a fixed 3.3V output, so the feedback pin should be connected to ground (docs say analog, but I'm still not sure) or no connect. The TPS62172DSG also has a lower output current compared to that of the chip shown in the diagram, so the value of the inductor will have to change (value TBD).

For Next Time
- Draw PCM3060 connections

## Day 7

So the value of the inductor on the buck converter stays the same, it's just the Isat rating which changes apparently (to something greater than 750mA - check later), which I will need to take into consideration when choosing parts. An external 9V is being applied to the system to power the op amps in both the input and output stages, so I'll implement a mid-rail generator (virtual ground at 4.5V) using a resistor divider. For the input stage op amp I'll use something similar to the TL072, and the output stage will use the RC4580 or similar. I don't really care too much about small differences in how clean the sound is etc., but if my prototype doesn't work I want to have used cost efficient parts. I also finished connections between the IS25WP QSPI flash and the STM32H7, made using QUAD SPI Bank 1 pins for the singular external flash. I currently have the RESET pin in no connect, but I'll have to research this later for a possible pull to high. 

For Next Time
- Same goals as last time