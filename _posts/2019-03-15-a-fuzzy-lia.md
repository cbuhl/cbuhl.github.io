---
layout: post
title: "Fuzzy requirements for a microcontroller LIA"
excerpt_separator: "<!--more-->"
categories:
  - Post Formats
tags:
  - lia
  - requirements
  - embedded
  - development
---

So. I want to make a Lock-in amplifier on a cheap STM32 microcontroller. I've opted for a fair bit of muscle, and I'm using the STM32F767zi, based on a Nucleo development board. 

Everything is rather fuzzy but the initial (and most simple) requirements for the uC setup are as follows:

<!--more-->
 - 2 ADC channels. One for the reference and one for the signal. 
     - The sampling frequency must be better than 240kHz. I aim to have a reference signal at around 30kHz, and then to have about 8x oversampling/dithering. 
     - For a clock cycle of 216MHz, that leaves me 216000/240 = 900 cycles between sample. That should be possible.
 - 1 DAC channel for the reference
 - 1 DAC channel for an output signal. This is not awfully necessary, and should generally only update with maybe 1kHz. I'm designing the instrument to have full integration with a computer, anyway. 
 - The DMA should be used as heavily as possible. I have essentially no clue how it works, but it sounds clever. This especially matters for the ADC.
 - Internal reference should be maintained with a counter. From [Sonaillon et al.](https://aip.scitation.org/doi/10.1063/1.1854196), I gather that using a linear interpolation between 64 points in a look-up table is plenty fine for the harmonic distortion which is introduced. I can also add an analogue low pass filter on the output stage. Initially I have assumed that a 32bit counter is necessary.

### Plan for development
The LIA (Lock-In Amplifier) will come in a two critically different editions:

 1. Internal reference generation
 2. For a simple version, it's possible to just multiply the two signals, for a number of samples, and take the RMS value of the signal. The phase should be adjustable. 
 3. The next step-up is to have a 90deg phase delay on a part of the reference, and then get both the in-phase and the quadrature signal (I/Q mode). This is really a must-have, I think. 
 4. PLL allowing for an external reference signal.
 
 99. Expanding the firmware, so some of the other ADCs can be read in a slow-mode, for interfacing. Digital input/output is also very nice to have, to control experiments and so on. 
 
The first will allow me to do almost all of the experiments I need. The second is necessary if it should get any real traction for other people.


### Ideas for algorithms
There are a few ways of doing the calculations, but it is fairly straight forward, even for the I/Q mode, I should be able to generate an output on the DAC with a minimal delay, and then use the internal look-up table for comparison. The look-up table could be both a sine and a cosine version, giving me the 90deg shift inherently. 
