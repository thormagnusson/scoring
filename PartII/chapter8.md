# Envelopes and Shaping Sounds

In both analog and digital synthesis, we typically operate with sound sources that are constantly running - whether those are analog oscillators or digital unit generators. This is great fun of course and we can delight in altering parameters by turning knobs or or setting control values, sculpting the sound we are after. However, this sound is not very musical. Hardly any musical instruments can have infinite sound, and in instrumental sounds we typically get an initial burst of energy, the sound then reaches some sort of equilibrium until it fades out.
 
The way we shape these sounds in both analog and digital synthesis is to use so-called "envelopes." They wrap around our sound and give it the desired shape we're after. Most people have for example heard about the ADSR envelope (where the shape is Attack, Decay, Sustain, and Release) which is one of the available envelopes in SuperCollider:

![The shape of an ADSR envelope](images/ch8_adsr.png)

Envelopes in SuperCollider come in two types, sustaining (un-timed) and non-sustaining (timed) envelopes. A gate is a trigger (a positive number) that holds the envelope open until it gets a message to close it (such as 0 or less). This is like a finger pressing down a key on a MIDI keyboard. If we were using an ADSR envelope, when the finger presses the key, we would run the A (attack) and the D (decay), but then the S (sustain) would last as long as the finger is pressed. On R (release), when the finger releases the key, the R argument defines how long it takes for the sound to fade out. Synths with gated envelopes are can therefore be of un-definite time, i.e., its time is not set at the point of initialising the synth.

However, using a non-gated envelope, or a timed one, we set the duration of the sound at the time of triggering the synth. Here we don't need to use a gate to trigger and release a synth.

## Envelope types

Envelopes are powerful as we can define precisely the shape of a sound. This could be the amplitude of a sound, but it could also be a definition of frequency, filter cutoff, and so on. Let's look at a few common envelope types in SuperCollider:

{line-numbers=off}
~~~~~~~
Env.linen(1, 2, 3, 0.6).test.plot;
Env.triangle(1, 1).test.plot;
Env.sine(1, 1).test.plot;
Env.perc(0.05, 1, 1, -4).test.plot;
Env.adsr(0.2, 0.2, 0.5, 1, 1, 1).test.plot;
Env.asr(0.2, 0.5, 1, 1).test.plot;
Env.cutoff(1, 1).test(2).plot;
// using .new you can define your own envelope with as many points as you like
Env.new([0, 1, 0.3, 0.8, 0], [2, 3, 1, 4],'sine').test.plot;
Env.new([0,1, 0.3, 0.8, 0], [2, 3, 1, 4],'linear').test.plot;
Env.new({1.0.rand}!10, {2.0.rand}!9).test.plot;
Env.new({1.0.rand}!100, {2.0.rand}!99).test.plot;
~~~~~~~

Different sounds require different envelopes. For example, if we wanted to synthesise a snare sound, we might choose to use the .perc method of Env. 

{line-numbers=off}
~~~~~~~
{ LPF.ar(WhiteNoise.ar(0.5), 2000) * EnvGen.ar(Env.perc(0.001, 0.5)) ! 2 }.play

// And more bespoke envelopes can be created with the .new method:
{ Saw.ar(EnvGen.ar(Env.sine(0.3).range(140, 120))) * EnvGen.ar(Env.new([0, 1, 0, 0.5, 0], [0.3, 0, 0.1,0])) ! 2 }.play

// Note that above we are using a .sine envelope to modulate the frequency argument of the Saw UGen.
~~~~~~~

Envelopes define points in time that have a target value, duration and shape. So we can define the value, length and shape of each of the nodes. The .new method expects arrays for the value, duration and shape arguments. This can be very useful, as through a very simple syntax you can create complex transitions of value through time:

{line-numbers=off}
~~~~~~~
Env.new([0, 1, 0.5, 1, 0], [1, 2, 3, 2], \welch).plot;
Env.new([0, 1, 0.5, 1, 0], [1, 2, 3, 2], \step).plot;
Env.new([0, 1, 0.5, 1, 0], [1, 2, 3, 2], \sqr).plot;
Env.new([0, 1, 0.5, 1, 0], [1, 2, 3, 2], [2, 0, 5, 3]).plot;
Env.new([0, 1, 0.5, 1, 0], [1, 2, 3, 2], [0, 0, 0, 0]).plot;
Env.new([0, 1, 0.5, 1, 0], [1, 2, 3, 2], [5, 5, 5, 5]).plot;
Env.new([0, 1, 0.5, 1, 0], [1, 2, 3, 2], [20, -20, 20, 20]).plot;
~~~~~~~

The last array defines the curve where 0 is linear, positive number curves the segment up, and a negative number curves it down. Check the Env documentation for further explanation.


## The EnvGen - Envelope Generator

The envelope itself does nothing. It is simply a description of a form; of values in time and the shape of the line between those values. If we want to *apply* this envelope to a signal, we need to use the EnvGen UGen to play the envelope within a synth graph. You note that the EnvGen has an .ar or a .kr argument, so it works either in audio rate or control rate. The envelope arguments are the following:

{line-numbers=off}
~~~~~~~
EnvGen.ar(envelope, gate, levelScale, levelBias, timeScale, doneAction)
~~~~~~~

where the first argument is the envelope type (for example Env.perc(0.1, 1)), the second argument is the gate (not used with timed envelopes, but the default of the gate argument is 1, so it triggers the synth), the third argument is levelScale, which can scale the levels (such as amplitude) of the envelope, the fourth is levelBias which offsets the envelope's breakpoints, the fifth is timeScale, which can shorten or stretch the envelope (so a second long Env.sine(1), could become 10 second long), and finally we have the doneAction, but this defines what will happen to the synth instance after the envelope has done its job. 

### doneActions

The doneActions are an important aspect of how the SC-server works. One of the key strengths of SuperCollider is how a synth can be created and removed very effectively, making it useful for granular synthesis, or playback of notes. Here a grain or a note can be a synth that exists for 20 milliseconds or 20 minutes. Users of data flow languages, such as Pure Data, will appreciate how useful this is, as synths can be spawned at wish, and don't need to be hard wired beforehand. 

When the synth has exceeded its lifetime through the function of the envelope it will typically become silent. However, we don't want to pile synths up after they have played, but rather free the server of them. Unused synths will still run, use up processing power (CPU), and eventually cause some distortion in the sound; for example, if hundreds of synths have not been freed from the server and are still using CPU.
 
The doneActions are the following:
	
* 0 - Do nothing when the envelope has ended.
* 1 - Pause the synth running, it is still resident.
* 2 - Remove the synth and deallocate it.
* 3 - Remove and deallocate both this synth and the preceding node.
* 4 - Remove and deallocate both this synth and the following node.
* 5 - Same as 3. If the preceding node is a group then free all members of the group.
* 6 - Same as 4. If the following node is a group then free all members of the group.
* 7 - Same as 3. If the synth is part of a group, free all preceding nodes in the group.
* 8 - Same as 4. If the synth is part of a group, free all following nodes in the group.
* 9 - Same as 2, but pause the preceding node.
* 10 - Same as 2, but pause the following node.
* 11 - Same as 2, but if the preceding node is a group then free its synths.
* 12 - Same as 2, but if the following node is a group then free its synths.
* 13 - Frees the synth and all preceding and following nodes.

The doneActions are used in the EnvGen UGen all the time and it is important not to forget it. However there are other UGens in SuperCollider that also can free their enclosing synth when an event has happened - such as finishing playing a sample buffer. The other UGens are the following:

* PlayBuf and RecordBuf - doneAction when the buffer has been played or recorded.
* Line and XLine - doneAction when a line has ended.
* Linen - doneAction when the envelope is finished.
* LFGauss - doneAction after the completion of a cycle.
* DemandEnvGen - Similar to EnvGen.
* DetectSilence - doneAction when the UGen detects silence below a threshold.
* Duty and TDuty - doneAction evaluated when a duty stream ends.


{line-numbers=off}
~~~~~~~
SynthDef(\sine, {arg freq=440, amp=0.1, gate=1, dA = 2;
	var signal, env;
	signal = SinOsc.ar(freq, 0, amp);
	env = EnvGen.ar(Env.adsr(0.2, 0.2, 0.5, 0.3, 1, 1), gate, doneAction: dA);
	Out.ar(0, Pan2.ar(signal * env, 0));
}).add

s.plotTree // watch the nodes appearing on the server tree
~~~~~~~

In the examples below, when you add a node, it is always added at the top of the node tree. This is how SC server does it by default. Synths can be added anywhere in the three though, but that will be discussed later in the chapter on busses, nodes and groups. [xxx, 15. ]

{line-numbers=off}
~~~~~~~
// doneAction = 0
a = Synth(\sine, [\dA, 0])
a.release
a.set(\gate, 1)

// doneAction = 1
a = Synth(\sine, [\dA, 1])
a.release
a.set(\gate, 1)
a.run(true)

// doneAction = 2
a = Synth(\sine, [\dA, 2])
a.release
a.set(\gate, 1) // it's gone! (see server synth count)

// doneAction = 3
a = Synth(\sine, [\dA, 3])
b = Synth(\sine, [\freq, 660, \dA, 3])
a.release

// doneAction = 3
a = Synth(\sine, [\dA, 3])
b = Synth(\sine, [\freq, 660, \dA, 3], addAction:\addToTail)
b.release

// doneAction = 3
a = Synth(\sine, [\freq, 440, \dA, 3])
b = Synth(\sine, [\freq, 660, \dA, 3])
c = Synth(\sine, [\freq, 880, \dA, 3])
b.release // will release b and c

// doneAction = 4
a = Synth(\sine, [\freq, 440, \dA, 4])
b = Synth(\sine, [\freq, 660, \dA, 4])
c = Synth(\sine, [\freq, 880, \dA, 4])
b.release // will release a and b

// doneAction = 5
g = Group.new;
a = Synth(\sine, [\freq, 440, \dA, 0], target:g)
b = Synth(\sine, [\freq, 660, \dA, 0], target:g)
c = Synth(\sine, [\freq, 880, \dA, 5])
c.release // will only free c (itself)

// doneAction = 5
g = Group.new;
a = Synth(\sine, [\freq, 440, \dA, 0], target:g)
b = Synth(\sine, [\freq, 660, \dA, 0], target:g)
c = Synth(\sine, [\freq, 880, \dA, 5], addAction:\addToTail)
c.release // will free itself and the preceding group

// doneAction = 6
g = Group.new;
a = Synth(\sine, [\freq, 440, \dA, 0], target:g)
b = Synth(\sine, [\freq, 660, \dA, 0], target:g)
c = Synth(\sine, [\freq, 880, \dA, 6])
c.release // will free itself and the following group

// doneAction = 7
g = Group.new;
a = Synth(\sine, [\freq, 440, \dA, 0], target:g )
b = Synth(\sine, [\freq, 660, \dA, 0], target:g)
c = Synth(\sine, [\freq, 880, \dA, 7], target:g)
d = Synth(\sine, [\freq, 1100, \dA, 0], target:g)
e = Synth(\sine, [\freq, 1300, \dA, 0], target:g)
c.release // will free itself and preceding nodes in a group

// doneAction = 8
g = Group.new;
a = Synth(\sine, [\freq, 440, \dA, 0], target:g)
b = Synth(\sine, [\freq, 660, \dA, 0], target:g)
c = Synth(\sine, [\freq, 880, \dA, 8], target:g)
d = Synth(\sine, [\freq, 1100, \dA, 0], target:g)
e = Synth(\sine, [\freq, 1300, \dA, 0], target:g)
c.release // will free itself and preceding nodes in a group

// doneAction = 9
a = Synth(\sine, [\freq, 440, \dA, 9])
b = Synth(\sine, [\freq, 660, \dA, 0])
a.release // will free itself and pause the preceding node
b.run(true) // it was only paused

// doneAction = 10
g = Group.new;
a = Synth(\sine, [\freq, 440, \dA, 0], target:g)
b = Synth(\sine, [\freq, 660, \dA, 0], target:g)
c = Synth(\sine, [\freq, 880, \dA, 10])
d = Synth(\sine, [\freq, 1100, \dA, 0])
c.release // will free itself and pause following nodes in a group
g.run(true) // it was only paused

// doneAction = 11
a = Synth(\sine, [\freq, 440, \dA, 11])
b = Synth(\sine, [\freq, 660, \dA, 0])
a.release // will free itself and the preceding node

// doneAction = 12
g = Group.new;
a = Synth(\sine, [\freq, 440, \dA, 0], target:g)
b = Synth(\sine, [\freq, 660, \dA, 0], target:g)
c = Synth(\sine, [\freq, 880, \dA, 12])
d = Synth(\sine, [\freq, 1100, \dA, 0])
c.release // will free itself and the following node

// doneAction = 13
g = Group.new;
a = Synth(\sine, [\freq, 440, \dA, 0], target:g)
b = Synth(\sine, [\freq, 660, \dA, 0], target:g)
c = Synth(\sine, [\freq, 880, \dA, 13])
d = Synth(\sine, [\freq, 1100, \dA, 0])
x = Synth(\sine, [\freq, 2100, \dA, 0])
e = Synth(\sine, [\freq, 1300, \dA, 0])
c.release // will free itself and the following node
~~~~~~~

## Triggers and Gates


The difference between a gated and timed envelope has become clear in the above examples, but to put it in very simple terms, think of the piano as having a timed envelope (as the note dies after a while), but the organ as having a gated envelope (as the note only stops when the key is released). For user input it is good to be able to keep the envelope open as long as the user wants and free it at some event, such as releasing a key (or a person exiting a room in a sound installation).

### Gates

Gates are typically used to start a sound that contains an envelope of some sort. They 'open up' for a flow values to pass through for a period of time (timed or untimed). When a gate closes, it typically runs the *release* part of the envelope used.

{line-numbers=off}
~~~~~~~
d = Synth(\sine, [\freq, 1100]) // key down
d.release // key up

// compare with
d = Synth(\sine, [\freq, 840]) // key down
d.free // kill immediately

// gate holds the EnvGen open. Here using Dust (random impulses) to trigger a new envelope
{EnvGen.ar(Env.adsr(0.001, 0.8, 1, 1), Dust.ar(1)) *  Saw.ar(55)!2}.play

// Here using Impulse (periodic impulses)
{EnvGen.ar(Env.adsr(0.001, 0.8, 1, 1), Impulse.ar(2)) *  SinOsc.ar(LFNoise0.ar(2).range(200, 1000))!2}.play

// With a doneAction: 2 we kill the synth after the first envelope
{EnvGen.ar(Env.adsr(0.001, 0.8, 0.1, 0.1), Impulse.ar(2), doneAction:2) *  SinOsc.ar(2222)!2}.play

// but if we increase the release time of the envelope, it will be retriggered before the doneAction can kill it
{EnvGen.ar(Env.adsr(0.001, 0.8, 0.1, 1), Impulse.ar(2), doneAction:2) *  SinOsc.ar(1444)!2}.play
~~~~~~~

Triggers are similar to gates, they start a process, but they do not have the *release* function Gates have. So they are used to trigger envelopes.

trigger rate - Arguments that begin with "t_" (e.g. t_trig), or that are specified as \tr in the def's rates argument (see below), will be made as a TrigControl. Setting the argument will create a control-rate impulse at the set value. This is useful for triggers.

### Triggers

In the example above we saw how Dust and Impulse could be used to trigger an envelope. The trigger can be set from everywhere (code, GUI, system, etc) but we need to use "t_" in front of trigger arguments.

{line-numbers=off}
~~~~~~~
(
a = { arg t_gate = 1;
	var freq;
	freq = EnvGen.kr(Env.new([200, 200, 800], [0, 1.6]), t_gate);
     SinOsc.ar(freq, 0, 0.2) ! 2 
}.play;
)

a.set(\t_gate, 1)  // try to evaluate this line repeatedly
a.free // if you observe the server window you see this synth disappearing

(
a = { arg t_gate = 1;
	var env;
	env = EnvGen.kr(Env.adsr, t_gate);
     SinOsc.ar(888, 0, 1 * env) ! 2 
}.play;
)

a.set(\t_gate, 1)  // repeat this
a.free // free the synth (since it didn't have a doneAction:2)

// If you are curious about what doneAction:2 would have done, try this:
(
a = { arg t_gate = 1;
	var env;
	env = EnvGen.kr(Env.adsr, t_gate, doneAction:2);
     SinOsc.ar(888, 0, 1 * env) ! 2 
}.play;
)

a.set(\t_gate, 1)  // why does this line not retrigger the synth?
// Now try the same with doneAction:0
~~~~~~~

If you want to keep the same synth on the server and trigger it from another process than the synthesis control parameter process you can use gates and triggers for the envelope. Use doneAction:0 to keep the synth on the server before or after the envelope is triggered.

Let's turn the examples above into SynthDefs and explore the concept of gates:

{line-numbers=off}
~~~~~~~
SynthDef(\trigtest, {arg freq, amp, dur=1, gate;
	var signal, env;
	env = EnvGen.ar(Env.adsr(0.01, dur, amp, 0.7), gate, doneAction:0); 
	signal = SinOsc.ar(freq) * env;
	Out.ar(0, signal);
}).add

a = Synth(\trigtest, [\freq, 333, \amp, 1, \gate, 0]) // gate is 0, no sound
a.set(\gate, 1)
a.set(\gate, 0)

// the synth is still running, even if it is silent
a.set(\freq, 788) // change the frequency

a.set(\gate, 1)
a.set(\gate, 0)
~~~~~~~

The example below does the same, but here with a fixed time envelope. Since that envelope *finishes* when it is done, it does not work with gates. We need a trigger to trigger it back to life.

{line-numbers=off}
~~~~~~~
// here we use a t_trig to retrigger the synth
SynthDef(\trigtest2, {arg freq, amp, dur=1, t_trig;
	var signal, env;
	env = EnvGen.ar(Env.perc(0.01, dur, amp), t_trig, doneAction:0); 
	signal = SinOsc.ar(freq) * env;
	Out.ar(0, signal);
}).add

a = Synth(\trigtest2, [\freq, 333, \amp, 1, \t_trig, 1])

a.set(\freq, 788)
a.set(\t_trig, 1);
a.set(\amp, 0.28)
a.set(\t_trig, 1);

a.set(\freq, 588)
a.set(\t_trig, 1);
a.set(\amp, 0.8)
a.set(\t_trig, 1);
~~~~~~~

Exercise: Explore the difference between a gate and a trigger.

## MIDI Keyboard Example

The techniques we've been exploring above are useful when creating user interfaces for your synth. As an example we could create a synth definition to be controlled by a MIDI controller. Other usage could be networked communication, input from other software, or running musical patterns within SuperCollider itself. In the example below we build upon the example we did in chapter 4, but here we add pitch bend and vibrato.

{line-numbers=off}
~~~~~~~
MIDIIn.connectAll; // we connect all the incoming devices
MIDIFunc.noteOn({arg ...x; x.postln; }); // we post all the args

//First we create a synth definition for this example:
SynthDef(\moog, {arg freq=440, amp=1, gate=1, pitchBend=0, cutoff=20, vibrato=0;
	var signal, env;
	signal = LPF.ar(VarSaw.ar([freq, freq+2]+pitchBend+SinOsc.ar(vibrato, 0, 1, 1), 0, XLine.ar(0.7, 0.9, 0.13)), (cutoff * freq).min(16000));
	env = EnvGen.ar(Env.adsr(0), gate, levelScale: amp, doneAction:2);
	Out.ar(0, signal*env);
}).add;

a = Array.fill(127, { nil }); // create an array of nils, where the Synths will live
g = Group.new; // we create a Group to be able to set cutoff of all active notes
c = 6;
MIDIdef.noteOn(\myOndef, {arg vel, key, channel, device; 
	// we use the key as index into the array as well
	a[key] = Synth(\moog, [\freq, key.midicps, \amp, vel/127, \cutoff, 10], target:g);
});
MIDIdef.noteOff(\myOffdef, {arg vel, key, channel, device; 
	a[key].release;
	a[key] = nil; // we put nil back in the array as we use it in the if-statements below
});

MIDIdef.cc(\myPitchBend, { arg val; 
	c=val.linlin(0, 127, -10, 10); 
	"Pitch Bend : ".post; val.postln;
	a.do({arg synth; 
		if( synth != nil , { synth.set(\pitchBend, val ) }); 
	});	
});

MIDIdef.bend(\myVibrato, { arg val; 
	c=val.linlin(0, 127, 1, 20); 
	"Vibrato : ".post; val.postln;
	a.do({arg synth; 
		if( synth != nil , { synth.set(\vibrato, c ) }); 
	});	
});
~~~~~~~



