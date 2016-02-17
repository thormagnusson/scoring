# Musical Patterns in SC Lang

Throughout this tutorial we have been creating synthesizers, effects, routing them through busses, putting them into groups and more, but for many the question is how to make musical patterns or arrange events in time. For this we need some kind of a representation of the events, for example stored in an array, or generated algorithmically on the fly. Chapter 3 introduced some basic ways of controlling synths, but in this section we will explore in a bit more detail how to arrange musical events in time.


## The SynthDefs

For now we'll use two synth definitions.
    
    SynthDef(\sine, {arg out=0, amp=0.1, freq=440, envdur=1, pan=0.0;
    	var signal;
    	signal = Pan2.ar(SinOsc.ar(freq, 0, amp**amp).cubed, pan); // note the pan
    	signal = signal * EnvGen.ar(Env.perc(0.01, envdur), doneAction:2);
    	Out.ar(out, signal);
    }).add;
    
    SynthDef(\synth1, {arg out=0, freq=440, envdur=1, amp=0.4, pan=0;
        var x, env;
        env = EnvGen.kr(Env.perc(0.001, envdur, amp), doneAction:2);
        x = Mix.ar([FSinOsc.ar(freq, pi/2, 0.5), Pulse.ar(freq,Rand(0.3,0.7))]);
        x = RLPF.ar(x,freq*4,Rand(0.04,1));
        x = Pan2.ar(x,pan);
        Out.ar(out, x*env);
    }).add; 


## Routines and Tasks

We have already explored how to play a melody using a Task and a Routine (check the documentation for each, but in short a Task is a Routine that can be paused).

Function has a method called "fork" which will turn the function into a Routine (co-routine, and some could think of it as a "thread" - although technically it's not), but this allows for a process to run independently of what is happening elsewhere in the program. 

    Routine({
    	1.postln; 
    	Synth(\sine, [\freq, 220]);
    	0.5.wait;
    	2.postln;
    	Synth(\sine, [\freq, 220*2]);
    	0.5.wait;
    	3.postln;
    	Synth(\sine, [\freq, 220*3]);
    	0.5.wait;
    }).play

This could also be written as:

    { 3.do({arg i; (i+1).postln; Synth(\sine, [\freq, 220*(i+1)]); 0.5.wait }) }.fork

Or unpacked:

    { 
    	3.do({arg i; 
    		(i+1).postln; 
    		Synth(\sine, [\freq, 220*(i+1)]); 
    		0.5.wait;
    	}) 
    }.fork


So with a little melody stored in an array we could play it repeatedly:

    m = [60, 63, 64, 61];
    
    { inf.do({arg i; Synth(\sine, [\freq, m.wrapAt(i).midicps]); 0.5.wait }) }.fork

The "fork" is running a routine and the routine is played by SuperCollider's default TempoClock.

If you keep that code running and then evaluate:

    TempoClock.default.tempo = 2
 
You will see how the tempo changes, as the 0.5.wait in the Routine is half a beat of the tempo clock that has now changed its tempo.


## Clocks in SuperCollider

All temporal tasks in SuperCollider run from one of the language's clocks. There are 3 clocks in SuperCollider:

	- SystemClock (the main clock that starts when you launch SC)
	- TemploClock (same as SystemClock but counts in beats as opposed seconds)
	- AppClock (musically unreliable, but good for communicating with GUIs or external hardware)

Routines, Tasks and Patterns can all run by these 3 different clocks. You pass the clocks as arguments to them.

### SystemClock

Let's have a quick look at the SystemClock:

    (
    SystemClock.sched(2.0,{ arg time;  
    	time.postln; 
    	0.5 // wait between next scheduled event
    });
    )

    (
    SystemClock.sched(2.0,{ arg time;  
    	"HI THERE! Long wait".postln; 
    	nil // no wait - no next scheduled event
    });
    )

You can also schedule an event for an absolute time:

    (
    SystemClock.schedAbs( (thisThread.seconds + 4.0).round(1.0),{ arg time;
    	("the time is exactly " ++ time.asString 
    		++ " seconds since starting SuperCollider").postln;
    });
    )

### AppClock

The AppClock works pretty much the same but uses different source clocks (Apples NSTimers). 

You could try to create a GUI which is updated by a clock.

    w = Window.new("oo", Rect(100, 100, 240, 100)).front;
    x = Slider.new(w, Rect(20, 20, 200, 40));

    // This works
    {inf.do({x.value_(1.0.rand); 0.4.wait})}.fork(AppClock)
    
    // However this won't work (as it's using the TempoClock by default)
    {inf.do({x.value_(1.0.rand); 0.4.wait})}.fork

You will get an error message that could become familiar:

"Operation cannot be called from this Process. Try using AppClock instead of SystemClock."

You can also get this done by "deferring" the command to the AppClock using .defer.

    {inf.do({ {x.value_(1.0.rand)}.defer; Synth(\sine); 0.4.wait})}.fork

So here we are using the SystemClock to play the \sine synth, but deferring the updating of the GUI to the AppClock.

### TempoClock

TempoClocks are typically used for musical tasks. You can run many tempo clocks at the same time, at different tempi or in different meters. TempoClocks are ideal for high priority scheduling of musical events, and if there is a need for external communication, such as MIDI, GUI or Serial communication, the trick is to defer that message with a "{}.defer".

Let's explore the tempo clock:

    t = TempoClock(2); // tempo is 2 beats per second (120 bpm);

Many people who think in BPM (beats per minute) typically set the argument to the tempo clock as "120/60" (which equals to 2 beats per second), or "60/60" (which is 1 bps, and SuperCollider's "default" tempo).

The clock above is now in a variable "t" and we can use it to schedule events (at a particular beat in the future):

    t.schedAbs(t.beats.ceil, { arg beat, sec; [beat, sec].postln; 1});
    t.schedAbs(t.beats.ceil, { arg beat, sec; "ho ho --".post; [beat, sec].postln; 1 });

And we can change the tempo:

    t.tempo_(4)

    t.beatDur // we can ask the clock the duration of the beats
    t.beats // the beat time of the clock
    
    t.clear

Polyrhythm of 3/4 against 4/4

    (
    t = TempoClock(4);
    t.schedAbs(t.beats.ceil, { arg beat, sec;
    	beat.postln;
    	if (beat % 2==0, {Synth(\sine, [\freq, 444])});
    	if (beat % 4==0, {Synth(\sine, [\freq, 333])});
    	if (beat % 3==0, {Synth(\sine, [\freq, 888])});
    	1; // repeat
    });
    )
    t.tempo_(6)

Polyrhythm of 5/4 against 4/4

    (
    t = TempoClock(4);
    t.schedAbs(t.beats.ceil, { arg beat, sec;
    	if (beat % 2==0, {Synth(\sine, [\freq, 444])});
    	if (beat % 4==0, {Synth(\sine, [\freq, 333])});
    	if (beat % 5==0, {Synth(\sine, [\freq, 888])});
    	1; // repeat
    });
    
    )

Or perhaps a polyrhythm of 5/4 against 4/4 where the bass line is in 4/4 and the high synth in 5/4.

    (
    t = TempoClock(4);
    
    t.schedAbs(t.beats.ceil, { arg beat, sec;
    	if (beat % 2==0, {Synth(\sine, [\freq, 60.midicps])});
    	if (beat % 4==0, {Synth(\sine, [\freq, 64.midicps])});
    	if (beat % 5==0, {Synth(\synth1, [\freq, 72.midicps])});
    	if (beat % 5==3, {Synth(\synth1, [\freq, 77.midicps])});
    	1; // repeat
    });
    )

Another version

    (
    t = TempoClock(4);
    
    t.schedAbs(t.beats.ceil, { arg beat, sec;
    	if (beat % 4==0, {"one".postln; Synth(\sine, [\freq, 60.midicps])});
    	if (beat % 4==2, {"two".postln; Synth(\sine, [\freq, 72.midicps])});
    	if ((beat % 4==1) || (beat % 4==3), {Synth(\sine, [\freq, 84.midicps])});
    	
    	if (beat % 5==0, {Synth(\synth1, [\freq, 89.midicps, \amp, 0.2])});
    	if (beat % 5==2, {Synth(\synth1, [\freq, 96.midicps, \amp, 0.2])});
    	1; // repeat
    });
    )

We can try to make this a bit more interesting by creating another synth:

    (
    SynthDef( \klanks, { arg freqScale = 1.0, amp = 0.1;
    	var trig, klan;
    	var  p, exc, x, s;
    	trig = Impulse.ar( 0 );
    	klan = Klank.ar(`[ Array.fill( 16, { linrand(8000.0 ) + 60 }), nil, Array.fill( 16, { rrand( 0.1, 2.0)})], trig, freqScale );
    	klan = (klan * amp).softclip;
    	DetectSilence.ar( klan, doneAction: 2 );
    	Out.ar( 0, Pan2.ar( klan ));
    }).store;
    )

And play the same polyrhythm.

    (
    t = TempoClock(4);
    
    t.schedAbs(t.beats.ceil, { arg beat, sec;
    	if (beat % 4==0, {"one".postln; Synth(\klanks, [\freqScale, 40.midicps])});
    	if (beat % 4==2, {"two".postln; Synth(\klanks, [\freqScale, 52.midicps])});
    	if ((beat % 4==1) || (beat % 4==3), {Synth(\klanks, [\freqScale, 43.midicps])});
    	
    	if (beat % 7==0, {Synth(\synth1, [\freq, 88.midicps, \amp, 0.2])});
    	if (beat % 7==3, {Synth(\synth1, [\freq, 96.midicps, \amp, 0.2])});
    	if (beat % 7==5, {Synth(\synth1, [\freq, 86.midicps, \amp, 0.2])});
    
    	1; // repeat
    });
    )
    
    t.tempo_(8)

an example showing tempo changes 

    (
    t = TempoClock(80/60); // 80 bpm
    // schedule an event at next whole beat
    t.schedAbs(t.beats.ceil, { arg beat, sec; 
    	"beat : ".post; beat.postln;
    	if (beat % 4==0, { Synth(\sine, [\freq, 60.midicps]) });
    	if (beat % 4==2, { Synth(\sine, [\freq, 67.midicps]) });
    	if (beat % 0==0, { Synth(\sine, [\freq, 72.midicps]) });
    	1 // 1 here means that we are repeating/looping this
    });
    t.schedAbs(16, { arg beat, sec; 
    	" ****  tempochange on beat : ".post; beat.postln; 
    	t.tempo_(150/60); // 150 bpm
    });
    5.do({ |i| // on beats 32, 36, 40, 44, 48 
    	t.schedAbs(32+(i*4), { arg beat, sec;
    		" ****  tempo is now : ".post; (150-(10*(i+1))).post; " BPM".postln; 
    		t.tempo_((150-(10*(i+1)))/60); // going down by 10 bpm each time
    	});
    });
    t.schedAbs(60, { arg beat; t.tempo_(200/60) }); // 200 bpm
    t.schedAbs(76, { arg beat;
    	t.clear;
    	t.schedAbs(t.beats.ceil, { arg beat, sec; 
    		"beat : ".post; beat.postln;
    		if (beat % 4==0, { Synth(\sine, [\freq, 67.midicps]) });
    		if (beat % 4==2, { Synth(\sine, [\freq, 74.midicps]) });
    		if (beat % 0==0, { Synth(\sine, [\freq, 79.midicps]) });
    		1 // 1 here means that we are repeating/looping this
    	});
    	t.schedAbs(92, { arg beat; t.stop }); // stop it!
    }); // 200 bpm
    t.schedAbs(92, { arg beat; t.stop }); // if we tried to stop it here, it would have been "cleared"
    )

## A survey of Patterns

We can try to play the above synth definitions with Patterns and it will play using the default arguments of patterns (see the Event source file). Let's start by exploring the Pbind pattern. As we saw in chapter 3, if you run the code below:

    ().play // "()" is an empty Event dictionary
    Pbind().play // Pbind plays an empty Event

You can hear that there are default arguments, like a note played every second, an instrument is used (SuperCollider's \default) and a frequency (440Hz).

In the example below, we use Pbind (Pattern that binds keys (synth def arguments) and their arguments). Here we pass the \sine synth def as the argument for the \instrument (again as defined in the Event class).

    Pbind(\instrument, \sine).play // it plays our synth definition

    Pbind(\instrument, \sine, \freq, Pseq([60, 65, 57, 62].midicps)).play // it plays our synth definition

Our \sine synth has a frequency argument, and we are sending the frequency directly. However, if we wanted we could also send 'note' or 'midinote' arguments, but here the values are converted internally to the \freq argument of \sine.

    Pbind(\instrument, \sine, \note, Pseq([0, 5, 7, 2])).play // it plays our synth definition

    Pbind(\instrument, \sine, \midinote, Pseq([60, 65, 57, 62])).play // it plays our synth definition


Pattern definitions (Pdef) are a handy way to define and play patterns. They are a bit like Synth definitions in that they have a unique name and can be recompiled on the fly.


    (
    Pdef(\scale, Pbind(	\instrument, \sine,
    					\freq, Pseq([62, 64, 67, 69, 71, 74], inf).midicps,
    					\dur,  Pseq([0.25, 0.5, 0.25, 0.25, 0.5, 0.5], inf)
    )); 
    )
    
    a = Pdef(\scale).play;
    a.pause 	// pause. the stream
    a.resume 	// resume it
    a.stop 	// stop it (resets it)
    a.play 	// start again

Then we can set variables in our instrument using .set

    Pdef(\scale).set(\out, 20); // outbus 20 
    Pdef(\scale).set(\out, 0); // outbus 0 

    // here we set the duration of the envelope in our instrument
    Pdef(\scale).set(\envdur, 0.2);

Patterns use default keywords defined in the Event class, so take care not to use those keywords in your synth definitions. If we had used dur instead of envdur for the envelope in our instrument, this would happen:

    Pdef(\scale).set(\dur, 0.1);

because dur is a keyword of Patterns (the main ones are \dur, \freq, \amp, \out, \midi)

resetting the freq info is not possible however :

    Pdef(\scale).set(\freq, Pseq([72,74,72,69,71,74], inf).midicps);

one solution would be to resubmit the Pattern Definition:

    (
    Pdef(\scale, Pbind( \instrument, \sine,
    				\freq, Pseq([72,74,72,69,71,74], inf).midicps, // different sequence
    				\dur,  Pseq([0.25, 0.5, 0.25, 0.25, 0.5, 0.5], inf)
    )); 
    )

and it's still in our variable "a", it's just the definition that's different

    a.pause
    a.resume

### Patterns and environmental variables

Or we could use Pdefn (read the helpfiles to compare Pdef and Pdefn) (here we are using envrionment variables to refer to patterns)

We use a Pdefn to hold the scale

    Pdefn(\scaleholder, { |arr| Pseq(arr.freqarr) });
    // and we add an array to it
    Pdefn(\scaleholder).set(\freqarr, Array.fill(6, {440 +(300.rand)} ));

then we play a Pdef with the Pdefn

    Pdef(\scale, 
    		Pbind( 	\instrument, \synth1,
    				\freq, Pn(Pdefn(\scaleholder), inf), // loop
    				\dur, 0.4
    			)
    			
    ); 
    a = Pdef(\scale).play;
    
    // and we can reset our scale 
    Pdefn(\scaleholder).set(\freqarr, Array.fill(3, {440 +(300.rand)} ));


Another example

    (
    Pdefn(\deg, Pseq([0, 3, 2],inf));
    
    Pset(\instrument, \synth1, 
    	Ppar([
    		Pbind(\degree, Pdefn(\deg)),
    		Pbind(\degree, Pdefn(\deg), \dur, 1/3)
    ])
    ).play;
    )
    
    Pdefn(\deg, Prand([0, 3, [1s, 4]],inf));
    Pdefn(\deg, Pn(Pshuf([4, 3, 2, 7],2),inf));
    Pdefn(\deg, Pn(Pshuf([0, 3],2),inf));
    
    (
    Pdefn(\deg, Plazy { var pat;
    				pat = [Pshuf([0, 3, 2, 7, 6],2), Pshuf([3, 2, 6],2), Pseries(11, -1, 11)].choose;
    				Pn(pat, inf)
    		});
    )


/////////////// p


    (
    Pdef(\player).set(\instrument, \synth1);
    
    Pdef(\player,
    	Pbind(
    		\instrument, 	Pfunc({ |e| e.instrument }),
    		\midinote, 	Pseq([45,59,59,43,61,43,61,61,45,33,31], inf),
    		\dur, 		Pseq ([0.25,1,0.25,0.5,0.5,0.5,0.125,0.125,0.5,0.25,0.25], inf),
    		\amp, 		Pseq ([1,0.1,0.2,1,0.1125,0.1125,1,0.1125,0.125,0.25,1,0.5], inf)
    	)
    );
    )
    
    Pdef(\player).play;
    
    Pdef(\player).set(\instrument, \synth1);
    Pdef(\player).set(\envdur, 0.1);
    Pdef(\player).set(\envdur, 0.25);
    Pdef(\player).set(\envdur, 1);
    Pdef(\player).set(\instrument, \sine);


///////////////////////////////////////////////////////


    (
    ~scale = [62,67,69, 77];
    
    c = Pdef(\p04b, 
    		Pbind(\instrument, \synth1, 
    					\freq, (Pseq.new(~scale, inf)).midicps, // freq arg
    					\dur, Pseq.new([1, 1, 1, 1], inf);  // dur arg
    		)
    );
    
    c = Pdef(\p04c, 
    		Pbind(\instrument, \synth1,
    					\freq, (Pseq.new(~scale, inf)).midicps, // freq arg
    					\dur, Pseq.new([1, 1, 1, 1], inf);  // dur arg
    		)
    );
    )
    
    Pdef(\p04b).quant_([2, 0, 0]);
    Pdef(\p04c).quant_([2, 0.5, 0]); // offset by half a beat
    Pdef(\p04b).play;
    Pdef(\p04c).play;

(quant can't be reset in real-time, so we use align to align patterns). align takes the same arguments as quant (see helpfile of Pdef)

    Pdef(\p04c).align([4, 0, 0]);
    Pdef(\p04c).align([4, 0.75, 0]); // offset by 3/4 a beat


Another useful pattern is Tdef (Task patterns)

    Tdef(\x, { loop({ Synth(\sine, [\freq, 200+(440.rand)]); 0.25.wait; }) });

    TempoClock.default.tempo = 2; // it runs on the default tempo clock

    Tdef(\x).play(quant:1);
    Tdef(\x).stop;

and we can redefine the definition "x" in realtime whilst playing

    Tdef(\x, { loop({ Synth(\synth1, [\freq, 200+(440.rand)]); 1.wait; }) });
    
    Tdef(\y, { loop({ Synth(\synth1, [\freq, 1200+(440.rand)]); 1.wait; }) });
    Tdef(\y).play(quant:1);
    
    Tdef(\y).stop;


to change the values in a pattern in realtime, use List instead of Array:

    ~notes = List[63, 61, 64, 65];
    
    Pbind(
    	\midinote, Pseq(~notes, inf),
    	\dur, Pseq([0.4, 0.2, 0.1, 0.2], inf)
    ).play;
    
    ~notes[1] = 80

yet another (known?) melody

    (
    Pbind(
     	\midinote, Pseq([72, 76, 79, 71, 72, 74, 72, 81, 79, 84, 79, 77, 76, 77, 76], 1),
     	\dur, Pseq([4, 2, 2, 3, 0.5, 0.5, 4, 4, 2, 2, 2, 1, 0.5, 0.5, 2]/4, 1)
     ).play
    )

### USING Pfx (effects patterns)

Make the synthdef and add it

    SynthDef(\testenv2, { arg in=0, dur=2;
    	var env;
    	env = EnvGen.kr(Env.sine(dur), doneAction:2).poll;
    	XOut.ar(0, 1, (In.ar(in, 1)+WhiteNoise.ar(0.1)) * env); // add noise for clarity
    }).add;
    
    
    p = Pbind(\degree, Pseq([0, 4, 4, 2, 8, 3, 2, 0]), \dur, 0.5);
    p.play
    q = Pfx(p, \testenv, \dur, 4); // play it... all working (sine env is 4 secs)
    q.play

Now write the def to disk

    SynthDef(\testenv2, { arg in=0, dur=2;
    	var env;
    	env = EnvGen.kr(Env.sine(dur), doneAction:2).poll;
    	XOut.ar(0, 1, (In.ar(in, 1)+WhiteNoise.ar(0.1)) * env); // add noise for clarity
    }).writeDefFile;
    
    
    // quit SuperCollider, open it again and now try this
    p = Pbind(\degree, Pseq([0, 4, 4, 2, 8, 3, 2, 0]), \dur, 0.5);
    q = Pfx(p, \testenv, \dur, 4); // not working (sine env is 2 secs, the synthdef default)
    q.play

but here is the trick, read the SynthDescLib and try again!

    SynthDescLib.global.read;
    q = Pfx(p, \testenv, \dur, 4); // not working (sine env is 2 secs, the synthdef default)
    q.play

rendering the pattern as soundfile to disk (it will be written to your SuperCollider folder)

    q.render(
    	"ixi_tutorial_render_test.aif", 
    	4, 
    	sampleFormat: "int16", 
    	options: Server.default.options.numOutputBusChannels_(2)
    );


## TempoClock and Patterns

To chage the tempo of the above Patterns, you can use the default TempoClock (as you didn't register a TempoClock for the pattern)

    TempoClock.default.tempo = 1.2

But if you want to have each pattern playing different TempoClocks, you need to create 2 clocks and use them to drive each pattern. (this way one can do some nice phasing/polyrhytmic stuff)

    (
    t = TempoClock.new;
    u = TempoClock.new;
    Pdef(\p04b).play(t);
    Pdef(\p04c).play(u);
    u.tempo = 1.5
    )

it's hard to get this clear as they are running the same pitch patterns so let's redefine one of the patterns:

    (
    Pdef(\p04c, 
    		Pbind(\instrument, \synth1,
    					\freq, (Pseq.new(~scale.scramble, inf)).midicps*2, // freq arg
    					\dur, Pseq.new([1, 1, 1, 1], inf);  // dur arg
    		)
    )
    )
    // and try to change the tempo 
    u.tempo = 1;
    u.tempo = 1.2;
    u.tempo = 1.8;
    u.tempo = 3.2;



### Popcorn


// An example of making a tune using patterns. For an excellent example
// take a look at spacelab, in examples/pieces/spacelab.scd


SynthDescLib.global.read;
// the poppcorn 

(
~s1 = [72, 70, 72, 67, 64, 67, 60];
~s2 = [72, 74, 75, 74, 75, 74, 72, 74, 72, 74, 72, 70, 72, 67, 64, 67, 72];

~t1 = [0.25, 0.25, 0.25, 0.25, 0.125, 0.25, 0.625];
~t2 = [0.25, 0.25, 0.25, 0.125, 0.25, 0.125, 0.25, 0.25, 0.125, 0.25, 0.125, 0.25, 0.25, 0.25, 0.125, 0.25, 0.5 ];

c = Pdef(\moogy, 
		Pbind(\instrument, \synth1, // using our synth1 synthdef
					\freq, 
						Pseq.new([
							Pseq.new([
								Pseq.new(~s1.midicps, 2),
								Pseq.new(~s2.midicps, 1)
								], 2),
							Pseq.new([
								Pseq.new((~s1+7).midicps, 2),
								Pseq.new((~s2+7).midicps, 1)
								], 2)	
							], inf),
					\dur, Pseq.new([ 
							Pseq.new(~t1, 2),
							Pseq.new(~t2, 1)
							], inf)
		)
);
Pdef(\moogy).play
)


### Mozart

A little transcription of Mozart’s Piano Sonata No 16 in C major. Here the instrument has been put into a variable called "instr" so it's easier to quickly change the instrument.

(
var instr = \default;
Ppar([
// right hand - using the Event-style notation
Pseq([
        (\instrument: instr, \midinote: 72, \dur: 1),
        (\instrument: instr, \midinote: 76, \dur: 0.5),
        (\instrument: instr, \midinote: 79, \dur: 0.5),
        (\instrument: instr, \midinote: 71, \dur: 0.75),
        (\instrument: instr, \midinote: 72, \dur: 0.125),
        (\instrument: instr, \midinote: 74, \dur: 0.125),
        (\instrument: instr, \midinote: 72, \dur: 1),
        (\instrument: instr, \midinote: 81, \dur: 1),
        (\instrument: instr, \midinote: 79, \dur: 0.5),
        (\instrument: instr, \midinote: 84, \dur: 0.5),
        (\instrument: instr, \midinote: 79, \dur: 0.5),
        (\instrument: instr, \midinote: 77, \dur: 0.25),
        (\instrument: instr, \midinote: 76, \dur: 0.125),
        (\instrument: instr, \midinote: 77, \dur: 0.125),
        (\instrument: instr, \midinote: 76, \dur: 1)
], 1),

// left hand - array notation
Pbind(\instrument, instr, 
        \midinote, Pseq([60, 67, 64, 67, 60, 67, 64, 67, 62, 67, 65, 67, 60, 67, 64, 67,
	                 60, 69, 65, 69, 60, 67, 64, 67, 59, 67, 62, 67, 60, 67, 64, 67 ], 1),
        \dur, 0.25
        )], 1).play
)


## Syncing Patterns and TempoClocks


SynthDef(\string, {arg out=0, freq=440, pan=0, sustain=0.5, amp=0.3;
	var pluck, period, string;
	pluck = PinkNoise.ar(Decay.kr(Impulse.kr(0.005), 0.05));
	period = freq.reciprocal;
	string = CombL.ar(pluck, period, period, sustain*6);
	string = LeakDC.ar(LPF.ar(Pan2.ar(string, pan), 12000)) * amp;
	DetectSilence.ar(string, doneAction:2);
	Out.ar(out, string)
}).add;


SynthDef(\impulse, {
	Out.ar(0, Impulse.ar(0)!2);	
}).add

Synth(\impulse)

Pbind(
	\instrument, \impulse,
	\dur, 1
).play(TempoClock.default, quant:1)

// not working
TempoClock.default.play({
	Synth(\impulse, [\amp, 2]); // this is the problem
	1.0
	}, quant:[1, Server.default.latency] );

// working
TempoClock.default.play({
	s.sendBundle(0.2, ["/s_new", \impulse, s.nextNodeID, 0, 1]);
	1.0
	}, quant:[1, 0] );

TempoClock.default.tempo = 2.5

Pbind(
	\instrument, \string,
	\freq, Pseq([440, 880], inf),
	\dur, 1
).play(TempoClock.default, quant:1);

TempoClock.default.play({arg i;
	s.sendBundle(0.2, ["/s_new", \string, s.nextNodeID, 0, 1, \freq, if(i.asInteger.even, {660}, {770}), \amp, 0.3]);
	1.0
	}, quant:[1, 0] );
 
