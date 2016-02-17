
# Subtractive Synthesis

Last chapter discussed additive synthesis. The idea is start with silence and add together partials and derive at the sound we are after. Subtractive synthesis works the opposite. We start with a rich sound - a broadband sound either rich in partials/harmonics or noise - and then *filter* the unwanted frequencies out. WhiteNoise and Saw waves are typical sound sources, as the noise has equal energy on all frequencies, but the saw wave has a naturally sounding harmonic structure with energy on every harmonic. 

## Noise Sources

The definition of noise is a signal that is aperiodic, i.e., there is no periodic repetition of some form in the signal. If there was such repetition, we would talk about a wave form and then a frequency of those repetitions. The frequency becomes pitch or musical notes. Not so in the world of noise: there are no repetitions that we can detect and thus we perceive it as the opposite of a signal; the antithesis of a meaning. Most of us remember the white noise of a dead analogue TV channel. Anyway, although noise might for some have negative connotations, it is a very useful musical element, in particular for synthesis as a rich input signal. 


    // WhiteNoise
    {WhiteNoise.ar(0.4)}.plot(1)
    {WhiteNoise.ar(0.4)}.play
    {WhiteNoise.ar(0.4)}.scope
    {WhiteNoise.ar(0.4)}.freqscope
    
    // PinkNoise 
    {PinkNoise.ar(1)}.plot(1)
    {PinkNoise.ar(1)}.play
    {PinkNoise.ar(1)}.freqscope
    
    // BrownNoise
    {BrownNoise.ar(1)}.plot(1)
    {BrownNoise.ar(1)}.play
    {BrownNoise.ar(1)}.freqscope

Take a look at the source file called Noise.sc (or hit Apple+Y on WhiteNoise). You will find lots of interesting noise generators. For example these:

    { Crackle.ar(XLine.kr(0.99, 2, 10), 0.4) }.freqscope.scope;

    { LFDNoise0.ar(XLine.kr(1000, 20000, 10), 0.1) }.freqscope.scope;
    
    { LFClipNoise.ar(XLine.kr(1000, 20000, 10), 0.1) }.freqscope.scope;
    
    // Impulse
    { Impulse.ar(80, 0.7) }.play
    { Impulse.ar(4, 0.7) }.play
    
    // Dust (random impulses)
    { Dust.ar(80) }.play
    { Dust.ar(4) }.play

We can not start to sculpt sound with the use of filters and envelopes. For example, what would this remind us of:

    {WhiteNoise.ar(1) * EnvGen.ar(Env.perc(0.001,0.3), doneAction:2)}.play

We can add a low pass filter (LPF) to the noise, so we cut off the high frequencies:

    {LPF.ar(WhiteNoise.ar(1), 3300) * EnvGen.ar(Env.perc(0.001,0.5), doneAction:2)}.play

And here we use mouse movements to control the cutoff frequency (the x-axis) and the envelope duration (y-axis):

    (
    fork{
    	100.do({
    		{LPF.ar(WhiteNoise.ar(1), MouseX.kr(200,20000, 1)) 
    			* EnvGen.ar(Env.perc(0.00001, MouseY.kr(1, 0.1, 1)), doneAction:2)}.play;
    		1.wait;
    	});
    }
    )

But what did that low pass filter do? LPF? HPF? These are filters that *pass through the low frequencies*, thus the name. A high pass filter will *pass through the high frequencies*. And a band pass filter will pass through the frequencies of a band that you specify. We can view the functionality of the low pass filter with the use of a frequency scope. Note also the quality parameter in the resonant low pass filter:

    {LPF.ar(WhiteNoise.ar(0.4), MouseX.kr(100, 20000).poll(20, "cutoff"))}.freqscope;
    {RLPF.ar(WhiteNoise.ar(0.4), MouseX.kr(100, 20000).poll(20, "cutoff"), MouseY.kr(0.01, 1).poll(20, "quality"))}.freqscope


## Filter Types

Filters are algorithms that are typically applied in the time domain of an audio signal. This, for example, means adding a delayed copy of the signal to itself. 

Here is a very primitive such filter:

    {
    var signal;
    var delaytime = MouseX.kr(0.000022675, 0.001); // from a sample 
    signal = Saw.ar(220, 0.5);
    d =  DelayC.ar(signal, 0.6, delaytime); 
    (signal + d).dup
    }.play

Let us try some of the filter UGens of SuperCollider:

    // low pass filter
    {LPF.ar(WhiteNoise.ar(0.4), MouseX.kr(40,20000,1)!2) }.play;
    
    // low pass filter with XLine
    {LPF.ar(WhiteNoise.ar(0.4), XLine.kr(40,20000, 3, doneAction:2)!2) }.play;
    
    // high pass filter
    {HPF.ar(WhiteNoise.ar(0.4), MouseX.kr(40,20000,1)!2) }.play;
    
    // band pass filter (the Q is controlled by the MouseY)
    {BPF.ar(WhiteNoise.ar(0.4), MouseX.kr(40,20000,1), MouseY.kr(0.01,1)!2) }.play;
    
    // Mid EQ filter attenuates or boosts a frequency band
    {MidEQ.ar(WhiteNoise.ar(0.024), MouseX.kr(40,20000,1), MouseY.kr(0.01,1), 24)!2 }.play;
    
    // what's happening here?
    {
    var signal = MidEQ.ar(WhiteNoise.ar(0.4), MouseX.kr(40,20000,1), MouseY.kr(0.01,1), 24);
    BPF.ar(signal, MouseX.kr(40,20000,1), MouseY.kr(0.01,1)) !2
    }.play;


### Resonating filters

A resonant filter does what is says on the tin, it *resonates* certain frequencies. The bandwidth of this resonance can vary, so with a WhiteNoise input, one could go from a very wide resonance (where the "quality" - the Q - of the filter is low), to a very narrow band resonance where the noise almost sounds like a sine wave. Let's explore this with WhiteNoise and a band pass filter:

    {BPF.ar(WhiteNoise.ar(0.4), MouseX.kr(100, 10000).poll(20, "cutoff"), MouseY.kr(0.01, 0.9999).poll(20, "rQ"))}.freqscope

Move your mouse around and explore how the Q factor, when increased, results in a narrower resonating bandwidth. 

In a low pass and high pass resonant filters, the energy at the cutoff frequency can be increased or decreased by setting the Q factor (or in SuperCollider, the reciprocal (inverse) of Q).

    // resonant low pass filter
    {RLPF.ar(
    	Saw.ar(222, 0.4), 
    	MouseX.kr(100, 12000).poll(20, "cutoff"), 
    	MouseY.kr(0.01, 0.9999).poll(20, "rQ")
    )}.freqscope;
    // resonant high pass filter
    {RHPF.ar(
    	Saw.ar(222, 0.4), 
    	MouseX.kr(100, 12000).poll(20, "cutoff"), 
    	MouseY.kr(0.01, 0.9999).poll(20, "rQ")
    )}.freqscope;

There are bespoke resonance filters in SuperCollider, such as Resonz, Ringz and Formant.

    // resonant filter
    { Resonz.ar(WhiteNoise.ar(0.5), MouseX.kr(40,20000,1), 0.1)!2 }.play

    // a short impulse won't resonate
    { Resonz.ar(Dust.ar(0.5), 2000, 0.1) }.play
    
    // for that we use Ringz
    { Ringz.ar(Dust.ar(2, 0.6), MouseX.kr(200,6000,1), 2) }.play
    
    // X is frequency and Y is ring time
    { Ringz.ar(Impulse.ar(4, 0, 0.3),  MouseX.kr(200,6000,1), MouseY.kr(0.04,6,1)) }.play
    
    { Ringz.ar(Impulse.ar(LFNoise2.ar(2).range(0.5, 4), 0, 0.3),  LFNoise2.ar(0.1).range(200,3000), LFNoise2.ar(2).range(0.04,6,1)) }.play
    
    { Mix.fill(10, {Ringz.ar(Impulse.ar(LFNoise2.ar(rrand(0.1, 1)).range(0.5, 1), 0, 0.1),  LFNoise2.ar(0.1).range(200,12000), LFNoise2.ar(2).range(0.04,6,1)) })}.play
    
    { Formlet.ar(Impulse.ar(4, 0.9), MouseX.kr(300,2000), 0.006, 0.1) }.play;
    
    { Formlet.ar(LFNoise0.ar(4, 0.2), MouseX.kr(300,2000), 0.006, 0.1) }.play;



## Klank and DynKlank

Just as Klang is a bank of fixed frequency oscillators, i.e., additive synthesis, Klank is a bank of fixed frequency resonators, where frequencies are subtracted of an input signal.

    { Ringz.ar(Dust.ar(3, 0.3), 440, 2) + Ringz.ar(Dust.ar(3, 0.3), 880, 2) }.play
    
    //  using only one Dust UGen to trigger all the filters:
    (
    { 
    var trigger, freq;
    trigger = Dust.ar(3, 0.3);
    freq = 440;
    Ringz.ar(trigger, 440, 2, 0.3) 		+ 
    Ringz.ar(trigger, freq*2, 2, 0.3) 	+ 
    Ringz.ar(trigger, freq*3, 2, 0.3) !2
    }.play
    )

    // but there is a better way:
    
    // Klank is a bank of resonators like Ringz, but the frequency is fixed. (there is DynKlank)
    
    { Klank.ar(`[[800, 1071, 1153, 1723], nil, [1, 1, 1, 1]], Impulse.ar(2, 0, 0.1)) }.play;
    
    // whitenoise input
    { Klank.ar(`[[440, 980, 1220, 1560], nil, [2, 2, 2, 2]], WhiteNoise.ar(0.005)) }.play;
    
    // AudioIn input
    { Klank.ar(`[[220, 440, 980, 1220], nil, [1, 1, 1, 1]], AudioIn.ar([1])*0.001) }.play;

Let's explore the DynKlank UGen. It does the same as Klank, but it allows us to change the values after the synth has been instantiated.

    { DynKlank.ar(`[[800, 1071, 1353, 1723], nil, [1, 1, 1, 1]], Dust.ar(8, 0.1)) }.play;
    
    { DynKlank.ar(`[[200, 671, 1153, 1723], nil, [1, 1, 1, 1]], PinkNoise.ar([0.007,0.007])) }.play;
    
    { DynKlank.ar(`[[200, 671, 1153, 1723]*XLine.ar(1, [1.2, 1.1, 1.3, 1.43], 5), nil, [1, 1, 1, 1]], PinkNoise.ar([0.007,0.007])) }.play;
    
    SynthDef(\dynklanks, {arg freqs = #[200, 671, 1153, 1723]; 
    	Out.ar(0, 
    		DynKlank.ar(`[freqs, nil, [1, 1, 1, 1]], PinkNoise.ar([0.007,0.007]))
    	)
    }).add
    
    a = Synth(\dynklanks)
    a.set(\freqs, [333, 444, 555, 666])
    a.set(\freqs, [333, 444, 555, 666].rand)

We know resonant filters when we hear them. The typical cry-baby wah wah guitar pedal is a band pass filter, for example. In the examples below we use a SinOsc to "move" the band pass frequency up and down the frequency spectrum. The SinOsc is here effectively working as a LFO (Low Frequency Oscillator - usually with a frequency below 20 Hz).

    { BPF.ar(Saw.ar(440), 440+(3000* SinOsc.kr(2, 0, 0.9, 1))) ! 2 }.play;
    { BPF.ar(WhiteNoise.ar(0.5), 1440+(300* SinOsc.kr(2, 0, 0.9, 1)), 0.2) ! 2}.play;


## Bell Synthesis using Subtractive Synthesis

The desired sound that you are trying to synthesize can be achieved through different methods. As an example, we could explore how to synthesize a bell sound with subtractive synthesis. 

    (
    {
    var chime, freqSpecs, burst, harmonics = 10;
    var burstEnv, burstLength = 0.001;
    freqSpecs = `[
    	{rrand(100, 1200)}.dup(harmonics), //freq array
    	{rrand(0.3, 1.0)}.dup(harmonics).normalizeSum, //amp array
    	{rrand(2.0, 4.0)}.dup(harmonics)]; //decay rate array
    burstEnv = Env.perc(0, burstLength); //envelope times
    burst = PinkNoise.ar(EnvGen.kr(burstEnv, gate: Impulse.kr(1))*0.4); //Noise burst
    Klank.ar(freqSpecs, burst)!2
    }.play
    )

This bell will be triggered every second. This is because the Impulse UGen is triggering the opening of the gate in the EnvGen (envelope generator) that uses the percussion envelope defined in the 'burstEnv' variable. If we wanted this to happen only once, we could set the frequency of the Impulse to zero. If we add a general envelope that frees the synth after being triggered, we could run a task that triggers bells every second.

    (
    Task({
    	inf.do({
    		{
    		var chime, freqSpecs, burst, harmonics = 30.rand;
    		var burstEnv, burstLength = 0.001;
    		freqSpecs = `[
    			{rrand(100, 8000)}.dup(harmonics), //freq array
    			{rrand(0.3, 1.0)}.dup(harmonics).normalizeSum, //amp array
    			{rrand(2.0, 4.0)}.dup(harmonics)]; //decay rate array
    		burstEnv = Env.perc(0, burstLength); //envelope times
    		burst = PinkNoise.ar(EnvGen.kr(burstEnv, gate: Impulse.kr(0))*0.5); //Noise burst
    		Klank.ar(freqSpecs, burst)!2 * EnvGen.ar(Env.linen(0, 4, 0), doneAction: 2) 
    		}.play;
    		[0.125, 0.25, 0.5, 1].choose.wait;
    	})
    }).play
    )

## Simulating the Moog

The much loved MiniMoog is a typical subtractive synthesis synthesizer. A few oscillator types can be mixed together and subsequently passed through a characteristic resonance low pass filter. We could try to simulate a setting on the MiniMoog, using the MoogFF UGen that simulates the Moog VCF (Voltage Controlled Filter) low pass filter and choosing, say, a saw wave form (The MiniMoog also has triangle, square, and two pulse waves).

We would typically start by sketching our synth by hooking up the UGens in a .play or .freqscope:

    {MoogFF.ar(Saw.ar(440), MouseX.kr(400, 16000), MouseY.kr(0.01, 4))}.freqscope

A common trick when simulating analogue equipment is to try to recreate the detuned oscillators of the analog synth (they are typically out of tune due to the difference of temperature within the synth itself). We can do this by adding another oscillator with a few Hz difference in frequency:

    // here we add two Saws and split the signal into two channels
    { MoogFF.ar(Saw.ar(440, 0.4) + Saw.ar(442, 0.4), 4000 ) ! 2 }.freqscope
    // like this:
    { ( SinOsc.ar(220, 0, 0.4) + SinOsc.ar(330, 0, 0.4) ) ! 2 }.play
    
    // here we "expand" the input of the filter into two channels (the array)
    { MoogFF.ar([Saw.ar(440, 0.4), Saw.ar(442, 0.4)], 4000 )  }.freqscope
    // like this - so different frequencies in each speaker:
    { [ SinOsc.ar(220, 0, 0.4), SinOsc.ar(330, 0, 0.4) ] }.play
    
    // here we "expand" the saw into two channels, but sum them and then split into two
    { MoogFF.ar(Saw.ar([440, 442], 0.4).sum, 4000 ) ! 2 }.freqscope
    // like this - and this is the one we'll use, although they're all fine:
    { SinOsc.ar( [220, 333], 0, 0.4) ! 2 }.play

We can then start to add arguments and prepare the synth graph for turning it into a SynthDef:

    { arg out=0, freq = 440, amp = 0.3, pan = 0, cutoff = 2, gain = 2, detune=2;
    	var signal, filter;
    	signal = Saw.ar([freq, freq+detune], amp).sum;
    	filter = MoogFF.ar(signal, freq * cutoff, gain );
    	Out.ar(out, Pan2.ar(filter, pan));
    }.play

The two synth graphs above are pretty much the same, except we have removed the mouse input in the latter one. You can see the frequency, amp, pan, and filter cutoff values are derived from the default arguments in the top line. There are only three things left for us to do in order to have a good working general synth: add an envelope, and wrap the graph up in a SynthDef with a name:

    SynthDef(\moog, { arg out=0, freq = 440, amp = 0.3, pan = 0, cutoff = 2, gain = 2, gate=1;
    	var signal, filter, env;
    	signal = Saw.ar(freq, amp);
    	env = EnvGen.ar(Env.adsr(0.01, 0.3, 0.6, 1), gate: gate, doneAction:2);
    	filter = MoogFF.ar(signal * env, freq * cutoff, gain );	
    	Out.ar(out, Pan2.ar(filter, pan));
    }).add;
    
    a = Synth(\moog);
    a.set(\freq, 222); // set the frequency of the synth
    a.set(\cutoff, 4); // set the cutoff (this would cut of at the 4th harmonic. Why?)
    a.set(\gate, 0); // kill the synth

We can now hook up a keyboard and play the \moog synth that we've designed. The MiniMoog is monophonic (only one note at a time), and it could be written like this:

    (
    c = 4;
    MIDIdef.noteOn(\myOndef, {arg vel, key, channel, device;
    	a.release; 
    	a = Synth(\moog, [\freq, key.midicps, \amp, vel/127, \cutoff, c]);
    	[key, vel].postln; 
    });
    MIDIdef.noteOff(\myOffdef, {arg vel, key, channel, device; 
    	a.release; 
    	//a = nil;
    	[key, vel].postln; 
    });
    )
    c = 10; // change the cutoff frequency at a later point 
    // the 'c' variable could be set from a GUI or a MIDI controller

The "a == nil", or "a.isNil" check is there to make sure that we don't press another note and overwriting the variable 'a' with another synth. What would happen then is that the noteOff method would free the last synth put into variable 'a' and not the prior ones. Try to remove the condition and see what happens. 

Finally, we might want to improve the MiniMoog and add a polyphonic feature. As we saw in an earlier chapter, we simply create an array for all the possible MIDI notes and turn them on and off:

    a = Array.fill(127, { nil });
    MIDIIn.connectAll;
    MIDIdef.noteOn(\myOndef, {arg vel, key, channel, device; 
    	// we use the key as index into the array as well
    	a[key] = Synth(\moog, [\freq, key.midicps, \amp, vel/127, \cutoff, 4]);
    });
    MIDIdef.noteOff(\myOffdef, {arg vel, key, channel, device; 
    	a[key].release;
    });

We will leave it up to you to decide how you want to control the cutoff and gain parameters of the MoogFF filter UGen. This could be done through knobs or sliders on a MIDI interface, on a GUI, or you could even decide to explore mapping key press velocity to the cutoff frequency, such that the note sounds brighter (or dimmer?) the harder you press the key.
