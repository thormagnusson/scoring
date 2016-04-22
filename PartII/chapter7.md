# Modulation - AM, RM and FM synthesis

Modulating one signal with another is one of the oldest and most common techniques in sound synthesis. Here, any parameter of an oscillator can be modulated by the output of another oscillator. Filters, PlayBufs (sound file players) and other things can also be modulated. In this chapter we will explore modulation, and in particular amplitude modulation (AM), ring modulation (RM) and frequency modulation (FM).

## LFOs (Low Frequency Oscillators)

As mentioned most parameters or controls in an oscillator can be controlled by the output of another. Low frequency oscillators (LFOs) are oscillators that typically operate under 20 Hz, although in SuperCollider there is no point in trying to define oscillators as LFOs, as we might always want to increase that frequency to 40 or 400 Hz!

Here below are examples of a triangle wave that has different controls modulated by another UGen.

In the first example we have the frequency of one oscillator modulated by the output (amplitude) of another:

{line-numbers=off}
~~~~~~~
{ SinOsc.ar( 440 * SinOsc.ar(1), 0, 0.4) }.play
~~~~~~~

We hear that the modulation is 2 Hz, not one, and that is because the output of the modulating oscillator goes up to 1 and down to -1 in one second. So for a one cycle of modulation per second, you would have to give it 0.5 as an amplitude. Furthermore, a frequency argument with a negative sign is automatically turned into a positive one, as negative frequency does not make sense.

Let's try the same for amplitude:

{line-numbers=off}
~~~~~~~
{ SinOsc.ar( 440, 0, 0.4 * SinOsc.ar(1)) }.play
// or perhaps using LFPulse (which outputs 1 and 0s if the amp is 1)
{ SinOsc.ar( 440, 0, 0.4 * LFPulse.ar(2)) }.play
~~~~~~~

We thus get the familiar effects of vibrato (modulation of frequency) and tremolo (modulation of amplitude) as they are commonly defined as:

{line-numbers=off}
~~~~~~~
// vibrato
{SinOsc.ar(440+SinOsc.ar(4, 0, 10), 0, 0.4) }.play
// tremolo
{SinOsc.ar(440, 0, SinOsc.ar(3, 0, 1)) }.play
~~~~~~~

In modulation synthesis we talk about a "modulator" (the oscillator that does the modulation) and the "carrier" which is the main signal being modulated.

{line-numbers=off}
~~~~~~~
// mouseX is the power of the vibrato
// mouseY is the frequency of the vibrato
{
	var modulator, carrier;
	modulator = SinOsc.ar(MouseY.kr(20, 5), 0, MouseX.kr(5, 20)); 
	carrier = SinOsc.ar(440 + modulator, 0, 1);
	carrier ! 2 // the output
}.play
~~~~~~~

There are special Low Frequency Oscillators (LFOs) in SuperCollider. They are typically not band limited, which means that they start to alias (or mirror back) into the frequency domain. Consider the difference between Saw (band-limited) and LFSaw (non-band-limited) here:

{line-numbers=off}
~~~~~~~
{Saw.ar(MouseX.kr(100, 10000), 0.5)}.freqscope
{LFSaw.ar(MouseX.kr(100, 10000), 0.5)}.freqscope
~~~~~~~

When you move your mouse, you can see how the band-limited Saw only gives you the harmonics above the fundamental frequency set by the mouse. On the other hand, with LFSaw, you get the harmonics mirroring back into the audible range at the Nyquist frequency (half the sampling rate, very often 22.050Hz).

But the LFUgens are good for modulation and we typically can run them in the control rate (using .kr rather than .ar - which is typically 64 times less calculation per second -> that is, if the block size is set to 64 samples)

{line-numbers=off}
~~~~~~~
// LFSaw
{ SinOsc.ar(LFSaw.kr(4, 0, 200, 400), 0, 0.7) }.play

// LFTri
{ SinOsc.ar(LFTri.kr(4, 0, 200, 400), 0, 0.7) }.play
{ Saw.ar(LFTri.kr(4, 0, 200, 400), 0.7) }.play

// LFPar
{ SinOsc.ar(LFPar.kr(0.2, 0, 400,800),0, 0.7) }.play

// LFCub
{ SinOsc.ar(LFCub.kr(0.2, 0, 400,800),0, 0.7) }.play

// LFPulse
{ SinOsc.ar(LFPulse.kr(3, 1, 0.3, 200, 200),0, 0.7) }.play
{ SinOsc.ar(LFPulse.kr(3, 1, 0.3, 2000, 200),0, 0.7) }.play

// LFOs can also perform at audio rate
{ LFPulse.ar(LFPulse.kr(3, 1, 0.3, 200, 200),0, 0.7) }.play
{ LFSaw.ar(LFSaw.kr(4, 0, 200, 400), 0, 0.7) }.play
{ LFTri.ar(LFTri.kr(4, 0, 200, 400), 0, 0.7) }.play
{ LFTri.ar(LFSaw.kr(4, 0, 200, 800), 0, 0.7) }.play
~~~~~~~

Finally, we should note here at the end of this section on LFOs that the LFO frequency can of course go as high as you would like, but then it ceases being an LFO and starts to do different type of synthesis, which we will look at below. In the examples here, you will start to hear strange artefacts arriving when the oscillation goes up over 20 Hz (observe the post window).

{line-numbers=off}
~~~~~~~
{SinOsc.ar(440+SinOsc.ar(XLine.ar(4, 200, 10).poll(20, "mod freq:"), 0, 20), 0, 0.4) }.play
{SinOsc.ar(440, 0, SinOsc.ar(XLine.ar(4, 200, 10).poll(20, "mod freq:"), 0, 1)) }.play
~~~~~~~


## Theremin

We have now obviously found the technique to create a Theremin using vibrato and tremolo:

{line-numbers=off}
~~~~~~~
// Using the MouseX to control amplitude
	{
		var f;
		f = MouseY.kr(4000, 200, 'exponential', 0.8);
		SinOsc.ar(
			freq: f+ (f*SinOsc.ar(7,0,0.02)),
			mul: MouseX.kr(0, 0.9)
		)
	}.play

// Using the MouseX to control vibrato speed
	{
		var f;
		f = MouseY.kr(4000, 200, 'exponential', 0.8);
		SinOsc.ar(
			freq: f+ (f*SinOsc.ar(3+MouseX.kr(1, 6),0,0.02)),
			mul: 0.3
		)
	}.play
~~~~~~~




## Amplitude Modulation (AM synthesis)

In one of the examples above, the XLine Ugen to the LFO frequency up over 20Hz and we started to get some exciting artefacts in the sound. What was happening was that "sidebands" were appearing, i.e., partials on either side of the sine. Amplitude synthesis is a modulation that modulates the carrier with unipolar values (that is, they are between 0 and 1 - not bipolar (-1 to 1)). 

In amplitude modulation, the sidebands are the sum and the difference of the carrier and the modulator frequency. For example, a 300 Hz carrier and 160 Hz modulator would generate 140 Hz and 460 Hz sidebands. However, the carrier frequency is always present.

{line-numbers=off}
~~~~~~~
{
	var modulator, carrier;
	modulator = SinOsc.ar(MouseX.kr(2, 20000, 1), 0, mul:0.5, add:1);
	carrier = SinOsc.ar(MouseY.kr(300,2000), 0, modulator);
	carrier ! 2;
}.play
~~~~~~~


If there are harmonics in the wave being modulated, each of the harmonics will have sidebands as well. - Check the saw wave.

{line-numbers=off}
~~~~~~~
{
	var modulator, carrier;
	modulator = SinOsc.ar(MouseX.kr(2, 2000, 1), mul:0.5, add:1);
	carrier = Saw.ar(533, modulator);
	carrier ! 2 // the output
}.play
~~~~~~~

In digital synthesis we can apply all kinds of mathematical operators to the sound, for example using .abs to calculate absolute values in the modulator. (this results in many sidebands - try also using .cubed and other unitary operators on the signal).

{line-numbers=off}
~~~~~~~
{
	var modulator, carrier;
	modulator = SinOsc.ar(MouseX.kr(2, 20000, 1)).abs;
	carrier = SinOsc.ar(MouseY.kr(200,2000), 0, modulator);
	carrier!2 // the output
}.play
~~~~~~~


## Ring Modulation

As mentioned above, ring modulation uses a bipolar modulation values (-1 to 1) whereas AM uses unipolar modulation values (0 to 1). This results in ordinary amplitude modulation outputting the original carrier frequency as well as the two side bands for each of the spectral components of the carrier and modulation signals. Ring modulation, however, cancels out the carrier frequencies and simply outputs the side-bands.

{line-numbers=off}
~~~~~~~
{
	var modulator, carrier;
	modulator = SinOsc.ar(MouseX.kr(2, 200, 1));
	carrier = SinOsc.ar(333, 0, modulator);
	carrier!2;
}.play
~~~~~~~

Ring modulation was used much in the early electronic music studios, for example in Cologne, BBC Radiophonic workshop and so on. The Barrons used the technique in the music for Forbidden Planet and so did Stockhausen in his Microphonie II, where voices are modulated with the sound of an Hammond organ. Let's try to ring modulate a voice:

{line-numbers=off}
~~~~~~~
b = Buffer.read(s, Platform.resourceDir +/+ "sounds/a11wlk01.wav");
{
	var modulator, carrier;
	modulator = SinOsc.ar(MouseX.kr(20, 200, 1));
	carrier = PlayBuf.ar(1, b, 1, loop:1) * modulator;
	carrier ! 2;
}.play;
~~~~~~~

Here a sine wave is modulating the voice of a girl saying "Columbia this is Houston, over...". We could use one sound file to ring modulate the output of another:

{line-numbers=off}
~~~~~~~
b = Buffer.read(s, Platform.resourceDir +/+ "sounds/a11wlk01.wav");
c = Buffer.read(s, "yourSound.wav");
c.play
{
	var modulator, carrier;
	modulator = PlayBuf.ar(1, c, 1, loop:1);
	carrier = PlayBuf.ar(1, b, 1, loop:1) * modulator;
	carrier ! 2;
}.play;
~~~~~~~


## Frequency Modulation (FM Synthesis)

FM Synthesis is a popular synthesis technique that works well for a number of sounds. It became popular with the Yamaha DX7 Synthesizer in the late 1980s, but it was invented in 197XXX when John Chowning, musician and researcher at Stanford University, discovered the power of FM synthesis. He was working in the lab one day when he accidentally plugged the output of one oscillator into the frequency input of another and he heard a sound rich with partials (or sidebands as we call them in modulation synthesis). It's important to realise that at the time, an oscillator was expensive equipment, and the possibility of getting so many partials out of only two oscillators was very exciting in musical, engineering, and economical terms.

Chowning's famous FM synthesis piece is called Stria and can be found on the interwebs. The piece was an eye opener for many musicians, as its sounds were so unusual in timbre, rendering the texture of the piece surprising and novel. Imagine being there at the time and hearing these "unnatural" sounds for the first time! 

1980s synth pop music is of course full with the sounds of FM synthesis, but musicians were typically using the DX7, but very often using the pre-installed sounds of the synth itself rather than making their own. The reason for this could be that FM synthesis is quite hard to learn, as there are so multiple parameters at play in any sound. The story is that the user interface of the DX7 prevented people from designing sounds in an effective and ergonomic way, thus the lack of new and exploratory sound design using that synth.

{line-numbers=off}
~~~~~~~
{SinOsc.ar(1400 + SinOsc.ar(MouseX.kr(2,2000,1), 0, MouseY.kr(1,1000)), 0, 0.5)!2}.freqscope
~~~~~~~

Using the frequency scope in the example above, you will see that when you move your mouse around, sidebands are appearing, spreading with even distance to each other, and the more amplitude the modulator has, the more sidebands you get. Let's explore the above example with comments, in order to get the terminology right:

{line-numbers=off}
~~~~~~~
// the same as above - with explanations:
{
SinOsc.ar(2000 	// the carrier and the carrier frequency
	+ SinOsc.ar(MouseX.kr(2,2000,1),  // the modulator and the modulator frequency
		0, 					  // the phase of the modulator
		MouseY.kr(1,1000) 		  // the modulation depth (index)
		), 
0,		// the carrier phase 
0.5)	// the carrier amplitude
}.play
~~~~~~~

What is happening is that we have a carrier oscillator (the first SinOsc) with a frequency of 2000 Hz. We then *add* to this frequency the output of another oscillator. Note that the amplitude of the modulator is very high: it goes up to 1000, which would become uncomfortable for your ears were you to play that on its own. So when you move the mouse across the x-axis, you notice that around the carrier frequency partial (of 2000Hz) there are appearing sidebands with the distance of the modulator frequency. That is, if the modulator frequency is 250 Hz, you get sidebands of 1750 and 2250; 1500 and 2500; 1250 and 2750, etc. The stronger the modulation depth, or the index, of the modulator (its amplitude basically), the louder the sidebands will become.

We could of course create all those sidebands with oscillators in an additive synthesis style, but note the efficiency of FM compared to Additive synthesis:

{line-numbers=off}
~~~~~~~
// FM
{PMOsc.ar(1000, 800, 12, mul: EnvGen.kr(Env.perc(0, 0.5), Impulse.kr(1)))}.play;
 // compared with additive synthesis:
{ 
Mix.ar( 
 SinOsc.ar((1000 + (800 * (-20..20))),  // we're generating 41 oscillators (see *)
  mul: 0.1*EnvGen.kr(Env.perc(0, 0.5), Impulse.kr(1))) 
)}.play 
 
~~~~~~~

TIP:

// * run this line : 
(1000 + (1000 * (-20..20)))
// and see the frequency array that is mixed down with Mix.ar
// (I think this is an example from David Cope)


Below are two patches that serve well to explore the power of simple FM synthesis. In the first one, a LFNoise0 UGen is used to trigger a new number between 20 and 60, 4 times per second. This number will be a floating point number (a fractional number) so it is rounded to an integer. Then the number is turned into frequency values using .midicps (where MIDI note value is turned into a value of cycles per second).

{line-numbers=off}
~~~~~~~
{ var freq, ratio, modulator, carrier;
freq = LFNoise0.kr(4, 20, 60).round(1).midicps; 
ratio = MouseX.kr(1,4); 
modulator = SinOsc.ar(freq * ratio, 0, MouseY.kr(0.1,10));
carrier = SinOsc.ar(freq + (modulator * freq), 0, 0.5);
carrier	
}.play

// let's fork it and create a perc Env!
{	
	40.do({
			{ var freq, ratio, modulator, carrier;
			freq = rrand(60, 72).midicps; 
			ratio = MouseX.kr(0.5,2); 
			modulator = SinOsc.ar(freq * ratio, 0, MouseY.kr(0.1,10));
			carrier = SinOsc.ar(freq + (modulator * freq), 0, 0.5);
			carrier * EnvGen.ar(Env.perc(0, 1), doneAction:2)
		}.play;
		0.5.wait;
	});
}.fork
~~~~~~~


### The PMOsc - Phase modulation

Frequency modulation and phase modulation are pretty much the same. In SuperCollider we have a PMOsc (Phase Modulation Oscillator), and we can try to make the above example using that:

{line-numbers=off}
~~~~~~~
{PMOsc.ar(1400, MouseX.kr(2,2000,1), MouseY.kr(0,1), 0)!2}.freqscope
~~~~~~~

You will note a feature in phase modulation, in that when the modulating frequency is low (< 20Hz), you don't get the vibrato-like effect of the frequency modulation synth.

The magic of the PMOsc can be studied if we look under the hood. PMOsc is a pseudo-UGen, i.e., it is not written in C and compiled as a plugin for the SC-server, but rather defined when the class library of SuperCollider is compiled (on startup or if you hit Cmd+K XXX) 

How does the PMOsc work? Let's check the source file (Cmd+j or Ctrl+j). You will see that the PMOsc.ar method simply returns (with the ^ symbol) a SinOsc with another SinOsc in the phase argument slot.

{line-numbers=off}
~~~~~~~
PMOsc  {
	*ar { arg carfreq,modfreq,pmindex=0.0,modphase=0.0,mul=1.0,add=0.0; 
		^SinOsc.ar(carfreq, SinOsc.ar(modfreq, modphase, pmindex),mul,add)
	}	
	*kr { arg carfreq,modfreq,pmindex=0.0,modphase=0.0,mul=1.0,add=0.0; 
		^SinOsc.kr(carfreq, SinOsc.kr(modfreq, modphase, pmindex),mul,add)
	}
}
~~~~~~~

Here are a few examples for studying the PM oscillator:

{line-numbers=off}
~~~~~~~
{ PMOsc.ar(MouseX.kr(500,2000), 600, 3, 0, 0.1) }.play; // modulate carfreq
{ PMOsc.ar(2000, MouseX.kr(200,1500), 3, 0, 0.1) }.play; // modulate modfreq
{ PMOsc.ar(2000, 500, MouseX.kr(0,10), 0, 0.1) }.play; // modulate index
~~~~~~~

The SuperCollider documentation of the UGen presents a nice demonstration of the UGen that looks a bit like this:

{line-numbers=off}
~~~~~~~
e = Env.linen(2, 5, 2);
fork{
    inf.do({
        { LinPan2.ar(EnvGen.ar(e) 
			*
			PMOsc.ar(2000.0.rand,800.0.rand, Line.kr(0, 12.0.rand,9),0,0.1), 
			1.0.rand2)
			}.play;
        2.wait;
    })
}
~~~~~~~

Other examples of PM synthesis:

{line-numbers=off}
~~~~~~~
{ var freq, ratio;
freq = LFNoise0.kr(4, 20, 60).round(1).midicps; 
ratio = MouseX.kr(1,4); 
SinOsc.ar(freq, 				// the carrier and the carrier frequency
		SinOsc.ar(freq * ratio, 	// the modulator and the modulator frequency
		0, 					// the phase of the modulator
		MouseY.kr(0.1,10) 		// the modulation depth (index)
		), 
0.5)		// the carrier amplitude
}.play
~~~~~~~

Same patch without the comments and modulator and carrier put into variables:

{line-numbers=off}
~~~~~~~
{ var freq, ratio, modulator, carrier;
	freq = LFNoise0.kr(4, 20, 60).round(1).midicps; 
	ratio = MouseX.kr(1,4); 
	modulator = SinOsc.ar(freq * ratio, 0, MouseY.kr(0.1,10));
	carrier = SinOsc.ar(freq, modulator, 0.5);
	carrier	
}.play
~~~~~~~

## The use of Envelopes in FM synthesis

Frequency modulation is a complex technique and Chowning's initial research paper shows a wide range of applications of this synthesis method. For example, in the patch below, we have a much lower modulation amplitude (between 0 and 1) but we multiply the carrier frequency with the modulator. 

{line-numbers=off}
~~~~~~~
(
var carrier, carFreq, carAmp, modulator, modFreq, modAmp; 
carFreq = 2000; 
carAmp = 0.2;		
modFreq = 327; 
modAmp = 0.2; 
{
	modAmp = MouseX.kr(0, 1); 	// choose normalized range for modulation
	modFreq = MouseY.kr(10, 1000, 'exponential');
	modulator = SinOsc.ar( modFreq, 0, modAmp);			
	carrier = SinOsc.ar( carFreq + (modulator * carFreq), 0, carAmp);
	[ carrier, carrier, modulator ]
}.play
)
~~~~~~~

And we can compare that technique with our initial FM example. In short, the frequency of the carrier is used as a parameter in the index (amplitude) of the modulator. These are design details and there are multiple ways of using FM synthesis to derive at the sound that you are after.

{line-numbers=off}
~~~~~~~
// current technique 
{ SinOsc.ar( 1400 + (SinOsc.ar( MouseY.kr(10, 1000, 1), 0, MouseX.kr(0, 1)) * 1400), 0, 0.5) ! 2 }.play
// our first example
{ SinOsc.ar(1400 + SinOsc.ar(MouseY.kr(10, 1000,1), 0, MouseX.kr(1,1000)), 0, 0.5) ! 2 }.play
~~~~~~~

One of the key techniques in FM synthesis is to use envelopes do control the parameters in the modulator. By changing the width and amplitude of the sidebands, we can get many interesting sounds, for example trumpets, mallets or bells.

Let us first create a basic FM synthesis synth definition and try to play it with diverse arguments:
 
{line-numbers=off}
~~~~~~~
SynthDef(\fmsynth, {arg outbus = 0, freq=440, carPartial=1, modPartial=1, index=3, mul=0.2, ts=1;
	var mod, car, env;
	// modulator frequency
	mod = SinOsc.ar(freq * modPartial, 0, freq * index );
	// carrier frequency
	car = SinOsc.ar((freq * carPartial) + mod, 0, mul );
	// envelope
	env = EnvGen.ar( Env.perc(0.01, 1), doneAction: 2, timeScale: ts);
	Out.ar( outbus, car * env)
}).add;

Synth(\fmsynth, [ \outbus, 0, \freq, 600.0, \carPartial, 1.5, \ts, 1]);
Synth(\fmsynth, [ \outbus, 0, \freq, 600.0, \carPartial, 2.5, \ts, 2]);
Synth(\fmsynth, [ \outbus, 0, \freq, 600.0, \carPartial, 3.5, \ts, 2]);
Synth(\fmsynth, [ \outbus, 0, \freq, 600.0, \carPartial, 4.0, \ts, 2]);
Synth(\fmsynth, [ \outbus, 0, \freq, 300.0, \carPartial, 1.5, \ts, 2]);
Synth(\fmsynth, [ \outbus, 0, \freq, 600.0, \carPartial, 0.5, \ts, 2]);

Synth(\fmsynth, [ \outbus, 0, \freq, 600.0, \carPartial, 1.5, \modPartial, 1, \ts, 2]);
Synth(\fmsynth, [ \outbus, 0, \freq, 300.0, \carPartial, 1.5, \modPartial, 1, \ts, 2]);
Synth(\fmsynth, [ \outbus, 0, \freq, 400.0, \carPartial, 1.5, \modPartial, 1, \ts, 2]);
Synth(\fmsynth, [ \outbus, 0, \freq, 800.0, \carPartial, 1.5, \modPartial, 1, \ts, 2]);

Synth(\fmsynth, [ \outbus, 0, \freq, 600.0, \carPartial, 1.5, \modPartial, 1, \ts, 2]);
Synth(\fmsynth, [ \outbus, 0, \freq, 600.0, \carPartial, 1.5, \modPartial, 1.1, \ts, 2]);
Synth(\fmsynth, [ \outbus, 0, \freq, 600.0, \carPartial, 1.5, \modPartial, 1.15, \ts, 2]);
Synth(\fmsynth, [ \outbus, 0, \freq, 600.0, \carPartial, 1.5, \modPartial, 1.2, \ts, 2]);
~~~~~~~

T>## FM7 Ugen 
T>
T> Check the FM7 UGen that is part of the SC3-plugins distribution. 

{line-numbers=off}
~~~~~~~
SynthDef(\fmsynthenv, {arg outbus = 0, freq=440, carPartial=1, modPartial=1, index=3, mul=0.2, ts=1;
	var mod, car, env;
	var modfreqenv, modindexenv;
	modfreqenv = EnvGen.kr(Env.perc(0.1, ts/10, 0.125))+1; // add 1 so we're not starting from zero
	modindexenv = EnvGen.kr(Env.sine(ts, 1))+1;
	mod = SinOsc.ar(freq * modPartial * modfreqenv, 0, freq * index * modindexenv);
	car = SinOsc.ar((freq * carPartial) + mod, 0, mul );
	env = EnvGen.ar( Env.perc(0.01, 1), doneAction: 2, timeScale: ts);
	Out.ar( outbus, Pan2.ar(car * env))
}).add;

Synth(\fmsynthenv, [ \freq, 440.0, \ts, 10]);
Synth(\fmsynthenv, [ \freq, 440.0, \ts, 1]);
Synth(\fmsynthenv, [ \freq, 110.0, \ts, 2]);
~~~~~~~
