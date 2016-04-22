# Physical Modeling

Physical modelling is a common synthesis technique where a mathematical model is built of some physical object. The maths here can be quite complex and outside the scope of this book. However, it is worth exploring the technique as there are PM UGens in SuperCollider and many musical instruments can easily be built using simple physical models, using filters and alike. Waveguide synthesis can model the physics of the acoustic instrument or sound generating object. It simulates the traveling of waves through a string or a tube. The physical structures of an instrument can be thought of as waveguides or a transmission lines. 
In physical modelling, as opposed to traditional synthesis types (AM, FM, granular, etc), we are not imitating the sound of an instrument, but rather simulating the instrument itself and the physical laws that are involved in the creation of a sound.In physical modelling of sound we typically operate with excitation and a resonant body. The excitation is the material and weight of the thing that hits, whilst the resonant body is what is being hit and resonates. In many cases it does not make sense separating the two this way mathematically, but from a user-perspective we can think of material bodies of wood, glass, metal, or a string, as examples, being hit by a finger, a plectrum, a metal hammer, or anything imaginable, for example falling sand. Further resolution can be designed in the model of the instrument, for example defining the bridge of a guitar, the type of strings, the type of body, the room which the instrument is in, etc.

For a good text on physical modelling, check Julius O. Smith's "Physical Audio Signal Processing":
http://ccrma.stanford.edu/~jos/pasp/pasp.html

Karplus-Strong synthesis (named after its authors) is a precursor of physical modelling and is good for synthesising strings and percussion sounds.

{line-numbers=off}
~~~~~~~
// we generate a short burst (the excitation)
{ Decay.ar(Impulse.ar(1), 0.1, WhiteNoise.ar) }.play

// we then wrap that noise in a fast repeating delay
{ CombL.ar(Decay.ar(Impulse.ar(1), 0.1, WhiteNoise.ar), 0.02, 0.001, 3, 1) }.play
~~~~~~~

The repeat rate of the delay becomes the pitch of the string, so 0.001 is 1000 Hz, or in a reciprocal relationship. We could therefore write 440.reciprocal in the delayTime argument of the CombL, and it would give us a string sound of 440 Hz. The duration of the string is controlled by the decayTime argument. This is the basic ingredient of a string synthesizer, but for further development, you might want to consider applying filters to the noise, or perhaps use another type of noise. Also, the time of the burst (above 100 ms) will affect the sound heavily.

{line-numbers=off}
~~~~~~~
SynthDef(\ks_string, { arg note, pan, rand, delayTime;
	var x, y, env;
	env = Env.new(#[1, 1, 0],#[2, 0.001]);
	// A simple exciter x, with some randomness.
	x = Decay.ar(Impulse.ar(0, 0, rand), 0.1+rand, WhiteNoise.ar); 
 	x = CombL.ar(x, 0.05, note.reciprocal, delayTime, EnvGen.ar(env, doneAction:2)); 
	x = Pan2.ar(x, pan);
	Out.ar(0, LeakDC.ar(x));
}).add;

{ // and play the synthdef
	20.do({
		Synth(\ks_string, 
			[\note, [48, 50, 53, 58].midicps.choose, 
			\pan, 1.0.rand2, 
			\rand, 0.1+0.1.rand, 
			\delayTime, 2+1.0.rand]);
		[0.125, 0.25, 0.5].choose.wait;
	});
}.fork;

// here using patterns
Pdef(\kspattern, 
	Pbind(\instrument, \ks_string, // using our sine synthdef
			\note, Pseq.new([60, 61, 63, 66], inf).midicps, // freq arg
			\dur, Pseq.new([0.25, 0.5, 0.25, 1], inf),  // dur arg
			\rand, Prand.new([0.2, 0.15, 0.15, 0.11], inf),  // dur arg
			\pan, 1.0.rand2,
			\delayTime, 2+1.0.rand;  // envdur arg
		)
).play;
~~~~~~~

Compare using white noise and pink noise as an exciter, as well as using a resonant filter to filter the burst:

{line-numbers=off}
~~~~~~~
// white noise
{  
	var burstEnv, burst; 
	burstEnv = EnvGen.kr(Env.perc(0, 0.01), gate: Impulse.kr(1.5));
	burst = WhiteNoise.ar(burstEnv);
	CombL.ar(burst, 0.2, 0.003, 1.9, add: burst);  
}.play;

// pink noise
{  
	var burstEnv, burst; 
	burstEnv = EnvGen.kr(Env.perc(0, 0.01), gate: Impulse.kr(1.5));
	burst = PinkNoise.ar(burstEnv);
	CombL.ar(burst, 0.2, 0.003, 1.9, add: burst);  
}.play;

// here we use RLPF (resonant low pass filter) to filter the white noise burst
{  
	var burstEnv, burst; 
	burstEnv = EnvGen.kr(Env.perc(0, 0.01), gate: Impulse.kr(1.5));
	burst = RLPF.ar(WhiteNoise.ar(burstEnv), MouseX.kr(100, 12000), MouseY.kr(0.001, 0.999));
	CombL.ar(burst, 0.2, 0.003, 1.9, add: burst);  
}.play;
~~~~~~~

SuperCollider comes with a plug called *Pluck* which is an implementation of the Karplus-Strong synthesis. This should be more effective than the examples above, but of similar sound.

{line-numbers=off}
~~~~~~~
{Pluck.ar(WhiteNoise.ar(0.1), Impulse.kr(2), MouseY.kr(220, 880).reciprocal, MouseY.kr(220, 880).reciprocal, 10, coef:MouseX.kr(-0.1, 0.5)) !2 }.play(s)
~~~~~~~

We could create a SynthDef with Pluck.

{line-numbers=off}
~~~~~~~
SynthDef(\pluck, {arg freq=440, trig=1, time=2, coef=0.1, cutoff=2, pan=0;
	var pluck, burst;
	burst = LPF.ar(WhiteNoise.ar(0.5), freq*cutoff);
	pluck = Pluck.ar(burst, trig, freq.reciprocal, freq.reciprocal, time, coef:coef);
	Out.ar(0, Pan2.ar(pluck, pan));
}).add;

Synth(\pluck);
Synth(\pluck, [\coef, 0.01]);
Synth(\pluck, [\coef, 0.3]);
Synth(\pluck, [\coef, 0.7]);

Synth(\pluck, [\coef, 0.3, \time, 0.1]);
Synth(\pluck, [\coef, 0.3, \time, 5]);

Synth(\pluck, [\coef, 0.2, \time, 5, \cutoff, 1]);
Synth(\pluck, [\coef, 0.2, \time, 5, \cutoff, 2]);
Synth(\pluck, [\coef, 0.2, \time, 5, \cutoff, 5]);
Synth(\pluck, [\coef, 0.2, \time, 5, \cutoff, 15]);

// A guitar that might need a little distortion
Pbind(\instrument, \pluck,
	\freq, Pseq([72, 70, 67,65, 63, 60, 48], inf).midicps,
	\dur, Pseq([0.5, 0.5, 0.375, 0.125, 0.5, 2], 1),
	\cutoff, Pseq([15, 10, 5, 2, 10, 10, 15], 1)	
).play
~~~~~~~

## Biquad filter

In SuperCollider, the SOS UGen is a second order biquad filter that can be used to create various interesting sounds. We could start with simple glass-like sound:

{line-numbers=off}
~~~~~~~
{SOS.ar(Impulse.ar(2), 0.0, 0.05, 0.0, MouseY.kr(1.45, 1.998, 1), MouseX.kr(-0.999, -0.9998, 1))!2}.play
~~~~~~~

And with slight changes we have a more *woody* type of sound:

{line-numbers=off}
~~~~~~~
SynthDef(\marimba, {arg out=0, amp=0.1, t_trig=1, freq=100, rq=0.006;
	var env, signal;
	var rho, theta, b1, b2;
	b1 = 1.987 * 0.9889999999 * cos(0.09);
	b2 = 0.998057.neg;
	signal = SOS.ar(K2A.ar(t_trig), 0.3, 0.0, 0.0, b1, b2);
	signal = RHPF.ar(signal*0.8, freq, rq) + DelayC.ar(RHPF.ar(signal*0.9, freq*0.99999, rq*0.999), 0.02, 0.01223);
	signal = Decay2.ar(signal, 0.4, 0.3, signal);
	DetectSilence.ar(signal, 0.01, doneAction:2);
	Out.ar(out, signal*(amp*0.4)!2);
}).add;

Pbind(
	\instrument, \marimba, 
	\midinote, Prand([[1,5], 2, [3, 5], 7, 9, 3], inf) + 48, 
	\dur, 0.2 
).play;

// Or perhaps
SynthDef(\wood, {arg out=0, amp=0.3, pan=0, sustain=0.5, t_trig=1, freq=100, rq=0.06;
	var env, signal;
	var rho, theta, b1, b2;
	b1 = 2.0 * 0.97576 * cos(0.161447);
	b2 = 0.9757.squared.neg;
	signal = SOS.ar(K2A.ar(t_trig), 1.0, 0.0, 0.0, b1, b2);
	signal = Decay2.ar(signal, 0.4, 0.8, signal);
	signal = Limiter.ar(Resonz.ar(signal, freq, rq*0.5), 0.9);
	env = EnvGen.kr(Env.perc(0.00001, sustain, amp), doneAction:2);
	Out.ar(out, Pan2.ar(signal, pan)*env);
}).add;

Pbind(
	\instrument, \wood, 
	\midinote, Prand([[1,5], 2, [3, 5], 7, 9, 3], inf) + 48, 
	\dur, 0.2 
).play;
~~~~~~~

## Waveguide synthesis

Waveguide synthesis is the most common form of physical modelling, often using delay lines, filtering, feedback and other non-linear elements. The Waveguide flute below is based upon Hans Mikelson's Csound slide flute (ultimately derived from Perry Cook's) STK slide flute physical model.  SuperCollider port by John E. Bower, who kindly allowed for the flute's inclusion in this tutorial. 

{line-numbers=off}
~~~~~~~
SynthDef("waveguideFlute", { arg scl = 0.2, pch = 72, ipress = 0.9, ibreath = 0.09, ifeedbk1 = 0.4, ifeedbk2 = 0.4, dur = 1, gate = 1, amp = 2, vibrato=0.2;	
	var kenv1, kenv2, kenvibr, kvibr, sr, cr, block, poly, signalOut, ifqc,  fdbckArray;
	var aflow1, asum1, asum2, afqc, atemp1, ax, apoly, asum3, avalue, atemp2, aflute1;
	
	sr = SampleRate.ir;
	cr = ControlRate.ir;
	block = cr.reciprocal;
	ifqc = pch.midicps;	
	// noise envelope
	kenv1 = EnvGen.kr(Env.new( 
		[ 0.0, 1.1 * ipress, ipress, ipress, 0.0 ], [ 0.06, 0.2, dur - 0.46, 0.2 ], 'linear' )
	);
	// overall envelope
	kenv2 = EnvGen.kr(Env.new(
		[ 0.0, amp, amp, 0.0 ], [ 0.1, dur - 0.02, 0.1 ], 'linear' ), doneAction: 2 
	);
	// vibrato envelope
	kenvibr = EnvGen.kr(Env.new( [ 0.0, 0.0, 1, 1, 0.0 ], [ 0.5, 0.5, dur - 1.5, 0.5 ], 'linear') )*vibrato;
	// create air flow and vibrato
	aflow1 = LFClipNoise.ar( sr, kenv1 );
	kvibr = SinOsc.ar( 5, 0, 0.1 * kenvibr );
	asum1 = ( ibreath * aflow1 ) + kenv1 + kvibr;
	afqc = ifqc.reciprocal - ( asum1/20000 ) - ( 9/sr ) + ( ifqc/12000000 ) - block;
	fdbckArray = LocalIn.ar( 1 );
	aflute1 = fdbckArray;
	asum2 = asum1 + ( aflute1 * ifeedbk1 );
	//ax = DelayL.ar( asum2, ifqc.reciprocal * 0.5, afqc * 0.5 );
	ax = DelayC.ar( asum2, ifqc.reciprocal - block * 0.5, afqc * 0.5 - ( asum1/ifqc/cr ) + 0.001 );
	apoly = ax - ( ax.cubed );
	asum3 = apoly + ( aflute1 * ifeedbk2 );
	avalue = LPF.ar( asum3, 2000 );
	aflute1 = DelayC.ar( avalue, ifqc.reciprocal - block, afqc );
	fdbckArray = [ aflute1 ];
	LocalOut.ar( fdbckArray );
	signalOut = avalue;
	OffsetOut.ar( 0, [ signalOut * kenv2, signalOut * kenv2 ] );	
}).add;

// Test the flute
Synth(\waveguideFlute, [\amp, 0.5, \dur, 5, \ipress, 0.90, \ibreath, 0.00536, \ifeedbk1, 0.4, \ifeedbk2, 0.4, \pch, 60, \vibrato, 0.2] );

// test the flute player's skills:
Routine({
	var pitches, durations, rhythm;
	pitches = Pseq( [ 47, 49, 53, 58, 55, 53, 52, 60, 54, 43, 52, 59, 65, 58, 59, 61, 67, 64, 58, 53, 66, 73 ], inf ).asStream;
	durations = Pseq([ Pseq( [ 0.15 ], 17 ), Pseq( [ 2.25, 1.5, 2.25, 3.0, 4.5 ], 1 ) ], inf).asStream;
	17.do({
		rhythm = durations.next;		
		Synth(\waveguideFlute, [\amp, 0.6, \dur, rhythm, \ipress, 0.93, \ibreath, 0.00536, \ifeedbk1, 0.4, \ifeedbk2, 0.4, \pch, pitches.next] );
		rhythm.wait;	
	});
	5.do({
		rhythm = durations.next;		
		Synth(\waveguideFlute, [\amp, 0.6, \dur, rhythm + 0.25, \ipress, 0.93, \ibreath, 0.00536, \ifeedbk1, 0.4, \ifeedbk2, 0.4, \pch, pitches.next] );		
		rhythm.wait;
	});	
}).play;
~~~~~~~


## Filters

Filters are a vital element in physical modelling. The main concepts here are some kind of an exciter (where in SuperCollider we might use triggers such as Impulse, Dust, or filtered noise) and a resonator (such as the Resonz and Klank resonators, Delays, Reverbs, etc.)

### Ringz

The Ringz is a powerful ringing filter, with a decay time, so the impulse can ring for N amount of seconds. Let's explore some examples:

{line-numbers=off}
~~~~~~~
// triggering a ringing filter by one impulse
{ Ringz.ar(Impulse.ar(0), 2000, 2) }.play

// one impulse per second
{ Ringz.ar(Impulse.ar(1), 2000, 2) }.play

// here using an envelope to soften the attack
{ Ringz.ar(EnvGen.ar(Env.perc(0.01, 1, 1), Impulse.ar(1)), 2000, 2) }.play

// playing with the frequency
{ Ringz.ar(Impulse.ar(4)*0.2, LFNoise0.ar(4)*2000, 0.1) }.play

// using XLine to increase rate and frequency
{ Ringz.ar(Impulse.ar(XLine.ar(1, 10, 4))*0.2, LFNoise0.ar(XLine.ar(1, 10, 4))*2000, 0.1) }.play

// using Dust instead of Impulse
{ Ringz.ar(Dust.ar(3, 0.3), 2000, 2) }.play

// here we use an Impulse to trigger the incoming sound
{ Ringz.ar(Impulse.ar(MouseX.kr(1, 100, 1)), 1800, MouseY.kr(0.05, 1), 0.4) }.play;

// control frequency as well
{ Ringz.ar(Impulse.ar(10)*0.5, MouseY.kr(100,1000), MouseX.kr(0.001,1)) }.play

// you could also usa an envelope to soften the attack
{ Ringz.ar(EnvGen.ar(Env.perc(0.001, 1), Impulse.kr(MouseX.kr(1, 100, 1))), 1800, MouseY.kr(0.05, 1), 0.4) }.play;

// here resonating white noise instead of a trigger
{ Ringz.ar(WhiteNoise.ar(0.005), 600, 4) }.play

// would this be useful in synthesizing a flute?
{ Ringz.ar(LPF.ar(WhiteNoise.ar(0.005), MouseX.kr(100, 10000)), 600, 1) !2}.play

// a modified example from the documentation
{({Ringz.ar(WhiteNoise.ar(0.001),XLine.kr(exprand(100.0,5000.0), exprand(100.0,5000.0), 20), 0.5)}!10).sum}.play

// The Formlet UGen is a type of Ringz filter, useful for formant control:
{ Formlet.ar(Blip.ar(MouseX.kr(10, 400), 1000, 0.1), MouseY.kr(10, 1000), 0.005, 0.04) }.play;
~~~~~~~



### Resonz, Klank and DynKlank

The Resonz is a …

{line-numbers=off}
~~~~~~~
// mouse y controlling frequency - mousex controling bandwidth ratio
{ Resonz.ar(Impulse.ar(10)*1.5, MouseY.kr(40,10000), MouseX.kr(0.001,1)) }.play

// here with white noise - mouse y controlling frequency - mousex controling bandwidth ratio
{ Resonz.ar(WhiteNoise.ar(0.1), MouseY.kr(40,10000), MouseX.kr(0.001,1)) }.play

// playing with Ringz and Resonz
{ Ringz.ar(Resonz.ar(Dust.ar(20)*1.5, MouseY.kr(40,10000), MouseX.kr(0.001,1)), MouseY.kr(40,10000), 0.04) }.play;

// let's explore the resonance using the freqscope
{ Resonz.ar(WhiteNoise.ar(0.1), MouseY.kr(40,10000), MouseX.kr(0.001,1)) }.freqscope

// Klank is a bank of Resonz filters 
{ Klank.ar(`[[800, 1071, 1153, 1723], nil, [1, 0.9, 0.1, 2]], Impulse.ar(1, 0, 0.2)) }.play;

// Klank filtering WhiteNoise
{ Klank.ar(`[[800, 1200, 1600, 200], [1, 0.8, 0.4, 0.8], [1, 1, 1, 1]], WhiteNoise.ar(0.001)) }.play;

// DynKlang is dynamic - using the mouse to change frequency and ringtime
{   var freqs, ringtimes;
    freqs = [800, 1071, 1153, 1723] * MouseX.kr(0.5, 2);
    ringtimes = [1, 1, 1, 1] * MouseY.kr(0.001, 5);
	DynKlank.ar(`[freqs, nil, ringtimes ], PinkNoise.ar(0.001))
}.play;
~~~~~~~

### Decay

{line-numbers=off}
~~~~~~~

{ Decay.ar(Impulse.ar(XLine.kr(1,50,20), 0.25), 0.2, FSinOsc.ar(600), 0)  }.play;

{ Decay2.ar(Impulse.ar(XLine.kr(1,50,20), 0.25), 0.1, 0.3, FSinOsc.ar(600)) }.play;

SynthDef(\clap, {arg out=0, pan=0, amp=0.3, filterfreq=50, rq=0.01;
	var env, signal, attack, noise, hpf1, hpf2;
	noise = WhiteNoise.ar(1)+SinOsc.ar([filterfreq/2,filterfreq/2+4 ], pi*0.5, XLine.kr(1,0.01,4));
	hpf1 = RLPF.ar(noise, filterfreq, rq);
	hpf2 = RHPF.ar(noise, filterfreq/2, rq/4);
	env = EnvGen.kr(Env.perc(0.003, 0.00035));
	signal = (hpf1+hpf2) * env;
	signal = CombC.ar(signal, 0.5, 0.03, 0.031)+CombC.ar(signal, 0.5, 0.03016, 0.06);
	signal = Decay.ar(signal, 1.5);
	signal = FreeVerb.ar(signal, 0.23, 0.1, 0.12);
	Out.ar(out, Pan2.ar(signal * amp, pan));
	DetectSilence.ar(signal, doneAction:2);
}).add;

Synth(\clap, [\filterfreq, 1700, \rq, 0.14, \amp, 0.1]);

~~~~~~~


## TBall, Spring and Friction

Physical modelling can involve the mathematical modelling of all kinds of phenomena, from wind to water to the simulation of moving or falling objects where gravity, speed, surface type, etc., are all parameters. The popular Box2D library (of AngryBirds fame) is one such library that simulates physical systems. In SuperCollider there are UGens that do that, for example TBall (Trigger Ball) and Spring

{line-numbers=off}
~~~~~~~
// arguments are trigger, gravity, damp and friction
{TBall.ar(Impulse.ar(0), 0.1, 0.2, 0.01)*20}.play

// a light ball falling on a bouncy surface on the moon?
{TBall.ar(Impulse.ar(0), 0.1, 0.1, 0.001)*20}.play

// a heavy ball falling on a soft surface?
{TBall.ar(Impulse.ar(0), 0.1, 0.2, 0.1)*20}.play
~~~~~~~

Having explored the qualities of the TBall as a system that outputs impulses according to a physical system, we can now apply these impulses in some of the resonant filters that we have explored above.

{line-numbers=off}
~~~~~~~
// here using Ringz to create a metal ball falling on a marble table
{Ringz.ar(TBall.ar(Impulse.ar(0), 0.09, 0.1, 0.01)*20, 3000, 0.08)}.play

// many balls falling randomly (using Dust)
{({Ringz.ar(TBall.ar(Dust.ar(2), 0.09, 0.1, 0.01)*20, rrand(2000,3000), 0.07)}!5).sum}.play

// here using Decay to create a metal ball falling on a marble table
{Decay.ar(TBall.ar(Impulse.ar(0), 0.09, 0.1, 0.01)*20, 1)}.play

// a drummer on the snare?
{LPF.ar(WhiteNoise.ar(0.5), 4000)*Decay.ar(TBall.ar(Impulse.ar(0), 0.2, 0.16, 0.003)*20, 1)!2}.play

{SOS.ar(TBall.ar(Impulse.ar(0), 0.09, 0.1, 0.01)*20, 0.6, 0.0, 0.0, rrand(1.991, 1.994), -0.9982)}.play

// Txalaparta? 
{({|x| SOS.ar(TBall.ar(Impulse.ar(1, x*0.1*x), 0.8, 0.2, 0.02)*20, 0.6, 0.0, 0.0, rrand(1.992, 1.99), -0.9982)}!6).sum}.play
~~~~~~~

The Spring UGen is a physical model of a resonating spring. Considering the wave properties of spring this can be very useful for synthesis.


{line-numbers=off}
~~~~~~~
{
	var trigger =LFNoise0.ar(1)>0;
	var signal = SinOsc.ar(Spring.ar(trigger,1,4e-06)*1220);
	var env = EnvGen.kr(Env.perc(0.001,5),trigger);
	Out.ar(0, Pan2.ar(signal * env, 0));
}.play

// Two springs:
{
	var trigger = LFNoise0.ar(1)>0;
	var springs = Spring.ar(trigger,1,4e-06) * Spring.ar(trigger,2,4e-07);
	var signal = SinOsc.ar(springs*1220);
	var env = EnvGen.kr(Env.perc(0.001,5),trigger);
	Out.ar(0, Pan2.ar(signal * env, 0));
}.play

// And here are two tweets (less than 140 characters) simulating timpani drums. 

play{{x=LFNoise0.ar(1)>0;SinOsc.ar(Spring.ar(x,4,3e-05)*(70.rand+190)+(30.rand+90))*EnvGen.kr(Env.perc(0.001,5),x)}!2}

// here heavy on the tuning pedal
play{{x=LFNoise0.ar(1)>0;SinOsc.ar(Spring.ar(x,4,3e-05)*(70.rand+190)+LFNoise2.ar(1).range(90,120))*EnvGen.kr(Env.perc(0.001,5),x)}!2}
~~~~~~~

In the SC3plugins you'll find a Friction UGen which is a physical model of a mass resting on a belt. The documentation of the UGen is good, but two examples are provided here for fun:

{line-numbers=off}
~~~~~~~
{Friction.ar(Ringz.ar(Impulse.ar(1), [400, 412]), 0.0002, 0.2, 2, 2.697)}.play

{Friction.ar(Klank.ar(`[[400, 412, 340]], Impulse.ar(1)), 0.0002, 0.2, 2, 2.697)!2}.play
~~~~~~~

## The MetroGnome 

How about trying to synthesise a wooden old-fashioned metronome?

{line-numbers=off}
~~~~~~~
(
SynthDef(\metro, {arg tempo=1, filterfreq=1000, rq=1.0;
var env, signal;
	var rho, theta, b1, b2;
	theta = MouseX.kr(0.02, pi).poll;
	rho = MouseY.kr(0.7, 0.9999999).poll;
	b1 = 2.0 * rho * cos(theta);
	b2 = rho.squared.neg;
	signal = SOS.ar(Impulse.ar(tempo), 1.0, 0.0, 0.0, b1, b2);
	signal = RHPF.ar(signal, filterfreq, rq);
	Out.ar(0, Pan2.ar(signal, 0));
}).add
)

// Move the mouse to find your preferred metronome (low left works best for me). We are here polling the MouseX and MouseY Ugens, so you will be able to follow their output in the post window.

a = Synth(\metro) // we create our metronome
a.set(\tempo, 0.5.reciprocal) // 120 bpm (0.5.reciprocal = 2 bps)
a.set(\filterfreq, 4000) // try 1000 (for example)
a.set(\rq, 0.1) // try 0.5 (for example)

// Let's reinterpret the Poème symphonique was composed by György Ligeti (in 1962)
// http://www.youtube.com/watch?v=QCp7bL-AWvw

SynthDef(\ligetignome, {arg tempo=1, filterfreq=1000, rq=1.0;
var env, signal;
	var rho, theta, b1, b2;
	b1 = 2.0 * 0.97576 * cos(0.161447);
	b2 = 0.97576.squared.neg;
	signal = SOS.ar(Impulse.ar(tempo), 1.0, 0.0, 0.0, b1, b2);
	signal = RHPF.ar(signal, filterfreq, rq);
	Out.ar(0, Pan2.ar(signal, 0));
}).add;

// and we create 10 different metronomes running in different tempi
// (try with 3 metros or 30 metros)
(
10.do({
	Synth(\ligetignome).set(
		\tempo, (rrand(0.5,1.5)).reciprocal, 
		\filterfreq, rrand(500,4000), 
		\rq, rrand(0.3,0.9) )
});
)
~~~~~~~

## The STK synthesis kit

Many years ago, Paul Lansky ported the STK physical modelling kit by Perry Cook and Gary Scavone for SuperCollider. This collection of UGens can be found in the SC3plugins, but they have not been maintained and the code is in a bad shape, although there are still some UGens that work. It could be a good project for someone wanting to explore the source of a classic physical modelling source code to update the UGens for SuperCollider 3.7+. 

Here below we have a model of a xylophone:

{line-numbers=off}
~~~~~~~
SynthDef(\xylo, { |out=0, freq=440, gate=1, amp=0.3, sustain=0.5, pan=0|
	var sig = StkBandedWG.ar(freq, instr:1, mul:3);
	var env = EnvGen.kr(Env.adsr(0.0001, sustain, sustain, 0.3), gate, doneAction:2);
	Out.ar(out, Pan2.ar(sig, pan, env * amp));
}).add;

Synth(\xylo)

Pbind(\instrument, \xylo, \freq, Pseq(({|x|x+60}!13).mirror).midicps, \dur, 0.2).play
~~~~~~~

