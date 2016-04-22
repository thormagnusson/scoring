# Busses, Nodes, Groups and Signal Flow


// 	1) Busses in SC (Audio and Control Busses)
// 	2) Nodes
// 	3) Groups




## Busses in SC (Audio and Control Busses)

What are Busses? They are virtual placeholders of signals. A good description is to be found in the Server-Architecture helpfile:

"Audio Buses
Synths send audio signals to each other via a single global array of audio buses.  Audio buses are indexed by integers beginning with zero. Using buses rather than connecting synths to each other directly allows synths to connect themselves to the community of other synths without having to know anything about them specifically. The lowest numbered buses get written to the audio hardware outputs. Immediately following the output buses are the input buses, read from the audio hardware inputs. The number of bus channels defined as inputs and outputs do not have to match that of the hardware.

Control Buses
Synths can send control signals to each other via a single global array of control buses.  Buses are indexed by integers beginning with zero."


If you look at the source file of ServerOptions, you will see that there are default number of audio and control busses assigned to the server on booting. You can change these values, of course:
	var <>numAudioBusChannels=128;
	var <>numControlBusChannels=4096;
	var <>numInputBusChannels=8;
	var <>numOutputBusChannels=8;
We see that we've got 128 audio busses and 4096 control busses. This should be more than enough in most cases, but if you need more you can:
a) question why you need more! Are you designing your program wrongly?
b) change the number in the ServerOptions file and recompile.

We also see that by default we have 8 output and 8 input busses. This means that in this setting the 8th Audio bus is actually the 1st input channel. Change this to fit your soundcard if you want. 

Busses are not exactly the same as audio channels. Channels as we normally think of them are physical channels as in a mixer or a sound card, but a Bus is rather like an abstract representation of a channel. Thus a bus can be mono or stereo or even 5 channel, depending on your needs.

### Audio Busses (where signals run on audio rate (such as 44100 times per second))

// We send audio out on bus 0

(
SynthDef(\bustest, {arg outbus=0, freq=440;
	Out.ar(outbus, SinOsc.ar(freq, 0, 0.3));
}).add
)

a = Synth(\bustest, [\outbus, 0]) // left speaker
b = Synth(\bustest, [\outbus, 1]) // right speaker
c = Synth(\bustest, [\outbus, 2]) // channel 3?

// now we free a and b
a.free; b.free; 

// but c is still running on bus 2 - we just can't hear it (assuming you're in stereo)

// so we create a synthdef that can listen to any bus and output on any bus
(
SynthDef(\bustest2, {arg inbus=10, outbus=0;
	Out.ar(outbus, In.ar(inbus, 1));
}).add
)

// and we listen to bus 2 and output on bus 0. - don't worry about addAction now.
d = Synth(\bustest2, [\inbus, 2, \outbus, 0], addAction:\addToTail);

// If you were wondering about the comment on inbusses and outbusses, you can try
// to listen to the audio in bus (by default on bus 8) (if you have an active mic that is)

d = Synth(\bustest2, [\inbus, 8, \outbus, 0], addAction:\addToTail);


### Control Busses (where signals run on control rate (such as 689 times per second))

A control bus can be mapped to control values in many synths. Let's make a control synth that maps the freq value of the synth above.

(
SynthDef(\lfo, {arg ctrlbus = 2, freq=4, mul=100;
        Out.kr(ctrlbus, SinOsc.kr(freq, 0, mul: mul, add: 200)); // note the .kr
}).add;
)

// we create our synth
a = Synth(\bustest);

// we make a control bus that will be controlling the freq of our synth
b = Bus.control(s, 1);
b.value = 200;
// then we map the frequency of that bus to the freq parameter of the synth
a.map(\freq, b.index);
// and we can try to put different values into the control bus
b.value = 600;
b.value = 400;

// but the values of the control bus can by dynamic
c = Synth(\lfo, [\ctrlbus, b.index]);
c.set(\freq, 7);
c.set(\freq, 2);
c.set(\mul, 200);

// let's change the lfo to a LFSaw

(
SynthDef(\lfosaw, {arg ctrlbus = 2, freq=4, mul=100;
        Out.kr(ctrlbus, LFSaw.kr(freq, 0, mul: mul, add: 200)); // note the .kr
}).add;
)

c.free;
d = Synth(\lfosaw, [\ctrlbus, b.index]);
d.set(\freq, 7);
d.set(\freq, 2);
d.set(\mul, 200);


// This way you can really plug synths into each other just like on an
// old fashioned modular synth. 
// For a different take on modular coding, check the JIT lib.



## Nodes

We have already been using nodes in this tutorial. Creating a synth like this:
	a = Synth(\bustest);
is creating a node. We can then set the frequency of the node
	a.set(\freq, 880);
or just free it:
	a.free;
	
Nodes live on busses. They can be seen as a mythic monster with a head and a tail facing upwards that eats audio flowing down. This monster (the node) can take audio in from one bus and output into another bus. (Your SynthDef handle's that). The audio runs from the head to the tail. You can put your synths in front of the monster (where the sound will run through it) or at the tail (where it will receive the signal that runs through it).

When you start SC there is a default group that receives all nodes
s.queryAllNodes; // Note the RootNode (ID 0) and the default Group (ID 1)

By default synths are added to the HEAD of a group (in this instance the default group)

So in the following program you don't hear anything (but see the 2 synths on the server window)
(
{Out.ar(2, PinkNoise.ar(0.3)!2)}.play;
{In.ar(2, 2)}.play // this is added to the head of the bus (but the PinkNoise is below)
)

But now you hear: (because the sound is put on to the head AFTER the listener (In))
(
{In.ar(2, 2)}.play;
{Out.ar(2, PinkNoise.ar(0.3)!2)}.play;
)

Or better: simply add the In listener to the tail of the default group and we hear:
(
{Out.ar(2, PinkNoise.ar(0.3)!2)}.play;
{In.ar(2, 2)}.play(addAction:\addToTail)
)

This is the meaning of \addToHead, \addToTail, \addAbove, and \addBelow.

And if we keep these synths running we can see that they have been added to the Group (default)
S
.queryAllNodes; 

{Out.ar(2, SinOsc.ar(200)!2)}.play; // adding to head by default

s.queryAllNodes; 


Here is a practical example using a reverb and a delay for a snare

(
SynthDef(\reverb12, {arg inbus=0, outbus=0, predelay=0.048, combdecay=5, allpassdecay=1, revVol=0.31;
	var sig, y, z;
	sig = In.ar(inbus, 2); 
	z = DelayN.ar(sig, 0.1, predelay); // max 100 ms predelay
	y = Mix.ar(Array.fill(7,{ CombL.ar(z, 0.05, rrand(0.03, 0.05), combdecay) })); 
	6.do({ y = AllpassN.ar(y, 0.050, rrand(0.03, 0.05), allpassdecay) });
	Out.ar(outbus, sig + (y * revVol)); 
}).add; 

SynthDef(\delay12, {arg inbus=0, outbus=0, maxdelaytime=6, delaytime=0.3, decaytime=2.31;
	var sig, y, z;
	sig = In.ar(inbus, 2); 
	sig = CombN.ar(sig, maxdelaytime, delaytime, decaytime);
	Out.ar(outbus, sig); 
}).add; 

SynthDef(\snare12, { arg out=0, tempo=2;
	var snare, base, hihat;
	tempo = Impulse.ar(tempo); // for a drunk drummer replace Impulse with Dust !!!

	snare = 	WhiteNoise.ar(Decay2.ar(PulseDivider.ar(tempo, 4, 2), 0.005, 0.5));

	Out.ar(out, snare * 0.4 !2)
}).add;
)

A snare on outbus 0 - no effects
a = Synth(\snare12, [\outbus, 20]);

We create a reverb synth on audiobus 20 and delay on audiobus 22 (stereo signal)

b = Synth(\reverb12, [\inbus, 20, \outbus, 0]);
c = Synth(\delay12, [\inbus, 22, \outbus, 0]);

s.queryAllNodes; 

a.set(\outbus, 20)
a.moveBefore(b)
s.queryAllNodes; 

a.set(\outbus, 22)
a.moveBefore(c)
s.queryAllNodes; 


{Out.ar(20, AudioIn.ar(1))}.play(addAction:\addToHead) // we add audio in to the snaredrum

And we could add a synth AFTER the delay
a = Synth(\snare12, [\outbus, 22, \tempo, 4], addAction:\addToTail)

Or we add it BEFORE the delay
a = Synth(\snare12, [\outbus, 22, \tempo, 4], addAction:\addToHead)


## Groups

Groups can be useful if you are making complex things and you want to group certain things together. You can think of it like grouping in Photoshop (i.e. making a group that you can move around without having to move every line). For a good explanation of Groups, check Mark Polishook's tutorial in the distribution of SC

Group example (check the Group and Node helpfiles for more)

g = Group.new // we create a new group

And few synths that respond to the freq argument, but multiply it differently

{arg freq=333, out=0; Out.ar(out, SinOsc.ar(freq,0,0.12))}.play(g)
{arg freq=333, out=0; Out.ar(out, SinOsc.ar(freq*1.2,0,0.12))}.play(g)
{arg freq=333, out=0; Out.ar(out, SinOsc.ar(freq*1.4,0,0.12))}.play(g)

g.set(\freq, 255) // we change the frequency and ALL the synths get a new frequency
g.set(\out, 10) // we move the output to bus 10

s.queryAllNodes; 

Here we could try to listen to bus 10, but it's added to the head

{Out.ar(0, In.ar(10,1))}.play(g)
s.queryAllNodes; 

// so we explicitly add the synth to the tail 
{Out.ar(0, In.ar(10,1))}.play(g, addAction:\addToTail)
s.queryAllNodes; 

We see that we now have 5 synths in a Group (called g)

h = Group.new // we create a new group

{arg freq=333, out=0; Out.ar(out, SinOsc.ar(freq,0,0.12))}.play(h)
{arg freq=333, out=0; Out.ar(out, SinOsc.ar(freq*1.2,0,0.12))}.play(h)
{arg freq=333, out=0; Out.ar(out, SinOsc.ar(freq*1.4,0,0.12))}.play(h)

h.set(\freq, 255) // we change the frequency and ALL the synths get a new frequency
h.set(\freq, 955) // we change the frequency and ALL the synths get a new frequency

s.queryAllNodes; 

h.moveAfter(g) // we can move h (not that it matters here, but when making effects, it's useful)

s.queryAllNodes



