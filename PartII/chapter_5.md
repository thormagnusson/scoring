# Additive Synthesis

In the early 19th century, the mathematician Joseph Fourier came up with a theory that stated that any sound can be described as a function of pure sine waves. This is a very important statement for computer music. It means that we can recreate any sound that we hear by adding number of sine waves together with different frequency, phase and amplitude. Obviously this was a costly technique in times of modular synthesis, as one would have to apply multiple oscillators to get the desired sound. This has changed with digital sound, where innumerable oscillators can be added together with little cost. Here is a proof:


    // we add 500 oscillators together and the CPU is less than 20% 
    {({SinOsc.ar(4444.4.rand, 0, 0.005)}!500).sum}.play

## Adding waves

Adding waves together seems simple, and indeed it is. By using the plus operator we can add two signals together and their values at the same time add up to the combined value. In the following images we can see how simple sinusoidal waves add up:

![Adding two waves of 440Hz together](images/ch5_two440Hz.png)

    {[SinOsc.ar(440), SinOsc.ar(440), SinOsc.ar(440)+SinOsc.ar(440)]}.plot
    // try this as well
    {a = SinOsc.ar(440, 0, 0.5); [a, a, a+a]}.plot

![Adding a 440Hz and a 220Hz wave together](images/ch5_200n400Hz.png)

    {[SinOsc.ar(440), SinOsc.ar(220), SinOsc.ar(440)+SinOsc.ar(220)]}.plot

![Adding two 440 waves together but one with inverted phase](images/ch5_200n400Hz.png)

    {[SinOsc.ar(440), SinOsc.ar(440, pi), SinOsc.ar(440)+SinOsc.ar(440, pi)]}.plot

You see that two waves at the same frequency added together becomes twice the amplitude. When two waves with the amplitude of 1 are added together we get an amplitude of 2 and in the graph we get a clipping where the wave is clipped at 1. This can cause a distortion, but also resulting in a different wave form, namely a square wave. You can explore this by giving a sine wave the amplitude of 10, but then clip the signal at, say -0.75 and 0.75.

{SinOsc.ar(440, 0, 10).clip(-0.75, 0.75)}.scope

Most instrumental sounds can be roughly described as a combination of sine waves. Those sinusoidal waves are called partials (the horizontal lines you see in a spectrogram when you analyse a sound). In the example below we mix ten sine waves of frequencies between 200 and 2000. You might well be able to detect a pitch in the example if you run it many times, but since these are random frequencies they are not necessarily lining up to give us a solid pitch. 

    {Mix.fill(10, {SinOsc.ar(rrand(200,2000), 0, 0.1)})}.freqscope
    {Mix.fill(10, {SinOsc.ar(rrand(200,2000), 0, 0.1)})}.spectrogram 
    
In harmonic sounds, like the piano, guitar, or the violin we get partials that are whole number multiples of the fundamental (the lowest) partial. If they are fundamental multiples, the partials are called **harmonics**. The harmonics can be of varied amplitude, phase, envelope form, and duration. A saw wave is a waveform with all the harmonics represented, but lowering in amplitude:

    {Saw.ar(880)}.freqscope

It is recommended that you play with adding waves together in various ways. Explore what happens when you add harmonics together (integer multiples of a fundamental frequency), 

    // adding two waves - the second is the octave (second harmonic) of the first
    {(SinOsc.ar(440,0, 0.4) + SinOsc.ar(880, 0, 0.4))!2}.play

    // here we add four harmonics (of equal amplitude) together
    (
    {	
    var freq = 200;
    SinOsc.ar(freq, 0, 0.2)   + 
    SinOsc.ar(freq*2, 0, 0.2) +
    SinOsc.ar(freq*3, 0, 0.2) + 
    SinOsc.ar(freq*4, 0, 0.2) 
    !2}.play
    )

The harmonic series is something we all know intuitively and have heard many times (swing a flexible tube around your head and you will get a sound in the harmonic series). The Blip UGen in SuperCollider allows you to dynamically control the number of harmonics of equal amplitude:

    {Blip.ar(440, MouseX.kr(1, 20))}.scope // using the Mouse
    {Blip.ar(440, MouseX.kr(1, 20))}.freqscope
    {Blip.ar(440, Line.kr(1, 22, 3) )}.play

## Creating wave forms out of sinusoids

In SuperCollider you can create all kinds of wave forms out of a combination of sine waves. By adding SinOscs together, you can derive at your own unique wave forms that you might use in your synths. In this section we will look at how we use additive synthesis to derive at diverse wave forms.

T>## Creating Arrays
T>
T> There are many ways of creating arrays, and in SuperCollider the syntax for creating arrays is quite flexible. They are all doing pretty much the same thing under the hood (check the source code of Object:dup and Object:!, by hitting Cmd+I or Ctrl+I). 

    // a) here is an array with 5 items:
    Array.fill(5, {arg i; i.postln;});
    // b) this is the same as (using a shortcut):
    {arg i; i.postln;}.dup(5)
    // c) or simply (using another shortcut):
    {arg i; i.postln;}!5
    
    // d) we can then sum the items in the array (add them together):
    Array.fill(5, {arg i; i.postln;}).sum;
    // e) we could do it this way as well:
    sum({arg i; i.postln;}.dup(5));
    // f) or this way:
    ({arg i; i.postln;}.dup(5)).sum;
    // g) or this way:
    ({arg i; i.postln;}!5).sum;
    // h) or simply this way:
    sum({arg i; i.postln;}!5);

Above we created a Saw wave which contains harmonics up to the [Nyquist rate] (http://en.wikipedia.org/wiki/Nyquist_rate), which is half of the sample rate SuperCollider is running. The Saw UGen is "band-limited" which means that it does not alias and mirror back into the audible range. (Compare with LFSaw which will alias - you can both hear and see the harmonics mirror back into the audio range).

    {Saw.ar(MouseX.kr(100, 1000))}.freqscope
    {LFSaw.ar(MouseX.kr(100, 1000))}.freqscope

We can now try to create a saw wave out of sine waves. There is a simple algorithm for this, where each partial is an integer multiple of the fundamental frequency, and decreasing in amplitude by the reciprocal of the partials's/harmonic's number (1/harmnum).

A 'Saw' wave with 30 harmonics:

    (
    f = {
            ({arg i;
                    var j = i + 1;
                    SinOsc.ar(300 * j, 0,  j.reciprocal * 0.5);
            } ! 30).sum // we sum this function 30 times
    !2}; // and we make it a stereo signal
    )

    f.plot; // let's plot the wave form
    f.play; // listen to it
    f.freqscope; // view and listen to it
    
By inverting the phase (using pi), we get an inverted wave form.


    (
    f = {
            Array.fill(30, {arg i;
                    var j = i + 1;
                    SinOsc.ar(300 * j, pi,  j.reciprocal * 0.5) // note pi
            }).sum // we sum this function 30 times
    !2}; // and we make it a stereo signal
    )
    
    f.plot; // let's plot the wave form
    f.play; // listen to it
    f.freqscope; // view and listen to it

A square wave is a type of a pulse wave (If the length of the on time of the pulse is equal to the length of the off time – also known as a duty cycle of 1:1 – then the pulse wave may also be called a square wave). The square wave can be created by sine waves if we ignore all the even harmonics and only add the odd ones.

    (
    f = {
            ({arg i;
                    var j = i * 2 + 1; // the odd harmonics (1,3,5,7,etc)
                    SinOsc.ar(300 * j, 0, 1/j)
            } ! 20).sum;
    };
    )
    
    f.plot;
    f.play;
    f.freqscope;

Let's quickly look at the regular Pulse wave in SuperCollider:
 
    { Pulse.ar(440, MouseX.kr(0, 1), 0.5) }.scope;
    // we could also recreate this with an algorithm on a sine wave:
    { if( SinOsc.ar(122)>0 , 1, -1 )  }.scope; // a square wave
    { if( SinOsc.ar(122)>MouseX.kr(0, 1) , 1, -1 )  }.scope; // MouseX controls the period
    { if( SinOsc.ar(122)>MouseX.kr(0, 1) , 1, -1 ) * 0.1 }.scope; // amplitude down

A triangle wave is a wave form, similar to the pulse wave in that it ignores the even harmonics, but it has a different algorithm for the phase and the amplitude:

    (
    f = {
            ({arg i;
                    var j = i * 2 + 1;
                    SinOsc.ar(300 * j, pi/2, 0.7/j.squared) // cosine wave (phase shift)
            } ! 20).sum;
    };
    )
    f.plot;
    f.play;
    f.freqscope;

We have now created various wave forms using sine waves, and here is how to wrap them up in a SynthDef for future use:

    SynthDef(\triwave, {arg freq=400, pan=0, amp=1;
    	var wave;
    	wave = ({arg i;
                    	var j = i * 2 + 1;
                    	SinOsc.ar(freq * j, pi/2, 0.6 / j.squared);
            	} ! 20).sum;
    	Out.ar(0, Pan2.ar(wave * amp, pan));
    }).add;
    
    a = Synth(\triwave, [\freq, 300]);
    a.set(\amp, 0.3, \pan, -1);
    b = Synth(\triwave, [\freq, 900]);
    b.set(\amp, 0.4, \pan, 1);
    s.freqscope; // if the freqscope is not already running
    b.set(\freq, 1400); // not band limited as we can see 
~~~~~~~

We have created various typical wave forms above in order to show how they are sums of sinusoidal waves. A good idea is to play with this further and create your own waveforms:

{line-numbers=off}
~~~~~~~
(
f = {
        ({arg i;
                var j = i * 2.cubed + 1;
                SinOsc.ar(MouseX.kr(20,800) * j, 0, 1/j)
        } ! 20).sum;
};
)
f.plot;
f.play;
~~~~~~~

{line-numbers=off}
~~~~~~~
(
f = {
        ({arg i;
                var j = i * 2.squared.distort + 1;
                SinOsc.ar(MouseX.kr(20,800) * j, 0, 0.31/j)
        } ! 20).sum;
};
)
f.plot;
f.play;
~~~~~~~


## Bell Synthesis

Not all sounds are harmonic. Many musical instruments are **inharmonic**, for example timpani drums, xylophones, and bells. Here the partials of the sound are not in a harmonic relationship (or multiples of some fundamental frequency). This does not mean that we can't detect pitch, as there will be certain partials that have stronger amplitude and longer duration than others. Since we know bells are inharmonic, the first thing we might try is to generate a sound with, say, 15 partials:

    { ({ SinOsc.ar(rrand(80, 800), 0, 0.1)} ! 15).sum }.play


Try to run this a few times. What we hear is a wave form that might be quite similar to a bell at first, but then the resemblance disappears, because the partials do not fade out. If we add an envelope to each of these sinusoids, we get a different sound:

    {
    Mix.fill( 10, { 	
    	SinOsc.ar(rrand(200, 700), 0, 0.1) 
    	* EnvGen.ar(Env.perc(0.0001, rrand(2, 6))) 
    });
    }.play
~~~~~~~

Above we are using Mix.fill instead of creating an array with ! and then .summing it. These two examples do the same thing, but it is good for the student of SuperCollider to learn different ways of reading and writing code.

You note that there is a "new" bell every time we run the above code. But what if we wanted the "same" bell? One way to do that is to "hard-code" the frequencies, durations, and the amplitudes of the bell.

{line-numbers=off}
~~~~~~~
{
var freq = [333, 412, 477, 567, 676, 890, 900, 994];
var dur = [4, 3.5, 3.6, 3.1, 2, 1.4, 2.4, 4.1];
var amp = [0.4, 0.2, 0.1, 0.4, 0.33, 0.22, 0.13, 0.4];
Mix.fill( 8, { arg i;
	SinOsc.ar(freq[i], 0, 0.1) 
	* EnvGen.ar(Env.perc(0.0001, dur[i])) 
});
}.play
~~~~~~~

Generating a SynthDef using a non-deterministic algorithms (such as random) in the SC-lang will also generate a SynthDef that is the "same" bell. Why? This is because the values (430.rand) are defined when the synth definition is compiled. Try to recompile the SynthDef and you get a new sound:

{line-numbers=off}
~~~~~~~
(
SynthDef(\mybell, {arg freq=333, amp=0.4, dur=2, pan=0.0;
	var signal;
	signal = Mix.fill(10, {
		SinOsc.ar(freq+(430.rand), 1.0.rand, 10.reciprocal) 
		* EnvGen.ar(Env.perc(0.0001, dur), doneAction:2) }) ;
	signal = Pan2.ar(signal * amp, pan);
	Out.ar(0, signal);
}).add
)
// let's try our bell
Synth(\mybell) // same sound all the time
Synth(\mybell, [\freq, 444+(400.rand)]) // new frequency, but same sound
// try to redefine the SynthDef above and you will now get a different bell:
Synth(\mybell) // same sound all the time
~~~~~~~

Another way of generating this bell sound would be to use the SynthDef from last tutorial, but here adding a duration to the envelope:

{line-numbers=off}
~~~~~~~
(
SynthDef(\sine, {arg freq=333, amp=0.4, dur, pan=0.0;
	var signal, env;
	env = EnvGen.ar(Env.perc(0.01, dur), doneAction:2);
	signal = SinOsc.ar(freq, 0, amp) * env;
	signal = Pan2.ar(signal, pan);
	Out.ar(0, signal);
}).add
);

(
var numberOfSynths;
numberOfSynths = 15;
Array.fill(numberOfSynths, {
	Synth(\stereosineWenv, [	
		\freq, 300+(430.rand),
		\phase, 1.0.rand,
		\amp, numberOfSynths.reciprocal, // reciprocal here means 1/numberOfSynths
		\dur, 2+(1.0.rand)]);
});
)
~~~~~~~

The power of using this style would be if you really wanted to be able to define all the parameters of the sound from the language, for example sonifying some complex information from gestural or other data.

### The Klang Ugen

Another interesting way of achieving this is to use the Klang UGen. Klang is a bank of sine oscillators that takes arrays of frequencies, amplitudes and phase as arguments.

{line-numbers=off}
~~~~~~~
{Klang.ar(`[ [430, 810, 1050, 1220], [0.23, 0.13, 0.23, 0.13], [pi,pi,pi, pi]], 1, 0)}.play
~~~~~~~

And we create a SynthDef with the Klang Ugen:

{line-numbers=off}
~~~~~~~
(
SynthDef(\saklangbell, {arg freq=400, amp=0.4, dur=2, pan=0.0; // we add a new argument
	var signal, env;
	env = EnvGen.ar(Env.perc(0.01, dur), doneAction:2); // doneAction gets rid of the synth
	signal = Klang.ar(`[freq * [1.2,2.1,3.0,4.3], [0.25, 0.25, 0.25, 0.25], nil]) * env;
	signal = Pan2.ar(signal, pan);
	Out.ar(0, signal);
}).add
)
Synth(\saklangbell, [\freq, 100])
~~~~~~~


## Xylophone Synthesis

Additive synthesis is good for various types of sound, but it suites very well for xylophones, bells and other metallic instruments (typically inharmonic sounds) as we saw with the bell example above. Using harmonic wave forms, such as a Saw wave, Square wave or Triangle wave would not be useful here as those are harmonic wave forms (as we know from the section above). 

In additive synthesis, people often analyse the sound they're trying to synthesise with generating a spectrogram of its frequencies.

![A spectrogram of a xylophone sound](images/ch5_xylophone.png)

The information the spectrogram gives us is three dimensional. It shows us the frequencies present in the sound on the vertical x-axis, the time on the horizontal y-axis, and amplitude is color (which we could imagine as the z-axis). We see that the partials don't have the same type of envelopes: some have strong attack, others come smoothly in; some have much amplitude, others less; some have a long duration whilst other have less; and of them vibrate in frequency. These parameters can mix. A loud partial could die out quickly while a soft one can live for a long time.

{line-numbers=off}
~~~~~~~
{ ({ SinOsc.ar(rrand(180, 1200), 0.5*pi, 0.1) // the partial
		*
	// each partial gets its own envelope of 0.5 to 5 seconds
	EnvGen.ar(Env.perc(rrand(0.00001, 0.01), rrand(0.5, 5)))
} ! 12).sum }.play
~~~~~~~

Analysing the bell above we can detect the following partials
* partial 1: xxx Hz, x sec. long, with amplitude of ca. x
* partial 2: xxx Hz, x sec. long, with amplitude of ca. x
* partial 3: xxx Hz, x sec. long, with amplitude of ca. x
* partial 4: xxx Hz, x sec. long, with amplitude of ca. x
* partial 5: xxx Hz, x sec. long, with amplitude of ca. x
* partial 6: xxx Hz, x sec. long, with amplitude of ca. x
* partial 7: xxx Hz, x sec. long, with amplitude of ca. x

We can now try to synthesize those harmonics:

{line-numbers=off}
~~~~~~~
{ SinOsc.ar(xxx, 0, 0.1)+
SinOsc.ar(xxx, 0, 0.1)+
SinOsc.ar(xxx, 0, 0.1)+
SinOsc.ar(xxx, 0, 0.1)+
SinOsc.ar(xxx, 0, 0.1)+
SinOsc.ar(xxx, 0, 0.1)
}.play
~~~~~~~

And we get a decent inharmonic sound (inharmonic is where the partials are not whole number multiples of a fundamental frequency). We would now need to set the right amplitude as well and we're still guessing from the spectrogram we made, but more importantly we should be using our ears.

{line-numbers=off}
~~~~~~~
{ SinOsc.ar(xxx, 0, xxx)+
SinOsc.ar(xxx, 0, xxx)+
SinOsc.ar(xxx, 0, xxx)+
SinOsc.ar(xxx, 0, 0.1)+
SinOsc.ar(xxx, 0, 0.1)+
SinOsc.ar(xxx, 0, 0.1)
}.play
~~~~~~~

Some of the partials have a bit of vibration and we could simply turn the oscillator into a 'detuned' oscillator by adding two sines together:

{line-numbers=off}
~~~~~~~
// a regular 880 Hz wave at full amplitude
{SinOsc.ar(880)!2}.play
// a vibrating 880Hz wave (vibration at 3 Hz), where each is amp 0.5
{SinOsc.ar([880, 883], 0, 0.5).sum!2}.play
// the above is the same as (note the .sum):
{(SinOsc.ar(880, 0, 0.5)+SinOsc.ar(883, 0, 0.5))!2}.play
~~~~~~~

{line-numbers=off}
~~~~~~~
{ SinOsc.ar([xxx, xxx], 0, xxx).sum+
SinOsc.ar([xxx, xxx], 0, xxx).sum+
SinOsc.ar([xxx, xxx], 0, xxx).sum+
SinOsc.ar([xxx, xxx], 0, xxx).sum+
SinOsc.ar([xxx, xxx], 0, xxx).sum+
SinOsc.ar([xxx, xxx], 0, xxx).sum
}.play
~~~~~~~

And finally, we need to create envelopes for each of the partials:

{line-numbers=off}
~~~~~~~
{ (SinOsc.ar([xxx, xxx], 0, xxx).sum *
EnvGen.ar(Env.perc(0.00001, xxx))) +
 (SinOsc.ar([xxx, xxx], 0, xxx).sum *
EnvGen.ar(Env.perc(0.00001, xxx))) +
 (SinOsc.ar([xxx, xxx], 0, xxx).sum *
EnvGen.ar(Env.perc(0.00001, xxx))) +
 (SinOsc.ar([xxx, xxx], 0, xxx).sum *
EnvGen.ar(Env.perc(0.00001, xxx))) +
 (SinOsc.ar([xxx, xxx], 0, xxx).sum *
EnvGen.ar(Env.perc(0.00001, xxx))) +
}.play
~~~~~~~

And let's listen to that. You will note that parenthesis have been put around each sine wave and its envelope multiplication. This is because SuperCollider calculates from left to right, and not giving + and - operators precedence, like in common maths and many other programming languages.

TIP: Operator Precedence - explore how these equations result in different outcomes

{line-numbers=off}
~~~~~~~
2+2*8 // you would expect 18 as the result, but SC returns what?
100/2-10 // here you would expect to get 40, and you get the same in SC. Why?
// now, for this reason it's a good practice to use parenthesis, e.g.,
2+(2*8)
100/(2-10) // if that's what you were trying to do
~~~~~~~

We have now created a reasonable representation of the bell sound that we listened to. The next thing to do is to turn that into a synth definition and make it stereo. Note that we add a general envelope with a doneAction:2, which will remove the synth from the server when it has stopped playing.

{line-numbers=off}
~~~~~~~
SynthDef(\bell, xxxx

// and we can play our new bell
Synth(\bell)
~~~~~~~

This bell has a specific frequency, but it would be nice to be able to pass a new frequency as a parameter. This could be done in many ways, one would be to pass the frequencies of each of the oscillators as arguments to the Synth. This would make the instrument quite flexible, but on the other hand it would weaken its unique character (now that so many more types of bell sounds - with their respective harmonic relationships - can be made with it). So here we decide to keep the same ratios between the partials for this unique bell sound, but a sound that can change in pitch. We find the ratios by dividing the frequencies by the lowest frequency.

{line-numbers=off}
~~~~~~~
[xxx, xxx2, xxx3, xxx4]/xxx
// which gives us this array:
[xxxxxxxxxxxxxxxxxxxxxxxxx]
~~~~~~~

We can now use those ratios in our synth definition

{line-numbers=off}
~~~~~~~
SynthDef(\bell, xxxx

// and we can play the bell with different frequencies
Synth(\bell, [\freq, 440])
Synth(\bell, [\freq, 220])
Synth(\bell, [\freq, 590])
Synth(\bell, [\freq, 1000.rand])
~~~~~~~


## Harmonics GUI

Below you find a Graphical User Interface that allows you to control the harmonics of a fundamental frequency (the slider on the right is the fundamental freq). Here we are also introduced to the Osc UGen, which is a wavetable oscillator that reads its samples from a waveform stored in a buffer.

{line-numbers=off}
~~~~~~~
// we create a SynthDef
SynthDef(\oscsynth, { arg bufnum, freq = 440, ts= 1; 
	a = Osc.ar(bufnum, freq, 0, 0.2) * EnvGen.ar(Env.perc(0.01), timeScale:ts, doneAction:2);
	Out.ar(0, a ! 2);
}).add;

// and then we fill the buffer with our waveform and generate the GUI 
(
var bufsize, ms, slid, cspec, freq;
var harmonics;

freq = 220;
bufsize=4096; 
harmonics=20;

b=Buffer.alloc(s, bufsize, 1);

x = Synth(\oscsynth, [\bufnum, b.bufnum, \ts, 0.1]);

// GUI :
w = SCWindow("harmonics", Rect(200, 470, 20*harmonics+140,150)).front;
ms = SCMultiSliderView(w, Rect(20, 20, 20*harmonics, 100));
ms.value_(Array.fill(harmonics,0.0));
ms.isFilled_(true);
ms.valueThumbSize_(1.0);
ms.canFocus_(false);
ms.indexThumbSize_(10.0);
ms.strokeColor_(Color.blue);
ms.fillColor_(Color.blue(alpha: 0.2));
ms.gap_(10);
ms.action_({ b.sine1(ms.value, false, true, true) }); // set the harmonics
slid=SCSlider(w, Rect(20*harmonics+30, 20, 20, 100));
cspec= ControlSpec(70,1000, 'exponential', 10, 440);
slid.action_({	
	freq = cspec.map(slid.value); 	
	[\frequency, freq].postln;
	x.set(\freq, cspec.map(slid.value)); 
	});
slid.value_(0.3); 
slid.action.value;
SCButton(w, Rect(20*harmonics+60, 20, 70, 20))
	.states_([["Plot",Color.black,Color.clear]])
	.action_({	a = b.plot });
SCButton(w, Rect(20*harmonics+60, 44, 70, 20))
	.states_([["Start",Color.black,Color.clear], ["Stop!",Color.black,Color.clear]])
	.action_({arg sl;
		if(sl.value ==1, {
			x = Synth(\oscsynth, [\bufnum, b.bufnum, \freq, freq, \ts, 1000]);
			},{x.free;});
	});	
SCButton(w, Rect(20*harmonics+60, 68, 70, 20))
	.states_([["Play",Color.black,Color.clear]])
	.action_({
		Synth(\oscsynth, [\bufnum, b.bufnum, \freq, freq, \ts, 0.1]);
	});	
SCButton(w, Rect(20*harmonics+60, 94, 70, 20))
	.states_([["Play rand",Color.black,Color.clear]])
	.action_({
		Synth(\oscsynth, [\bufnum, b.bufnum, \freq, rrand(20,100)+50, \ts, 0.1]);
	});	
)
~~~~~~~

The "Play" and "Play rand" buttons on the interface allow you to hit Enter repeatedly whilst changing the harmonic energy of the sound. Can you synthesise a clarinet or an oboe this way? Can you find the sound of a trumpet? You can get close, but of course each of the harmonics would ideally have their own envelope and amplitude (as we saw in the xylophone synthesis above).



##  Some Additive SynthDefs with routines playing them

The examples above might have raised the question whether all the parameters of the synth could be set from the outside as arguments passed to the synth in the form of arrays. This is possible, of course, but it requires that the arrays are created as inputs when the SynthDef is compiled. In the example below, the partials and the amplitudes of 15 oscillators are set on compilation as the default arguments in respective arrays.

Note the # in front of the arrays in the arguments. It means that they are literal (fixed size) arrays.

{line-numbers=off}
~~~~~~~
(
SynthDef(\addSynthArray, { arg freq=300, dur=0.5, mul=100, addDiv=8, partials = #[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15], amps = #[ 0.30, 0.15, 0.10, 0.07, 0.06, 0.05, 0.04, 0.03, 0.03, 0.03, 0.02, 0.02, 0.02, 0.02, 0.02 ]; 	
	var signal, env;
	env = EnvGen.ar(Env.perc(0.01, dur), doneAction: 2);
	signal = Mix.arFill(partials.size, {arg i;
				SinOsc.ar(
					freq * harmonics[i], 
					0,
					amps[i]	
				)});
	
	Out.ar(0, signal.dup * env)
	}).add
)

// a saw wave sounding wave with 15 harmonics 
Synth(\addSynthArray, [\freq, 200])
Synth(\addSynthArray, [\freq, 300])
Synth(\addSynthArray, [\freq, 400])
~~~~~~~

This is because the synth it is using the default arguments of the SynthDef. Let's try to pass a partials array

{line-numbers=off}
~~~~~~~
Synth(\addSynthArray, [\freq, 400, \partials, {|i| (i+1)+rrand(-0.2, 0.2)}!15])
~~~~~~~

What happened here? Let's scrutinize the partials argument.

{line-numbers=off}
~~~~~~~
{|i| (i+1)+rrand(-0.2, 0.2)}!15
breaks down to
{|i|i}!15
or 
{arg i; i } ! 15
// but we don't want a frequency of zero, so we add 1
{|i| (i+1) }!15
// and then we add random values from -0.2 to 0.2
{|i| (i+1) + rrand(-0.2, 0.2) }!15
// resulting in frequencies such as 
{|i| (i+1) + rrand(-0.2, 0.2) * 440 }!15
~~~~~~~

We can now create a piece that sets new partial frequencies and their amplitude on every note. As mentioned above this could be carefully decided, or simply done randomly. If it is completely random, it might be worth looking into the Rand UGens though, as they allow for a random value to be generated within every synth.

{line-numbers=off}
~~~~~~~
// test the routine here below. uncommend and comment the variables f and a
(
fork {  // fork is basically a Routine
        100.do({
        		// partial frequencies:
         		// f = Array.fill(15, {arg i; i=i+1; i}).postln; // harmonic spectra (saw wave)
         		f = Array.fill(15, {10.0.rand}); // inharmonic spectra (a bell?)
         		// partial amplitudes:
         		// a = Array.fill(15, {arg i; i=i+1; 1/i;}).normalizeSum.postln; // saw wave amps
         		a = Array.fill(15, {1.0.rand}).normalizeSum.postln; // random amp on each harmonic
         	  	Synth(\addSynthArray).set(\harmonics, f, \amps, a);
            		1.wait;
        });
      }  
)
~~~~~~~



{line-numbers=off}
~~~~~~~
(
n = rrand(10, 15);
{ Mix.arFill(n , { 
		SinOsc.ar( [67.0.rrand(2000), 67.0.rrand(2000)], 0, n.reciprocal)
		*
		EnvGen.kr(Env.sine(rrand(2.0, 10) ) )
	}) * EnvGen.kr(Env.perc(11, 6), doneAction: 2, levelScale: 0.75)
}.play;
)

fork {  // fork is basically a Routine
        100.do({
		n = rrand(10, 45);
		"Number of UGens: ".post; n.postln;
		{ Mix.fill(n , { 
			SinOsc.ar( [67.0.rrand(2000), 67.0.rrand(2000)], 0, n.reciprocal)
			*
			EnvGen.kr(Env.sine(rrand(4.0, 10) ) )
		}) * EnvGen.kr(Env.perc(11, 6), doneAction: 2, levelScale: 0.75)
		}.play;
		rrand(5, 10).wait;
		})
}
~~~~~~~


## Using Control to set multiple parameters

There is another way to store and control arrays within a SynthDef. This is using the Control class. The controls are good for passing arrays into running Synths. In order to do this we use the Control UGen inside our SynthDef.

{line-numbers=off}
~~~~~~~
SynthDef("manySines", {arg out=0;
	var sines, control, numsines;
	numsines = 20;
	control = Control.names(\array).kr(Array.rand(numsines, 400.0, 1000.0));
	sines = Mix(SinOsc.ar(control, 0, numsines.reciprocal)) ;
	Out.ar(out, sines ! 2);
}).add;
~~~~~~~

Here we make an array of 20 frequency values inside a Control variable and pass this array to the SinOsc UGen which makes a "multichannel expansion," i.e., it creates a sinewave in 20 succedent audio busses. (If you had a sound card with 20 channels, you'd get a sine out of each channel) But here we mix the sines into one signal. Finally in the Out UGen we use "! 2" which is a multichannel expansion trick that makes this a 2 channel signal (we could have used signal.dup).

{line-numbers=off}
~~~~~~~
b = Synth("manySines");
~~~~~~~

And here below we can change the frequencies of the Control

{line-numbers=off}
~~~~~~~
// our control name is "array"
b.setn(\array, Array.rand(20, 200, 1600)); 
b.setn(\array, {rrand(200, 1600)}!20); 
b.setn(\array, {rrand(200, 1600)}.dup(20));
// NOTE: All three lines above do exactly the same, just different syntax
~~~~~~~

Here below we use DynKlang (dynamic Klang) in order to change the synth in runtime:

{line-numbers=off}
~~~~~~~
(
SynthDef(\dynklang, { arg out=0, freq=110;
	var klank, n, harm, amp;
	n = 9;
	// harmonics
	harm = Control.names(\harm).kr(Array.series(4,1,4));
	// amplitudes
	amp = Control.names(\amp).kr(Array.fill(4,0.05));
	klank = DynKlang.ar(`[harm,amp], freqscale: freq);
	Out.ar(out, klank);
}).add;
)

a = Synth(\dynklang, [\freq, 230]);

a.set(\harm,  Array.rand(4, 1.0, 4.7))
a.set(\freq, rrand(30, 120))
a.set(\amp, Array.rand(4, 0.005, 0.1))
~~~~~~~


## Klang and Dynklang

It can be laborious to build an array of synths and set the frequencies and amplitudes of each. For this we have a UGen called Klang. Klang is a bank of sine oscillators. It is more efficient than the DynKlang, but less flexible. (Don't confuse with Klank and DynKlank which we will explore in the next chapter).

{line-numbers=off}
~~~~~~~
// bank of 12 oscillators of frequencies between 600 and 1000
{ Klang.ar(`[ Array.rand(12, 600.0, 1000.0), nil, nil ], 1, 0) * 0.05 }.play;
~~~~~~~

{line-numbers=off}
~~~~~~~
// here we create synths every 2 seconds
(
{
loop({
	{ Pan2.ar( 
		Klang.ar(`[ Array.rand(12, 200.0, 2000.0), nil, nil ], 0.5, 0)
		* EnvGen.kr(Env.sine(4), 1, 0.02, doneAction: 2), 1.0.rand2) 	
	}.play;
	2.wait;
})
}.fork;
)
~~~~~~~

Klang can not recieve updates to its frequencies nor can it be modulated. For that we use DynKlang (Dynamic Klang).


{line-numbers=off}
~~~~~~~
(
{ 
	DynKlang.ar(`[ 
		[800, 1000, 1200] + SinOsc.kr([2, 3, 0.2], 0, [130, 240, 1200]),
		[0.6, 0.4, 0.3],
		[pi,pi,pi]
	]) * 0.1
}.freqscope;
)

// amplitude modulation
(
{ 
	DynKlang.ar(`[ 
		[800, 1600, 2400, 3200],
		[0.1, 0.1, 0.1, 0.1] + SinOsc.kr([0.1, 0.3, 0.8, 0.05], 0, [1, 0.8, 0.8, 0.6]),
		[pi,pi,pi]
	]
) * 0.1
}.freqscope;
)
~~~~~~~

The following patch shows how a GUI is used to control the amplitudes of the DynKlang oscillator array

{line-numbers=off}
~~~~~~~
(	// create controls directly with literal arrays:
SynthDef(\dynsynth, {| freqs = #[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0], 
	amps = #[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0], 
	rings = #[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]|
	Out.ar(0, DynKlang.ar(`[freqs, amps, rings]))
}).add
)

(
var bufsize, ms, slid, cspec, rate;
var harmonics = 20;
GUI.qt;

x = Synth(\dynsynth).setn(
				\freqs, Array.fill(harmonics, {|i| 110*(i+1)}), 
				\amps, Array.fill(harmonics, {0})
				);

// GUI :
w = Window("harmonics", Rect(200, 470, 20*harmonics+40,140)).front;
ms = MultiSliderView(w, Rect(20, 10, 20*harmonics, 110));
ms.value_(Array.fill(harmonics,0.0));
ms.isFilled_(true);
ms.indexThumbSize_(10.0);
ms.strokeColor_(Color.blue);
ms.fillColor_(Color.blue(alpha: 0.2));
ms.gap_(10);
ms.action_({
	x.setn(\amps, ms.value*harmonics.reciprocal);
}); 
)
~~~~~~~

