# Guitar Pedal

a custom guitar pedal PCB - Crater is reference to the Comet Distortion modification of  the BOSS DS-1 distortion pedal

[Docs](Reference)
[Github](https://github.com/nano13579/crater)

## Day 2

I set up the basic schematic adding the peripherals I had mentioned earlier. I'm also adding a Micro-B USB and debug and I/O and some buttons + LEDs. And a crystal. Schematic is still in planning stage in which I've currently just placed various components I plan on using. However, I think I want to shift my focus to the passive components/ICs that I'll be using for the effects that I plan to have. 

Addons for Effects:
- 
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