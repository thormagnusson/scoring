# Granular Synthesis

Granular synthesis is a synthesis technique that became available for most practical purposes with digital computer music software. Early pioneers were Barry Truax and Iannis Xenakis, but the technique has been well explored in the work of Curtis Roads, both in his musical output and in a fine book called *Microsound*.
The idea in granular synthesis is to synthesize a sound using small grains, typically of 10-50 millisecond duration, that are wrapped in envelopes. These grains can then result in a continuous sound or more discontinuous 'grain clouds'. Here the individual grains become the building blocks, almost atoms, of a more complex structure. 

## Granular Synthesis

Granular synthesis is used in many pitch shifting and time stretching features of commercial software so most people would be well aware of its functionality and power. Let us explore the pitch shifting through the use of an indigenous SuperCollider UGen, the PitchShift. In the examples below, the grains are 100 ms windows that overlap. What is really happening is that the sample is played at variable rate (where rate of 2 is an octave higher), but the grains are layered on top of each other in order to maintain the time of the sound. 

![An example of a grain](images/ch10/grain.png)

{line-numbers=off}
~~~~~~~
b = Buffer.read(s, Platform.userAppSupportDir+/+"sounds/a11wlk01.wav");

// MouseX controls the pitch
{ PitchShift.ar(PlayBuf.ar(1, b, 1, loop:1), 0.1, MouseX.kr(0,2), 0, 0.01) ! 2}.play;
// Same as above, but here MouseY gives random pitch
{ PitchShift.ar(PlayBuf.ar(1, b, 1, loop:1), 0.1, MouseX.kr(0,2), MouseY.kr(0, 2), 0.01) ! 2}.play;
~~~~~~~

The grains are windows with a specific envelope (typically a Hanning envelope) and they overlap in order to create the continuous sound. Play around with the parameters of window size and overlap to explore how they result in different sound. The above examples used PitchShift for the purposes of changing the pitch but keeping the same playback rate. Below we use Warp1 to time stretch sound where the pitch remains the same. 

{line-numbers=off}
~~~~~~~
// speed up the sound (with same pitch)
{Warp1.ar(1, b, Line.kr(0,1, 1), 1, 0.1, -1, 8, 0.1, 2)!2}.play

// slow down the sound (with the same pitch)
{Warp1.ar(1, b, Line.kr(0,1, 10), 1, 0.09, -1, 8, 0.1, 2)!2}.play

// use the mouse to read the sound (at the same pitch)
{Warp1.ar(1, b, MouseX.kr(0,1), 1, 0.1, -1, 8, 0.1, 2)!2}.play

// A SinOsc reading the sound (at the same pitch)
{Warp1.ar(1, b, SinOsc.kr(0.07).range(0,1), 1, 0.1, -1, 8, 0.1, 2)!2}.play

// use the mouse to read the sound (and control the pitch)
{Warp1.ar(1, b, MouseX.kr(0,1), MouseY.kr(0.5,2), 0.1, -1, 8, 0.1, 2)!2}.play
~~~~~~~

### TGrains

The TGrains Ugen - or Trigger Grains - is a handy UGen for quick and basic granular synthesis. Here we can pass arguments such as number of grains per second, grain duration, rate (which is pitch), etc.

{line-numbers=off}
~~~~~~~
// mouse Y controlling number of grains per second
{TGrains.ar(2, Impulse.ar(MouseY.kr(1, 30)), b, 1, MouseX.kr(0,BufDur.kr(b)), 2/MouseY.kr(1, 10), 0, 0.8, 2)}.play

// mouse Y controlling pitch
{TGrains.ar(2, Impulse.ar(20), b, MouseY.kr(0.5, 2), MouseX.kr(0,BufDur.kr(b)), 2/MouseY.kr(1, 10), 0, 0.8, 2)}.play

// random pitch location, with mouse X controlling number 
// of grains per second an mouse Y controlling grain duration
{
TGrains.ar(2, 
	Impulse.ar(MouseX.kr(1, 50)), 
	b, 
	LFNoise0.ar(40, add:1), 
	LFNoise0.ar(40).abs * BufDur.kr(b), 
	MouseY.kr(0.01, 0.05), 
	0, 
	0.8, 
	2)
}.play
~~~~~~~

### GrainIn 

GrainIn enables you to granularize incoming audio. This UGen is part of a collection of other granular Ugens, such as GrainSin, GrainFM, and GrainBuf. Take a look at the documentation of these UGens and explore their functionality. 

{line-numbers=off}
~~~~~~~
SynthDef(\sagrain, {arg amp=1, grainDur=0.1, grainSpeed=10, panWidth=0.5;
	var pan, granulizer;
	pan = LFNoise0.kr(grainSpeed, panWidth);
	granulizer = GrainIn.ar(2, Impulse.kr(grainSpeed), grainDur, SoundIn.ar(0), pan);
	Out.ar(0, granulizer * amp);
}).add;

x = Synth(\sagrain)

x.set(\grainDur, 0.02)
x.set(\amp, 0.02)
x.set(\amp, 1)

x.set(\grainDur, 0.1)
x.set(\grainSpeed, 5)
x.set(\panWidth, 1)
~~~~~~~

### Custom built granular synthesis

Having explored some features of granular synthesis above, the best way to really understand what granular synthesis is would be to make our own granular synth engine that spawns grains of some sort according to our own rate, pitch, wave form, and envelope. 

In the examples above we have continued the chapter on Buffers and used sampled sound as the source of our granular synthesis. Here below we will explore the technique with simpler waveforms, such as the sine wave.

{line-numbers=off}
~~~~~~~
SynthDef(\sineGrain, { arg freq=800, amp=0.4, dur=0.1, pan=0;
	var signal, env;
	// A Gaussian curve envelope that's released from the server after playback
	env = EnvGen.kr(Env.sine(dur, amp), doneAction: 2);
	signal = FSinOsc.ar(freq, 0, env);
	OffsetOut.ar(0, Pan2.ar(signal, pan)); 
}).add;

Synth(\sineGrain, [\freq, 500, \dur, 0.05]) // 50 ms grain duration

// we can then trigger 1000 grains, one every 10 ms
(
Task({
   1000.do({ 
   		Synth(\sineGrain, 
			[\freq, rrand(440, 1600), // 
			\amp, rrand(0.1,0.3),
			\dur, rrand(0.02, 0.1)
			]);
		0.01.wait;
	});
}).start;
)
~~~~~~~

If our grains all have the same pitch, we should be able to generate a continuous sine wave out of the grains as they will be overlapping as shown in this image

[image]

{line-numbers=off}
~~~~~~~
Task({
   1000.do({ 
   		Synth(\sineGrain, 
			[\freq, 440,
			\amp, 0.4,
			\dur, 0.1
			]);
		0.05.wait; // density
	});
}).start;
~~~~~~~

But the sound is not perfectly continuous. This is because when we create a Synth, it is being sent as quickly as possible to the server. As the language-synth communication is *asynchronous* there might be slight time differences in the time it takes to send the OSC message over to the server, and this causes the fluctuation. We therefore need to timestamp our messages and it can be done either through messaging style communication with the server, or using s.bind (which makes an OSC bundle under the hood and sends a time stamped OSC message to the server).

{line-numbers=off}
~~~~~~~
Task({
   1000.do({ 
		s.sendBundle(0.2, 
			["/s_new", \sineGrain, x = s.nextNodeID, 0, 1], 
			["/n_set", x, \freq, 440, \amp, 0.4, \dur, 0.1]
		);
		0.05.wait; // density
	});
}).start;

// Or simply (and probably more readably)
Task({
   1000.do({
		s.bind{
			Synth(\sineGrain, 
				[\freq, 440,
				\amp, 0.4,
				\dur, 0.1
			]);
		};
		0.05.wait; // density
	});
}).start;
~~~~~~~

There can be different envelopes in the grains. Here we use a Perc envelope:

{line-numbers=off}
~~~~~~~
SynthDef(\sineGrainWPercEnv, { arg freq = 800, amp = 0.1, envdur = 0.1, pan=0;
	var signal;
	signal = FSinOsc.ar(freq, 0, EnvGen.kr(Env.perc(0.001, envdur), doneAction: 2)*amp);
	OffsetOut.ar(0, Pan2.ar(signal, pan)); 
}).add;

Task({
   1000.do({
		s.bind{
			Synth(\sineGrainWPercEnv, 
				[\freq, rrand(1300, 4000),
				\amp, rrand(0.1, 0.2),
				\envdur, rrand(0.1, 0.2),
				\pan, 1.0.rand2
			]);
		};
		0.01.wait; // density
	});
}).start;

// Or doing the same using the Pbind Pattern
Pbind(
	\instrument, \sineGrainWPercEnv,
	\freq, Pfunc({rrand(1300, 4000)}),
	\amp, Pfunc({rrand(0.1, 0.2)}),
	\envdur, Pfunc({rrand(0.1, 0.2)}),
	\dur, 0.01, // density
	\pan, Pfunc({1.0.rand2})
).play;
~~~~~~~

The two examples above serve as a good explanation of how Patterns and Tasks work. We've got the same SynthDef, same arguments, but Patterns do operate with default keywords (like \instrument, \freq, \amp, and \dur). We therefore had to make sure that our envelope argument was not called \dur, since Pbind uses that to control the density (or the time it takes until the next event is fired - so "\dur, 0.01" in the pattern is the same as "0.01.wait" in the Task)

{line-numbers=off}
~~~~~~~
Pbind(
	\instrument, \sineGrainWPercEnv,
	\freq, Pseq([1000, 2000, 4000], inf), // try to add 3000 in here
	\amp, Pfunc({rrand(0.1, 0.2)}),
	\envdur, Pseq([0.01, 0.02, 0.04], inf),
	\dur, Pseq([0.01, 0.02, 0.04], inf), // density
	\pan, Pseq([0.9, -0.9],inf)
).play;
~~~~~~~

Finally, let's try this out with a buffer. 

{line-numbers=off}
~~~~~~~
b = Buffer.read(s, Platform.userAppSupportDir+/+"sounds/a11wlk01-44_1.aiff");

SynthDef(\bufGrain,{ arg out = 0, buffer, rate=1.0, amp = 0.1, dur = 0.1, startPos=0;
	var signal;
	signal = PlayBuf.ar(1, buffer, rate, 1, startPos) * EnvGen.kr(Env.sine(dur, amp), doneAction: 2);
	OffsetOut.ar(out, signal ! 2)
}).add;

Synth(\bufGrain, [\buffer, b]); // try it

Task({
   1000.do({ arg i;
   		Synth(\bufGrain, 
			[\buffer, b,
   			\rate, rrand(0.8, 1.2),
			\amp, rrand(0.05,0.2),
			\dur, rrand(0.06, 0.1),
			\startPos, i*100 // jumping 100 samples per grain
		]);
		0.01.wait;
	});
}).start;
~~~~~~~

## Concatenative Synthesis

Concatenative synthesis is a rather recent technique of data-driven synthesis, where source sounds are analysed into a database, segmented into *units*, but then an target sound (for example live audio input) is analysed and matched with the closest unit in the database which is then played. This is done at a very granular level, prompting Zils and Pachet to call the technique *musaicing*, from *mu*sical mo*saicing*, as it enables the synthesis of a coherent sound at a macro level that is built up of smaller units of sound, just like in traditional mosaics. The technique is therefore quite related to granular synthesis in the sense that a macro-sound is built out of micro-sounds.
The technique can be quite complex to work with as users might have to analyse and build up a database of source sounds. However, people have built plugins and classes in SuperCollider that help with this purpose and in this section here we will explore some of the work done in this area by Nick Collins, a long time SuperCollider user and developer.



b = Buffer.read(s,Platform.userAppSupportDir+/+"sounds/a11wlk01.wav");


{Concat2.ar(SoundIn.ar(0),PlayBuf.ar(1, b, 1, loop:1),1.0,1.0,1.0,0.1,0,0.0,1.0,0.0,0.0)}.play

// mouse X used control the match length
{Concat2.ar(SoundIn.ar(0),PlayBuf.ar(1, b, 1, loop:1),1.0,1.0,1.0,MouseX.kr(0.0,0.1),0,1.0,0.0,1.0,1.0)}.play


