# ProxySpace and JIT lib

JIT lib, or Just in Time library, is a system that allows people to write Ugen Graphs (signal processing on the SC server) and rewrite them in real time. This is ideal for live coding, teaching, experimenting and all kinds of compositional work. 

## ProxySpace

In order to use the JIT lib you create a ProxySpace which becomes the Environment or reference space for the synths that will live on it.

p = ProxySpace.new;
p.fadeTime = 3; // fadeTime is the time of crossfading

p[\snd].play; // we create a reference \snd in the environment
p[\snd] = { SinOsc.ar(440) };
p[\snd] = { Saw.ar(333, 0.4) };

p[\ctrl] = {LFSaw.ar(2)}
p[\snd] = {WhiteNoise.ar(p[\ctrl])}
p[\ctrl] = {LFNoise2.ar(2)}
p[\snd] = {Saw.ar([p[\ctrl]*1000, p[\ctrl]*1000+1], 0.3)}
p[\ctrl] = {LFSaw.ar(0.4)}

Or from the ProxySpace examples file. Here we find the "~" symbol used to reference signals in the dictionary:

~lfo = { LFNoise2.kr(30, 300, 500) };
~out = { SinOsc.ar(~lfo.kr, 0, 0.15)  };
~out = { SinOsc.ar(~lfo.kr * [1, 1.2], 0, 0.1) * Pulse.ar(~lfo.kr * [0.1, 0.125], 0.5) };
~lfo = { LFNoise1.kr(30, 40) + SinOsc.kr(0.1, 0, 200, 500) };
~out = { SinOsc.ar(~lfo.kr * [1, 1.2], 0, 0.1)  };
~lfo = 410;




## Ndef 

## Tdef




