# Time Domain Effects

In this book, we divide the section on audio effects into two separate chapters, on time domain and frequency domain effects, respectively. This is for a good reason as the two are completely different techniques of manipulating audio, where the former, the time domain effects, are well know from the world of analogue audio, whereas the latter, manipulation in the frequency domain, is only realistically possible through the use of computers running Fast Fourier Transformation (FFT) algorithms. This will be explained later.

Most of the audio effects that we know (and you can roughly think about the availability of guitar pedal boxes, where each box contains the implementation of some audio effect) are familiar and easy to understand effects that were often *discovered* by accident or invented through some form of serendipitous exploration. There are diverse stories of John Lennon and George Martin discovering flanging on an Abbey Road tape machine, but earlier examples exist, although the technique had not been given this name then. 
The time domain effects are either manipulation of samples in *time* (typically where the signal is split and something is done to one of them, such as delaying it, and they then added again) or in *amplitude* (where samples can be changed in value, for example to get a distortion effect). This chapter will explore the diverse audio effects that can be easily created using the UGens available in SuperCollider.

## Delay

When we delay a signal, we can achieve various effects, from a simple echo to a more complex reverb. Typical variables are delay time (how long it takes before the sound appears again) and decay time (how long it will repeat). In SuperCollider, there are three main type of delays: Delay, Comb and Allpass:

- DelayN/DelayL/DelayC are simple echos with no feedback. 
- CombN/CombL/CombC are comb delays with feedback (decaytime)
- AllpassN/AllpassL/AllpassC die out faster than the comb, but have feedback as well

All of these delays come with different interpolation algorithms (N, L, C, standing for No interpolation, Linear interpolation, and Cubic interpolation). Interpolation is about what happens between two discrete values, for example samples. Will you get a jump when the next value appears (N), a line from one value to the next (L) or a curvy shape between the two (C), simulating better analogue signal behaviour. These are all good for different purposes, where the N is computationally cheap, but C is good if you are sweeping delay time and you want more nuanced interpolation that can deal with values between two samples.

Generally, we can talk about three types of time when using Delays, resulting in different types of effects:

	Short ( < 10 ms)
	Medium ( 10 - 50 ms)
	Long ( > 50 ms)

A short delay (1-2 samples) can create a FIR (Finite Impulse Response) lowpass filter.
Increase the delay time (1-10 ms) and a comb filter materialises.
Medium delays result in a thin signal but could also an ambience and width in the sound.
Long delays create discrete echo which imitates sound bouncing of hard walls.

Delays can also have variable delay time which can result in the following effects:
	Phase Shifting
	Flanging
	Chorus
These effects are explained in dedicated sections here below 


### Short Delays (< 10 ms)

Let's explore what a short delay means. This is a delay that's hardly perceivable by the human ear if you would for example delay a click sound or an impulse.

{
	x = Impulse.ar(1);
	d =  DelayN.ar(x, 0.001,  MouseX.kr(s.sampleRate.reciprocal, 0.001).poll);
	(x+d)!2
}.play 

In the example above we have a delay from from a sample (e.g., 44100.reciprocal, or 0.000022675 seconds, or 0.023 ms) to 10 milliseconds. The impulse is the shortest sound possible (one sample of of 1 in amplitude), so it serves well in this experiment. When you move the mouse from the left to the right of the screen you will probably perceive the sound as one event, but you will notice that the sound changes slightly in timbre. It is filtered. And indeed, as we will see in the filter chapter, most filters work by way of delaying samples and multiplying the feedback or feedforward samples by different values. 
We could try the same with a more continuous signal, for example a Saw wave. You will hear that the timbre of the wave changes when you move the mouse around, as it is effectively being filtered (adding two signals together where one is slightly delayed)

{
	x = Saw.ar(440, 0.4);
	d =  DelayC.ar(x, 0.001,  MouseX.kr(s.sampleRate.reciprocal, 0.001).poll);
	(x+d)!2
}.play

Note that in the example above I'm using DelayC, as opposed to the DelayN in the Impulse code. This is because the delay time is so small, at sample level, that interpolation becomes important. Try to change the DelayC to DelayN (no interpolation) and listen to what happens, particularly when moving the mouse at the left of the screen at shorter delay times. The best way to explore the filtering effect might be to use WhiteNoise:

{
	x = WhiteNoise.ar(0.1);
	d =  DelayN.ar(x, 0.001,  MouseX.kr(s.sampleRate.reciprocal, 0.001));
	(x+d)!2
}.play

In the examples above we have been adding the two signals together (the original and the delayed signal) and then duplicating it (!2) into two arrays, for a two-speaker output. Adding the signals create the filtering effect, but if we simply put each signal in each speaker, we get a completely different effect, namely spatialisation:

{
	x = WhiteNoise.ar(0.1);
	d =  DelayC.ar(x, 0.006,  MouseX.kr(s.sampleRate.reciprocal, 0.006));
	[x, d]
}.play

We have now entered the realm of psychoacoustics, but this can be explained quickly by the fact that sound travels around 343 metres per second, or 34cm per millisecond, roughly 0.6 millisecond difference in arrival to the ears of a normal head, if the sound is coming from one side direction. This is called Interaural Time Difference (ITD) and is one of they key factors for sound localisation. We could explore this in the following example, where we have a signal that is "delayed" from 0.001 ms before to 0.001 ms after the original signal. Try this with headphones, you should get some impression of sound moving from the left to the right ear.

{
	x = Impulse.ar(1);
	l =  DelayC.ar(x, 1.001,  1+MouseX.kr(-0.001, 0.001));
	r =  DelayC.ar(x, 1.001,  1+MouseX.kr(0.001, -0.001));
	[l, r] // left and right channels
}.play

// load some sound files into buffers (use your own)
d = Buffer.read(s,"sounds/digireedoo.aif");
e = Buffer.read(s,"sounds/holeMONO.aif");
e = Buffer.read(s, "sounds/a11wlk01.wav"); // this one is in the SC sounds folder

In the example below, explore the difference algorithms implemented in Delay, Comb and Allpass. The Delay does not have the decay time, therefore not resulting in the Karplus-Strong type of sound that we get with the other two. The details of the difference in the internal implementation of Comb and Allpass are too complex for this book, but it has to do with the how gain coefficients are calculated, where a combined feedback and feedforward combs equal an allpass. 

{
	var signal, delaytime = MouseX.kr(0.00022675, 0.01, 1);
	signal = PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1);
	// signal = Saw.ar(440,0.3);
	// signal = WhiteNoise.ar(0.3);
	d =  DelayC.ar(signal, 0.6, delaytime);
	// d =  AllpassC.ar(signal, 0.6, delaytime, MouseY.kr(0.001,1, 1));
	// d =  CombC.ar(signal, 0.6, delaytime, MouseY.kr(0.001,1, 1));
	(signal + d).dup
}.play

Is this familiar?

{CombC.ar(SoundIn.ar(0), 0.6, LFPulse.ar(0.25).range(0.0094,0.013),  0.9)!2}.play

{
	var signal, delay, delaytime = MouseX.kr(0.00022675, 0.02, 1);
	signal = PlayBuf.ar(1, e, 1, loop:1);
	delay =  DelayC.ar(signal, 0.2, delaytime);
	[signal, delay]
}.play


Any amount of Delays can be added together to create the desired sound of course, something we will explore when we discuss reverbs: 

{
	var signal;
	var delaytime = MouseX.kr(0.1,0.4, 1);
	signal = Impulse.ar(1);	
	Mix.fill(14, {arg i; DelayL.ar(signal, 1, delaytime*(1+(i/10))) });
}.play


The old Karplus-Strong in its most basic form:

{
	var delaytime = MouseX.kr(0.001,0.2, 1);
	var decaytime = MouseY.kr(0.1,2, 1);
	var signal = Impulse.ar(1);
	CombL.ar(signal, 0.6, delaytime, decaytime)!2
}.play



### Medium Delay time ( 10 - 50 ms)

The examples above with delays under 10ms, resulted in change in timbre or spatial location, but we always felt that this was the same sonic event, even when using a one-sample impulse. It is dependent on subjects and context, but it can be said that we start to perceive a delayed event as two events if there is more than 20 ms delay between them. This code demonstrates that:

{x=Impulse.ar(1); y=DelayC.ar(x, 0.04, MouseX.kr(0.005, 0.04).poll); (x+y)!2}.play

The post window shows the milliseconds. A drummer who would be more than 20 ms off when trying to be on the exact beat would be showing a disappointing performance (of course, part of the art of a good percussionist is to be slightly ahead or behind, so the comment is not about intention) and any hardware interface that would have a latency of more than 20 ms would be considered rather poor interface.

Longer delays can also generate a *spatialisation* effect, although this is not modelling the interaural time difference (ITD), but rather creating the sensation of a wide sonic image. 

e = Buffer.read(s,"sounds/holeMONO.aif");

{
	var signal, delay, delaytime = MouseX.kr(0.00022675, 0.05, 1).poll;
	signal = PlayBuf.ar(1, e, 1, loop:1);
	delay =  DelayC.ar(signal, 0.2, delaytime);
	[signal, delay]
}.play
// Using microphone input
{
	var signal, delay, delaytime = MouseX.kr(0.00022675, 0.05, 1).poll;
	signal = SoundIn.ar(0);
	delay =  DelayC.ar(signal, 0.2, delaytime);
	[signal, delay]
}.play


### Longer Delays ( > 50 ms)


(
{
var signal;
var delaytime = MouseX.kr(0.05, 2, 1); // between 50 ms and 2 seconds - exponential.
signal = PlayBuf.ar(1, f.bufnum, BufRateScale.kr(f.bufnum), loop:1);

// compare DelayL, CombL and AllpassL

//d =  DelayL.ar(signal, 0.6, delaytime);
//d = CombL.ar(signal, 0.6, delaytime, MouseY.kr(0.1, 10, 1)); // decay using mouseY
d =  AllpassL.ar(signal, 0.6, delaytime, MouseY.kr(0.1,10, 1));

(signal+d).dup
}.play(s)
)



// same as above, here using AudioIn for the signal instead of the NASA irritation
(
{
var signal;
var delaytime = MouseX.kr(0.05, 2, 1); // between 50 ms and 2 seconds - exponential.
signal = AudioIn.ar(1);

// compare DelayL, CombL and AllpassL

//d =  DelayL.ar(signal, 0.6, delaytime);
//d = CombL.ar(signal, 0.6, delaytime, MouseY.kr(0.1, 10, 1)); // decay using mouseY
d =  AllpassL.ar(signal, 0.6, delaytime, MouseY.kr(0.1,10, 1));

(signal+d).dup
}.play(s)
)

### Random experiments


//
Server.default = s = Server.internal
FreqScope.new;
{CombL.ar(Impulse.ar(10), 6, 1, 1)}.play(s)


(
{
var signal;
var delaytime = MouseX.kr(0.01,6, 1);
var decaytime = MouseY.kr(1,2, 1);

signal = Impulse.ar(1);

d =  CombL.ar(signal, 6, delaytime, decaytime);

d!2
}.play(s)
)


// we can see the Comb effect by plotting the signal.

(
{
a = Impulse.ar(1);
d =  CombL.ar(a, 1, 0.001, 0.9);
d
}.plot(0.1)
)



// a little play with AudioIn
(
{
var signal;
var delaytime = MouseX.kr(0.001,2, 1);
signal = AudioIn.ar(1);

a = Mix.fill(10, {arg i; var dt;
		dt = delaytime*(i/10+0.1).postln;
		DelayL.ar(signal, 3.2, dt);});

(signal+a).dup
}.play(s)
)

/*
TIP: if you get this line printed ad infinitum:
exception in real time: alloc failed
You could go into the ServerOptions.sc (source file) and change
	var <>memSize = 8192;
to
	var <>memSize = 32768;
which allows the server to use up more memory (RAM)
*/



(
{ // watch your ears !!! Use headphones and lower the volume !!!
var signal;
var delaytime = MouseX.kr(0.001,2, 1);
signal = AudioIn.ar(1);

a = Mix.fill(13, {arg i; var dt;
		dt = delaytime*(i/10+0.1).postln;
		CombL.ar(signal, 3.2, dt);});

(signal+a).dup
}.play(s)
)


// A source code for a Comb filter might look something like this:
int  i, j, s;

for(i=0; i <= delay_size;i++)

  { if (i >= delay)
     j = i - delay;    // work out the buffer position
    else 
    j = i - delay + delay_size + 1;
    // add the delayed sample to the input sample
    s = input + delay_buffer[j]*decay;
    // store the result in the delay buffer, and output
    delay_buffer[i] = s;
    output = s;
  } 
  

## Phaser (phase shifting)

In a phaser, a signal is sent through an allpass filter, not filtering out any frequencies, but simply shifting the phase of the sound by delaying it. This sound is then added to the original signal. If the phase is 180 degrees, the sound is cancelled out, but if it is less than that, it will create variations in the spectra.

// phaser with a soundfile
e = Buffer.read(s, "sounds/a11wlk01.wav");

(
{
var signal;
var phase = MouseX.kr(0.000022675,0.01, 1); // from a sample resolution to 10 ms delay line

var ph;

signal = PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1);

ph = AllpassL.ar(PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1), 4, phase+(0.01.rand), 0);
/* // try 4 phasers
ph = Mix.ar(Array.fill(4, 
			{ AllpassL.ar(PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1), 4, phase+(0.01.rand), 0)}
		));
*/

(signal + ph).dup 
}.play
)


// try it with a sinewave (the mouse is shifting the phase of the input signal
(
{
var signal;
var phase = MouseX.kr(0.000022675,0.01); // from a sample to 10 ms delay line
var ph;

signal = SinOsc.ar(444,0,0.5);
//signal = PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1);
ph = AllpassL.ar(SinOsc.ar(444,0,0.5), 4, phase, 0);

 (signal + ph).dup 

}.play
)


// using an oscillator to control the phase instead of MouseX
// here using the .range trick:
{SinOsc.ar(SinOsc.ar(0.3).range(440, 660), 0, 0.5) }.play

(
{
var signal;
var ph;

// base signal
signal = PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1);
// phased signal
ph = AllpassC.ar(
		PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1), 
		4, 
		LFPar.kr(0.1, 0, 1).range(0.000022675,0.01), // a circle every 10 seconds 
		0); // experiment with what happens if you increase the decay length

 (signal + ph).dup // we add them together and route to two speakers
}.play
)

/*
NOTE: Theoretically you could use DelayC or CombC instead of AllpassC.
In the case of DelayC, you would have to delete the last argument (0) 
(as DelayC doesn't have decay argument).
*/









## Flanger

In a Flanger, a delayed signal is added to the original signal with a continuously-variable delay (usually smaller than 10 ms) creating a phasing effect. The term comes from times where tapes were used in studios and an operator would place the finger on the flange of one of the tapes to slow it down, thus causing the flanging effect.

Flanger is like a Phaser with dynamic delay filter (allpass), but it usually has a feedback loop.


(
SynthDef(\flanger, { arg out=0, in=0, delay=0.1, depth=0.08, rate=0.06, fdbk=0.0, decay=0.0; 

	var input, maxdelay, maxrate, dsig, mixed, local;
	maxdelay = 0.013;
	maxrate = 10.0;
	input = In.ar(in, 1);
	local = LocalIn.ar(1);
	dsig = AllpassL.ar( // the delay (you could use AllpassC (put 0 in decay))
		input + (local * fdbk),
		maxdelay * 2,
		LFPar.kr( // very similar to SinOsc (try to replace it) - Even use LFTri
			rate * maxrate,
			0,
			depth * maxdelay,
			delay * maxdelay),
		decay);
	mixed = input + dsig;
	LocalOut.ar(mixed);
	Out.ar([out, out+1], mixed);
}).add;
)

// audioIn on audio bus nr 10
{Out.ar(10, AudioIn.ar(1))}.play(s, addAction:\addToHead)

a = Synth(\flanger, [\in, 10], addAction:\addToTail)
a.set(\delay, 0.04)
a.set(\depth, 0.04)
a.set(\rate, 0.01)
a.set(\fdbk, 0.08)
a.set(\decay, 0.01)

// or if you prefer a buffer:
b = Buffer.read(s, "sounds/a11wlk01.wav"); // replace this sound with a nice sounding one !!!
{Out.ar(10, PlayBuf.ar(1, b.bufnum, BufRateScale.kr(b.bufnum), loop:1))}.play(addAction:\addToHead)

a = Synth(\flanger, [\in, 10], addAction:\addToTail)
a.set(\delay, 0.04)
a.set(\depth, 0.04)
a.set(\rate, 1)
a.set(\fdbk, 0.08)
a.set(\decay, 0.01)

// a parameter explosion results in a Chorus like effect:
a.set(\decay, 0)
a.set(\delay, 0.43)
a.set(\depth, 0.2)
a.set(\rate, 0.1)
a.set(\fdbk, 0.08)

// or just go mad:
a.set(\delay, 0.93)
a.set(\depth, 0.9)
a.set(\rate, 0.8)
a.set(\fdbk, 0.8)


## Chorus

The chorus effect happens when we add a delayed signal with the original with a time-varying delay. The delay has to be short in order not to be perceived as echo, but above 5 ms to be audible. If the delay is too short, it will destructively interfere with the un-delayed signal and create a flanging effect. Often, the delayed signals will be pitch shifted to create a harmony with the original signal.

There is no definite algorithm to create a chorus. There are many different ways to achieve it. As opposed to the Flanger above, this chorus does not have a feedback loop. But you could create a chorus effect out of a Flanger by using longer delay time (20-30 ms instead of 1-10 ms in the Flanger)

// a simple chorus
SynthDef(\chorus, { arg inbus=10, outbus=0, predelay=0.08, speed=0.05, depth=0.1, ph_diff=0.5;
	var in, sig, modulators, numDelays = 12;
	in = In.ar(inbus, 1) * numDelays.reciprocal;
	modulators = Array.fill(numDelays, {arg i;
      	LFPar.kr(speed * rrand(0.94, 1.06), ph_diff * i, depth, predelay);}); 
	sig = DelayC.ar(in, 0.5, modulators);  
	sig = sig.sum; //Mix(sig); 
	Out.ar(outbus, sig!2); // output in stereo
}).add


// try it with audio in
{Out.ar(10, AudioIn.ar(1))}.play(addAction:\addToHead)
// or a buffer:
b = Buffer.read(s, "sounds/a11wlk01.wav"); // replace this sound with a nice sounding one !!!
{Out.ar(10, PlayBuf.ar(1, b.bufnum, BufRateScale.kr(b.bufnum), loop:1))}.play(addAction:\addToHead)

a = Synth(\chorus, addAction:\addToTail)
a.set(\predelay, 0.02);
a.set(\speed, 0.22);
a.set(\depth, 0.5);
a.set(\pd_diff, 0.7);
a.set(\predelay, 0.2);


## Reverb

Achieving realistic reverb is a science on its own, to deep to delve into here. The most common reverb technique in digital acoustics is to use parallel comb delays that are fed into few Allpass delays.

Reverb can be analysed into 3 stages:
* Direct sound (from the soundsource)
* Early reflections (discrete 1st generation reflections from walls)
* Reverberation (Nth generation reflections that take time to build up, and fade out slowly)

SynthDef(\reverb, {arg inbus=0, outbus=0, predelay=0.048, combdecay=15, allpassdecay=1, revVol=0.31;
	var sig, y, z;
	sig = In.ar(inbus, 1); 
	
	// predelay
	z = DelayN.ar(sig, 0.1, predelay); // max 100 ms predelay
	
	// 7 length modulated comb delays in parallel :
	y = Mix.ar(Array.fill(7,{ CombL.ar(z, 0.05, rrand(0.03, 0.05), combdecay) })); 

	6.do({ y = AllpassN.ar(y, 0.050, rrand(0.03, 0.05), allpassdecay) });
	Out.ar(outbus, sig + (y * revVol) ! 2); // as fxlevel is 1 then I lower the vol a bit
}).add; 


{Out.ar(10, AudioIn.ar(1))}.play(addAction:\addToHead)

b = Buffer.read(s, "sounds/a11wlk01.wav"); // replace this sound with a nice sounding one !!!

{Out.ar(10, PlayBuf.ar(1, b.bufnum, BufRateScale.kr(b.bufnum), loop:1))}.play(addAction:\addToHead)

a = Synth(\reverb, [\inbus, 10], addAction:\addToTail)

a.set(\predelay, 0.048)
a.set(\combdecay, 2.048)
a.set(\allpassdecay, 1.048)
a.set(\revVol, 0.048)


## Tremolo

Tremolo is fluctuating amplitude of a signal, well known from analogue guitar amplifiers, and heard in surf music, or garage punk such as The Cramps.

SynthDef(\tremolo, {arg inbus=0, outbus=0, freq=1, strength=1; 
   var fx, sig; 
   sig = In.ar(inbus, 1); 
   fx = sig * SinOsc.ar(freq, 0, strength, 0.5, 2); 
   Out.ar(outbus, (fx+ sig).dup ) 
}).add; 

{Out.ar(10, AudioIn.ar(1))}.play(addAction:\addToHead)

b = Buffer.read(s, "sounds/a11wlk01.wav"); // replace this sound with a nice sounding one !!!
{Out.ar(10, PlayBuf.ar(1, b.bufnum, BufRateScale.kr(b.bufnum), loop:1))}.play(addAction:\addToHead)


a = Synth(\tremolo, [\inbus, 10], addAction:\addToTail)

a.set(\freq, 4.8)
a.set(\strength, 0.8)

## Distortion

Distortion can be achieved through diverse algorithms, but the most basic one could be to raise the amplitude of the signal so much that it starts to clip (below -1 and above 1), thus turning a sine wave into a square wave, adding harmonics.

(
{
var in, gain;
	in = AudioIn.ar(1);
	gain = MouseX.kr(1,100);
	in=in.abs;
	((in.squared + (gain*in))/(in.squared + ((gain-1)*in) + 1))
!2}.play
)

SuperCollider has a .distort method. 
(
{		// mouseX is pregain, mouseY is postgain
		var in, distortion, fx, y, z;
		in = AudioIn.ar(1);
		distortion = ((in * MouseX.kr(1,10)).distort * MouseY.kr(1,10)).distort;
		fx = Compander.ar(distortion, distortion, 1, 0, 1 ); // sustain
		Out.ar(0, LeakDC.ar(fx + in ) !2 );
}.play
)

// Here not using AudioIN:
b = Buffer.read(s, "sounds/a11wlk01.wav"); // replace this sound with a nice sounding one !!!
{Out.ar(10, PlayBuf.ar(1, b.bufnum, BufRateScale.kr(b.bufnum), loop:1))}.play(addAction:\addToHead)

(
{		// mouseX is pregain, mouseY is postgain
			var in, distortion, fx, y, z;
			in = In.ar(10);
			distortion = ((in * MouseX.kr(1,10)).distort * MouseY.kr(1,10)).distort;
			fx = Compander.ar(distortion, distortion, 1, 0, 1 ); // sustain
			Out.ar(0, LeakDC.ar(fx + in ) !2 );

}.play(addAction:\addToTail) // for addAction, see Synth helpfile or tutorial 13
)

## Compressor
 
The compressor reduces the dynamic range of a signal if it exceeds certain threshold. The compression ratio determines how much the signal exceeding the threshold is lowered. 4:1 compression ratio means that for every 4 dB of signal that goes into the unit, it lowers the signal such that only 1 dB is outputted.

(
{
	var in, compander;
	in = AudioIn.ar(1);
	compander = Compander.ar(in, in, MouseX.kr(0.001, 1, 1), 1, 0.5, 0.01, 0.01);
	compander ! 2 // stereo
}.play
)

e = Buffer.read(s, "sounds/a11wlk01.wav");

(
{
	var in, compander;
	in = PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1);
	compander = Compander.ar(in, in, MouseX.kr(0.0001, 1, 1), 1, 0.5, 0.01, 0.01);
	compander ! 2 // stereo
}.play
)

## Limiter

The limiter does essentially the same as the compressor, but it looks at the signal's peaks whereas the compressor looks at the average energy level. A limiter will not let the signal past the threshold, while the compressor does, according to the ratio settings.

The difference is in the slopeAbove argument of the Compander (0.5 in the compressor, but 0.1 in the limiter)

(
// limiter - Audio In
{
	var in, compander;
	in = AudioIn.ar(1);
	compander = Compander.ar(in, in, MouseX.kr(0.001, 1, 1), 1, 0.1, 0.01, 0.01);
	compander ! 2 // stereo
}.play
)

(
// limiter - Soundfile
{
	var in, compander;
	in = PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1);
	compander = Compander.ar(in, in, MouseX.kr(0.0001, 1, 1), 1, 0.1, 0.01, 0.01);
	compander ! 2 // stereo
}.play
)

## Sustainer

The sustainer works like an inverted compressor, it exaggerates the low amplitudes and tries to raise them up to the threshold defined.

(
// sustainer - Audio In
{
	var in, compander;
	in = AudioIn.ar(1);
	compander = Compander.ar(in, in, MouseX.kr(0.001, 1, 1), 0.1, 1, 0.01, 0.01);
	compander ! 2 // stereo
}.play
)

(
// sustainer - Soundfile
{
	var in, compander;
	in = PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1);
	compander = Compander.ar(in, in, MouseX.kr(0.0001, 1, 1), 0.1, 1, 0.01, 0.01);
	compander ! 2 // stereo
}.play
)

// for comparison, here is the file without sustain:
{PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1)!2}.play


## Noise gate

The noise gate allows a signal to pass through the filter only when it is above a certain threshold. If the energy of the signal is below the threshold, no sound is allowed to pass. It is often used in settings where there is background noise and one only wants to record the signal and not the (in this case) uninteresting noise.

(
// noisegate - Audio In
{
	var in, compander;
	in = AudioIn.ar(1);
	compander = 	Compander.ar(in, in, MouseX.kr(0.005, 1, 1), 10, 1, 0.01, 0.01);
	compander ! 2 // stereo
}.play
)

(
// noisegate - sound file
{
	var in, compander;
	in = PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1);
	compander = 	Compander.ar(in, in, MouseX.kr(0.001, 1), 10, 1, 0.01, 0.01);
	compander ! 2 // stereo
}.play
)

 The noise gate needs a bit of parameter tweaking to get what you want, so here is the same version as above, just with a MouseY controlling the slopeAbove parameter.

(
// noisegate - Audio In
{
	var in, compander;
	in = AudioIn.ar(1);
	compander = 	Compander.ar(in, in, MouseX.kr(0.005, 1, 1), MouseY.kr(1,20), 1, 0.01, 0.01);
	compander ! 2 // stereo
}.play
)


(
// noisegate - soundfile
{
	var in, compander;
	in = PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1);
	compander = 	Compander.ar(in, in, MouseX.kr(0.001, 1), MouseY.kr(1,20), 1, 0.01, 0.01);
	compander ! 2 // stereo
}.play
)


(
// for fun: a noisegater with a bit of reverb (controlled by mouseY)
// better use headphones - danger of feedback!
{
	var in, compander;
	var predelay=0.048, combdecay=3.7, allpassdecay=0.21, revVol=0.21;
	in = AudioIn.ar(1);
	compander = 	Compander.ar(in, in, MouseX.kr(0.005, 1, 1), 10, 1, 0.01, 0.01);
	z = DelayN.ar(compander, 0.1, predelay);
	y = Mix.ar(Array.fill(7,{ CombL.ar(z, 0.05, rrand(0.03, 0.05), MouseY.kr(1,20, 1)) })); 
	6.do({ y = AllpassN.ar(y, 0.050, rrand(0.03, 0.05), allpassdecay) });
	y!2
}.play
)

## Normalizer

Normalizer uses a buffer to store the sound in a small delay and look ahead in the audio. It will not overshoot like a Compander, but the downside is the delay. The normalizer normalizes the input amplitide to a given level.

(
// normalizer - Audio In
{
	var in, normalizer;
	in = AudioIn.ar(1);
	normalizer = Normalizer.ar(in, MouseX.kr(0.1, 0.9), 0.01);
	normalizer ! 2 // stereo
}.play
)

(
// normalizer - sound file
{
	var in, normalizer;
	in = PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1);
	normalizer = Normalizer.ar(in, MouseX.kr(0.1, 0.9), 0.01);
	normalizer ! 2 // stereo
}.play
)

## Limiter (Ugen)

Like the Normalizer, the Limiter uses a buffer to store the sound in a small delay buffer to look ahead in the audio. It will not overshoot like the Compander, but you have to put up with a slight the delay. The limiter limits the input amplitude to a given level.

(
// limiter - Audio In
{
	var in, normalizer;
	in = AudioIn.ar(1);
	normalizer = Limiter.ar(in, MouseX.kr(0.1, 0.9), 0.01);
	normalizer ! 2 // stereo
}.play
)

(
// limiter - sound file
{
	var in, normalizer;
	in = PlayBuf.ar(1, e.bufnum, BufRateScale.kr(e.bufnum), loop:1);
	normalizer = Limiter.ar(in, MouseX.kr(0.1, 0.9), 0.01);
	normalizer ! 2 // stereo
}.play
)

## Amplitude

Amplitude tracks the peak amplitude of a signal. It is not really an audio effect, but it can be a key element in the design of effects, (for example adaptive audio effects) and is therefore included here in this section.

In the example below, we map the input amplitude to frequency of a sine:

{SinOsc.ar(Amplitude.kr(AudioIn.ar(1), 0.1, 0.1, 12000, 0), 0, 0.3)}.play;


// with a noise gater as explained above
(
{
var noisegate, in;
in = AudioIn.ar(1);
noisegate = Compander.ar(in, in, MouseX.kr(0.005, 1, 1), MouseY.kr(1,20), 1, 0.01, 0.01);
SinOsc.ar(Amplitude.kr(noisegate, 0.1, 0.1, 12000, 0), 0, 0.3) ! 2
}.play;
)


// Compare the two following examples 

{SinOsc.ar(
	MouseX.kr(100, 6000, 1),
	0,
	Amplitude.kr(AudioIn.ar(1), 0.1, 0.1, 1, 0)
)!2}.play

// -- huh? --

{SinOsc.ar(
	MouseX.kr(100, 6000, 1),
	0,
	AudioIn.ar(1)
)!2}.play

## Pitch

Pitch tracks the pitch of a signal. If the pitch tracker has found the pitch, the hasFreq variable will be 1 (true), if it doesn't hold a pitch then it is 0 (false). (Read the helpfile about how it works)

NOTE: it can be useful to pass the input signal through a Low Pass Filter as it is easier to detect the pitch of a signal with less harmonics.

Tip: People often ask about the hashtag in front of the pitch and hasPitch variables. This is a way to assign two variables with valued from an array.

# a, b = [444, 555];
a
b

The simplest of patches - mapping pitch to the frequency of the sine

(
{
	var env, in, freq, hasFreq;
	
	// the audio input
	in = AudioIn.ar(1); 
	
	// the pitch variable and the hasFreq (Pitch.kr returns a list like this [freq, hasFreq])
	# freq, hasFreq = Pitch.kr(in, ampThreshold: 0.2, median: 7);
	
	// when the hasFreq is true (pitch is found) we generate a ADSR envelope that is open until
	// the hasFreq is false again or the amplitude is below the ampThreshold of the Pitch.
	env = EnvGen.ar(Env.adsr(0.51, 0.52, 1, 0.51, 1, -4), gate: hasFreq);
	
	// we plug the envolope to the volume argument of the Sine
	SinOsc.ar(freq, 0, env * 0.5) ! 2

}.play;
)

// a bit more complex patch where we use Amplitude to control volume:

(
{
	var env, in, freq, hasFreq, amp;
	
	// the audio input
	in = AudioIn.ar(1); 
	amp = Amplitude.kr(in, 0.25, 0.25);
	
	// the pitch variable and the hasFreq (Pitch.kr returns a list like this [freq, hasFreq])
	# freq, hasFreq = Pitch.kr(in, ampThreshold: 0.2, median: 7);
	
	// when the hasFreq is true (pitch is found) we generate a ADSR envelope that is open until
	// the hasFreq is false again or the amplitude is below the ampThreshold of the Pitch.
	env = EnvGen.ar(Env.adsr(0.51, 0.52, 1, 0.51, 1, -4), gate: hasFreq);
	
	// we plug the envolope to the volume argument of the Sine
	SinOsc.ar(freq, 0, env * amp) ! 2
	
}.play;
)



(
SynthDef(\pitcher,{
	var in, amp, freq, hasFreq, out, gate, threshold;
	
	threshold = 0.05; // change 
	
	// using a LowPassFilter to remove high harmonics
	in = LPF.ar(Mix.new(AudioIn.ar([1,2])), 2000);
	amp = Amplitude.kr(in, 0.25, 0.25);
	
	# freq, hasFreq = Pitch.kr(in, ampThreshold: 0.1, median: 7);
	gate = Lag.kr(amp > threshold, 0.01);	

	// -- to look at the values, uncomment the following lines 
	// -- (you need a recent build with the Poll class)
	//Poll.kr(Impulse.kr(10), freq, "frequency:");
	//Poll.kr(Impulse.kr(10), amp, "amplitude:");
	//Poll.kr(Impulse.kr(10), hasFreq, "hasFreq:");
	
	out = VarSaw.ar(freq, 0, 0.2, amp*hasFreq*gate);
	
	// uncomment (3 sines (octave lower, pitch and octave higher mixed into one signal (out)))
	//out = Mix.new(SinOsc.ar(freq * [0.5,1,2], 0, 0.2 * amp*hasFreq*gate));
	6.do({
		out = AllpassN.ar(out, 0.040, [0.040.rand,0.040.rand], 2)
	});
	Out.ar(0,out)
}).play(s);
)

In the example below we use the Tartini UGen by Nick Collins. In my experience it performs better than Pitch and is part of the SC3-plugins external plugins.

(
SynthDef(\pitcher,{
	var in, amp, freq, hasFreq, out, threshold, gate;

	threshold = 0.05; // change 
	in = LPF.ar(Mix.new(AudioIn.ar([1,2])), 2000);
	amp = Amplitude.kr(in, 0.25, 0.25);

	# freq, hasFreq = Tartini.kr(in);
	gate = Lag.kr(amp > threshold, 0.01);	
	
	// -- to look at the values, uncomment the following lines 
	// -- (you need a recent build with the Poll class)
	//Poll.kr(Impulse.kr(10), freq, "frequency:");
	//Poll.kr(Impulse.kr(10), amp, "amplitude:");
	//Poll.kr(Impulse.kr(10), hasFreq, "hasFreq:");
		
	out = Mix.new(VarSaw.ar(freq * [0.5,1,2], 0, 0.2, gate* hasFreq *amp ));
	//out = Mix.new(SinOsc.ar(freq * [0.5,1,2], 0, 0.2 * amp*hasFreq*gate));
	6.do({
		out = AllpassN.ar(out, 0.040, [0.040.rand,0.040.rand], 2)
	});
	Out.ar(0,out)
}).play(s);
)

## Filters

The filter Ugens in SuperCollider use complex time-domain algorithms to achieve the desired effect.

### Low Pass Filter
(
{
	var in;
	in = AudioIn.ar(1);
	LPF.ar(in, MouseX.kr(80, 4000));
}.play
)

(
{
	var in;
	in = Blip.ar(440);
	LPF.ar(in, MouseX.kr(80, 24000));
}.play
)

### Resonant Low Pass Filter
(
{
	var in;
	in = Blip.ar(440);
	RLPF.ar(in, MouseX.kr(80, 22000), MouseY.kr(0.0001, 1));
}.play
)

(
{
	var in;
	in = WhiteNoise.ar(1);
	RLPF.ar(in, MouseX.kr(80, 22000), MouseY.kr(0.0001, 1));
}.play
)

### High Pass Filter
(
{
	var in;
	in = Blip.ar(440);
	HPF.ar(in, MouseX.kr(80, 22000));
}.play
)

(
{
	var in;
	in = WhiteNoise.ar(1);
	HPF.ar(in, MouseX.kr(80, 22000));
}.play
)

### Resonant High Pass Filter

(
{
	var in;
	in = Blip.ar(440);
	RHPF.ar(in, MouseX.kr(80, 22000), MouseY.kr(0.0001, 1));
}.play
)

(
{
	var in;
	in = WhiteNoise.ar(1);
	RHPF.ar(in, MouseX.kr(80, 22000), MouseY.kr(0.0001, 1));
}.play
)



### Band Pass Filter
(
{
	var in;
	in = Blip.ar(440);
	BPF.ar(in, MouseX.kr(80, 22000), MouseY.kr(0.0001, 1));
}.play
)

(
{
	var in;
	in = WhiteNoise.ar(1);
	BPF.ar(in, MouseX.kr(80, 22000), MouseY.kr(0.0001, 1));
}.play
)

### Band Reject Filter

{ BRF.ar(Saw.ar(200,0.1), FSinOsc.kr(XLine.kr(0.7,300,20),0,3800,4000), 0.3) }.play;

{ BRF.ar(Saw.ar(200,0.5), MouseX.kr(100, 10000, 1), 0.3) }.play;



### SOS - A biquad filter

A second order filter, also known as biquad filter. The helpfile shows the algorithm itself:

out(i) = (a0 * in(i)) + (a1 * in(i-1)) + (a2 * in(i-2)) + (b1 * out(i-1)) + (b2 * out(i-2))

Where you can see that the filter reaches back to the second sample after the current one, and uses parameters (a0, a1, b1 and b2) to affect the function of the filter.

(
{
	var rho, theta, b1, b2;
	theta = MouseX.kr(0.2pi, pi);
	rho = MouseY.kr(0.6, 0.99);
	b1 = 2.0 * rho * cos(theta);
	b2 = rho.squared.neg;
	SOS.ar(WhiteNoise.ar(0.1 ! 2), 1.0, 0.0, 0.0, b1, b2)
}.play
)


### Resonant filter

This filter will resonate frequencies at the set frequency. The bwr parameter is the bandwidth ratio, that is, how much energy is passed on each side of the centre frequency.

{ Resonz.ar(WhiteNoise.ar(0.5), 2000, XLine.kr(1, 0.001, 8)) }.play

// high amp input (from Impulse) and low RQ makes a note 

{Resonz.ar(Impulse.ar(1.5, 0, 50), Rand(200,2000), 0.03) }.play

// try putting 500 in amp and 0.003 in RQ
{Resonz.ar(Impulse.ar(1.5, 0, 500), Rand(200,2000), 0.003) }.play


// for fun ( if you don't like the polyrhythm, put 1 instead of trig)
// or if you like it, then put some more tempi in there and appropriate weights

(
var trig;
var wait = 4;
Task({
	20.do({
		trig = [1, 1.5].wchoose([0.7, 0.3]);
		{Resonz.ar(Impulse.ar(trig, 0, 50*rrand(5,10)), Rand(200,2000), 0.003) ! 2}.play;
		(wait + rrand(0.1,1)).wait;
		wait = wait - rrand(0.01, 0.2);
	})
}).play
)

