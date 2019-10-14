# logue_RandomLFOTutorial

A "Random" filter LFO MOD effect for your logue sdk compatible synthesizers, with commented code and a brief explanation on how this all works...

**This is provided for informational and entertainment purposes only, any use of the following information is done solely at your own risk! No guarantee is made on the suitibility or accuracy of any information provided.**

### A quick word...
I've been having a ton of fun creating these plugins, and it's thirsty work. If you like stuff like this and my other work, by all means feel free to contribute whatever you can to the fund to help fund the beer supply!

This can be done here :  [Donate!](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=MSTCVLXMG7Z5J&source=url)





**So glad you could attend....**

So, you just bought your 'logue synthesizer, and realized it only has one LFO. And not only that, the single LFO only has saw, triangle and square modulation options! No sample-and-hold to be found here.

Fortunately, the logue SDK provides a means to extend the capabilities of your synthesizer! While you currently cannot generate custom LFO waveforms, you can of course create your own modulator effects (choruses and the like, typically), reverb and delay effects. But, they don't "have" to fall in that category. You can do as you please, within the CPU time alloted. For this demo, we are going to create a mod effect that emulates a filter that is modulated by a "random" LFO. The randomness is provided by a 1024 entry floating-point table generated at random.org.

Why a 'fixed' pre-calculated table? Well for the same reason this demo is all "straight C" (no C++ is actually required to make oscillators and effects) - for simplicities sake for beginners to start to be able to create their own effects. And for practical purposes, this works well enough.

This demo uses a 'chamberlin' style filter as described in the book "Musical Applications For Microprocessors", by Hal Chamberlin. A definite must read once you've got the handle of getting the SDK to make some noise! Personally I find these filters not only extremely efficient (the math is fairly simple - compare with the actual biquad implementation!). A very good writeup of this filter is currently available [here]( https://www.earlevel.com/main/2003/03/02/the-digital-state-variable-filter/)

I should probably note here, that as there currently is no actual "documentation" provided by KORG, most if not all of this is me "figuring this out". It is entirely possible, if not likely, that some of this is actually wrong, or interpreted incorrectly. Proceed at your own peril!
Getting Started

First, you must be able to build the demos provided by KORG, and be familiar with how to install them. There are some videos on YouTube etc. to help you get to this point. And, I'm not about to copy the entire SDK over at this time, but if you have installed the SDK on your system and can build the "waves" oscillator, or the 'biquad' modfx test, then you should be good to go.

Please note, this was built on a minilogue-centric system, as I do not own a prologue. So my plugins are generally built in the platform/minilouge-xd/..... folders. 

First, create a new directory in the SDK modfx\tests (e.g. logue-sdk\platform\minilogue-xd\modfx\tests\) folder called "RandomDemo". Into this folder copy the source contents (the files : makefile, makefilePrologue, project.mk, manifest.json, randomDemo.c and the randomtable.h files). The demos provided by KORG place the source in the \src folder, however I myself like to have the source files in the project folders themselves, this ultimately can be entirely up to you. The demos provided by KORG place the source in the \src folder, however I myself like to have the source files in the project folders themselves, this ultimately can be entirely up to you.

Next, enter the folder of the effect via your console (same means you used to build the demos), and type "make clean". This is generally a good idea before running make, as it cleans up any previous object files, and ensures you are building from scratch.
Building the source

To build the source, type "make" for the minilgoue xd, and if you wish to build this for the prologue, you could either copy this into the prologue platform directory, or, I've included a makefile that will allow you to do this. Just enter "make -f prologuemakefile" and it should generate the prlgunit.
The code

I believe the code itself is well commented to explain what is going on, but the fundamentally, there are 3 areas of concern:

'void MODFX_INIT(uint32_t platform, uint32_t api)'

- This is called whenever the effect is actually loaded. It is not called when the effect is simply turned off and on. Here, you will initialize any variables, classes etc that you wish to use. Personally, any module variables I use I try to initialize here, I do not rely solely on the compiler initializing these with the variable initializers for me despite writing it that way as well.

'void MODFX_PROCESS(const float *main_xn, float *main_yn,const float *sub_xn, float *sub_yn,uint32_t frames)'

This is the actual DSP function. It is here, where you will process the samples. With MOD FX , there are four separate buffers - two buffers for inputs ("main" and "sub" - applicable to prologue only when in dual / split modes), and for outputs: "main" and "sub"- again applicable to the prologue only. If you are properly implementing the prologue, you must have a separate, independent (duplicate) processor for the "sub" channel. For our demo we will merely pass the signal straight through for this channel. For the minilogue-xd, it is pointless to implement the sub channel as you will be wasting CPU cycles for something that is not supported.

The samples are stereo in, stereo out, and the buffers appear to store these as LRLRLR - alternating left right samples. This is also true for the resultant output. Our effect is MONO, so we load in the LEFT channel (and ignore the right channel input - note we still have to "skip" that sample), and store the resultant value twice in the output buffer for both channels.

'void MODFX_PARAM(uint8_t index, int32_t value)'

This is called whenever one of the time/depth knobs are turned. Any math you can 'pre-calculate' e.g. a frequency that is based off of the knob position, is wisely calculated here, as there is no need to waste any CPU cycles calculating anything you do not need to in the DSP loop!

The value provided by the API is a Q31 type, however the line 'const float valf = q31_to_f32(value);' converts this to a much more convenient float type, which ranges from 0 (knob left) to 1(knob right). We use this 0-1 value to calculate the LFO rate (by multiplying this with the MAX lfo rate we get a value from 0 to the max LFO rate, and we also use this to calculate the max frequency deviation similarly.


When version 2 is released for the minilogue xd (or when the minilogue xd supports the API that enables shift-time and shift-depth), we will add the ability to change the centre frequency and the resonance via these parameters

*Improvements for next time: Add a slew to the filter frequency to prevent sudden sharp frequency changes. Oversampling - oversampling would definitely be recommended, and is super easy to implement. This would also greatly increase the upper frequency range of the filter as well, as this filter type does not do well when you get close to 1/6 the sample rate. As well, as it is, it's not immune from "blowing up" - some simple safety checks / restrictions would help prevent this. Code optimizations. You can eliminate the D1/D2 variables in the filter easily to save some CPU time.*

*Currently, the software available on this site is all offered for free, with an optional donation link for each if you like. All software is provided as-is, with no guarantees whatsoever - and is to be used entirely at your own risk.*

*© All rights reserved. Any trademarks used are property of their respective owners*

*Be sure to read the included license with each software download!*
