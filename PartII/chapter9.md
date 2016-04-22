# Samples and Buffers

SuperCollider offers multiple ways of working with recorded sound. Sampling is one of the key techniques of computer music programming today, originating in tape-based instruments such as the Chamberlin or Mellotrone, but popularised in digital systems with samplers like E-mu Emulator and the Akai S-Series. Sampled sound is also the source of more recent techniques, such as granular and concatenative synthesis.

The first thing we need to know is that a sample is a collection of amplitude values in an array. If we were using 44.1kHz sample rate, we would have 44100 samples in the array if our sound was one second. And twice that amount if our sound was stereo.

We could therefore generate 1 second of whitenoise like this:

{line-numbers=off}
~~~~~~~
Array.fill(44100, {1.0.rand2});
~~~~~~~

The interesting question then is: how do we play these samples? What mechanism will read this and send it to the sound card? For that we use Buffers and UGens that can read them, such as PlayBuf.

## Buffers

In short, a buffer is a collection of values in the memory of the computer. In SuperCollider, buffers are loaded onto the **server** not the language, so in our white noise example above, we would have to find a way to move our collection of values from the language to the server (as that's where they would be played). Buffers can be used to contain all kinds of values in addition to sound, for example control data, gestural data from human movement, sonification data, and so on.

## Allocating a Buffer

In order to create a buffer, we need to allocate it on the server. This is done through an .alloc method:

{line-numbers=off}
~~~~~~~
b = Buffer.alloc(s, 44100 * 4.0, 1); // 4 seconds of sound on a 44100 Hz system, 1 channel

// in the post window we get this information:
//  - > Buffer(0, 176400, 1, 44100, nil) // bufnum, number of samples, channels, sample-rate, path

// If you run the line again, you will see that the bufnum has increased by 1.

// and we can get to this information by calling the server:
b.bufnum;

c = Buffer.alloc(s, 44100 * 4.0, 2); // same but now 2 channels

// This means that we now have twice the amount of samples, but same amount of frames
b.numFrames;
c.numFrames;

// and the number of channels
b.numChannels;
c.numChannels;

// It's clear though that 'c' has twice the amount of samples, even if both buffers have equal amount of frames

b.numFrames * b.numChannels;
c.numFrames * c.numChannels;

~~~~~~~

As mentioned buffers are collection of values in the RAM (Random Access Memory) of the computer. This means that the playhead can jump back and forth in the sound, play it fast or slow, backwards or forwards, and so on. But it also means that, unlike sound file playback from disk (where sound is buffered at regular intervals), the whole sound is stored in the memory of the computer. Try to open your Terminal and then run this line:

{line-numbers=off}
~~~~~~~
a = Array.fill(10, {Buffer.alloc(s,44100 * 8.0, 2)});

// You will see how the memory of the process called scsynth increases
// (scsynth is the name of the SuperCollider server process)

// now run the following line and watch when the memory is de-allocated.
10.do({arg i; a[i].free;})
~~~~~~~

We have now allocated some buffers on the server, but they only contain values of zero. Try playing it:

{line-numbers=off}
~~~~~~~
b.play
// We can load the samples from the server into an array ('a') in the language to check
// This means that the server will send the values from the server to the language over OSC.
b.loadToFloatArray(action: {arg array; a = array; a.postln;})

a.postln // and we see lots of 0s.
~~~~~~~

If we wanted to listen to the noise we created above, we could simply load the array into the buffer.

{line-numbers=off}
~~~~~~~
a = Array.fill(44100, {1.0.rand2}); // 1 second of noise (in an array in the language)
b = Buffer.loadCollection(s, a); // this line loads the array into the buffer (on the server)
b.play // and now we have a beautiful noise!

// We could then observe the samples by getting it back to the language like we did above:
a = Array.fill(44100, {arg i; i=i/10; sin(i)}); // fill a buffer with a sine wave
b = Buffer.loadCollection(s, a); // load the array onto the server
b.play // and now we have a beautiful sine!
b.loadToFloatArray(action: {arg array; a = array; Post << a}) // lots of samples
~~~~~~~

## Reading a soundfile into a Buffer

We can read a sound file into a buffer simply by providing the path to it. This path is either relative to the SuperCollider application (so 'hello.aif' could be loaded if it was next to the SuperCollider application). Note that the IDE allows you to drag a file from your file system into the code document and the full path appears.

{line-numbers=off}
~~~~~~~
b = Buffer.read(s, "sounds/a11wlk01.wav");
b.bufnum; // let's check its bufnum

{ PlayBuf.ar(1, b) ! 2 }.play // the first argument is the number of channels

// We can wrap this into a SynthDef, of course
(
SynthDef(\playBuf,{ arg out = 0, bufnum;
	var signal;
	signal = PlayBuf.ar(1, bufnum, BufRateScale.kr(bufnum));
	Out.ar(out, signal ! 2)
}).add
)
x = Synth(\playBuf, [\bufnum, b.bufnum]) // we pass in either the buffer or the buffer number

x.free; // free the synth 
b.free; // free the buffer

// for many buffers, the typical thing to do is to load them into an array:
b = Array.fill(10, {Buffer.read(s, "sounds/a11wlk01.wav")});

// and then we can access it from the index in the array
x = Synth(\playBuf, [\bufnum, b[2].bufnum])
~~~~~~~

Since the PlayBuf requires information on the number of channels in the sound file, users need to make sure that this is clear, so people often come up with systems like this in their code:

{line-numbers=off}
~~~~~~~
b = Buffer.read(s, Platform.userAppSupportDir+/+"sounds/a11wlk01.wav");

SynthDef(\playMono, { arg out=0, buffer, rate=1;
	Out.ar(out, PlayBuf.ar(1, buffer, rate, loop:1) ! 2)
}).add;

SynthDef(\playMono, { arg out=0, buffer, rate=1;
	Out.ar(out, PlayBuf.ar(2, buffer, rate, loop:1)) // no "! 2"
}).add;

// And then
If(b.numChannels == 1, {
	x = Synth(\playMono, [\buffer, b]) // we pass in either the buffer or the buffer number
}, {
	x = Synth(\playStereo, [\buffer, b]) // we pass in either the buffer or the buffer number
});
~~~~~~~

Note that we don't need the "!2" in the stereo version as that would simply make the left channel expand into the right (and add to the right channel), whereas the right channel would expand into Bus 3.
[Bus 1, Bus 2, Bus 3, Bus 4, Bus 5, etc….]
[  Left , Right ]
           [ Left , Right ]

Let us play a little with Buffer playback in order to get a feel for the possibilities of sound stored in random access memory.

{line-numbers=off}
~~~~~~~
// Change the playback speed
{Pan2.ar(PlayBuf.ar(1, b, MouseX.kr(-1,2), loop:1))}.play

// Scratch around in the file
{ PlayBuf.ar(1, b, MouseX.kr(-1.5, 1.5), loop: 1) ! 2 }.play

// Or perhaps a bit more excitingly 
{
	var speed;
	speed = MouseX.kr(-10, 10);
	speed = speed - DelayN.kr(speed, 0.1, 0.1);
	speed = MouseButton.kr(1, 0, 0.3) + speed ;
	PlayBuf.ar(1, b, speed, loop: 1) ! 2;
}.play

// Another version
{BufRd.ar(1, b, Lag.ar( K2A.ar( MouseX.kr(0,1)) * BufFrames.ir(b), 1))!2}.play

// Jumping to a random location in the buffer using LFNoise0
{PlayBuf.ar(1, b, 1, LFNoise0.ar(12)*BufFrames.ir(b), loop:1)!2}.play

// And so on ….
~~~~~~~

## Recording live sound into a Buffer

Live sound can of course be fed directly into a Buffer for further manipulation. This could be useful if you are recording the sound, transforming it, overdubbing, cutting it up, scratching, and so on. However, in many cases a simple SoundIn UGen might be sufficient (and no Buffers used).

{line-numbers=off}
~~~~~~~
b = Buffer.alloc(s, 44100 * 4.0, 1); // 4 second mono buffer
// Warning, you might get feedback if you're not using headphones
{ RecordBuf.ar(SoundIn.ar(0), b); nil }.play; // run this for at least 4 seconds
{ PlayBuf.ar(1, b) }.play; // play it back
~~~~~~~

SuperCollider really makes this simple. However, the RecordBuf does more than simply recording. Since it loops, you can also overwrite the data that is already in the buffer with the preLevel argument. The preLevel argument is the amount that the data that is in the buffer is multiplied with before it is added to the incoming sound. We can now explore this in a more SuperCollider way of doing things, with SynthDefs and Synths.

{line-numbers=off}
~~~~~~~
SynthDef(\recBuf, { arg buffer=0, recLevel=0.5, preLevel=0.5;
	var in;
	in = SoundIn.ar(0);
	RecordBuf.ar(in, buffer, 0, recLevel, preLevel, loop:1);
}).add;

// we record into the buffer
x = Synth(\recBuf, [\buffer, b, \preLevel, 0]);
x.free;

// and we can play it back using the playBuf synthdef we created above
z = Synth(\playMono, [\buffer, b])
z.free;

// We could also explore the overdubbing of sound (leave this running)
(
x = Synth(\recBuf, [\buffer, b]); // here preLevel is 0.5 by default
z = Synth(\playMono, [\buffer, b, \rate, 1.5]); 
)

// Change the playback rate of the buffer
z.set(\rate, 0.75);

// if we like what we have recorded, we can easily write it to disk as a soundfile:
b.write("myBufRecording.aif", "AIFF", 'int16');
~~~~~~~

It is clear that playing with the recLevel and preLevel of a buffer recording, can create interesting layers of sound, where instrumentalists can record on top of what they already recorded. People could also engage in an "I'm Sitting in a Room" exercise a la Lucier.

Finally, as mentioned at the beginning of this chapter, buffers can contain any data and are not necessarily bound to audio content. In the example below we use the buffer to record mouse values at control rate (which is sample rate / block size) and write that mouse movement to disk in the form of an audio file. 

{line-numbers=off}
~~~~~~~
b = Buffer.alloc(s, (s.sampleRate/s.options.blockSize) * 5, 1); // 5 secs of control rate
{RecordBuf.kr(MouseY.kr, b); SinOsc.ar(1000*MouseY.kr) }.play // recording the mouse
b.write("mouse.aif") // write the buffer to disk, aif is as good format as any

// play it back
b = Buffer.read(s, "mouse.aif")
{SinOsc.ar(1000*PlayBuf.kr(1, b))}.play
~~~~~~~

## BufRd and BufWr

There are other UGens that can be helpful when playing back buffers. BufRd (buffer read) and BufWr (buffer write) are good examples of this, and so is the LoopBuf (from the sc3-plugins that are in the SuperCollider Extensions distribution).

In the example below we use a Phasor to 'drive' the reading of the buffer. This reading has to read sample by sample from the buffer, for example by providing the start and the end sample you want to read:

{line-numbers=off}
~~~~~~~
{ BufRd.ar(1, b, Phasor.ar(0, 1, 0, BufFrames.kr(b))) }.play;

// This way we can easily use SinOsc to modulate the play rate
{ BufRd.ar(1, b, Phasor.ar(0, SinOsc.ar(1).range(0.5, 1.5), 0, BufFrames.kr(b))) }.play;

// And we can also use the mouse to drive the reading 
b = Buffer.read(s, "sounds/a11wlk01.wav");

// Move the mouse!
SynthDef(\scratch, {arg bufnum, pitch=1, start=0, end;
	var signal;
	signal = BufRd.ar(1, bufnum, Lag.ar(K2A.ar(MouseX.kr(1, end)), 0.4));
	Out.ar(0, signal!2);
}).play(s, [\bufnum, b.bufnum, \end, b.numFrames]);
~~~~~~~

## Streaming from disk

If your sound file is very long, it is probably a good idea to stream the sound from disk, just like popular digital audio workstations do. This is because long stereo files would quickly fill up your RAM if working with many sound files.

{line-numbers=off}
~~~~~~~
// We still need a buffer (but we are cueing it, i.e. not filling)
b = Buffer.cueSoundFile(s, Platform.resourceDir +/+ "sounds/a11wlk01-44_1.aiff", 0, 1);

SynthDef(\playcuedBuf,{ arg out = 0, bufnum;
	var signal;
	signal = DiskIn.ar(1, bufnum, loop:1);
	Out.ar(out, signal ! 2)
}).add;

x = Synth(\playcuedBuf, [\bufnum, b]);
~~~~~~~

## Wavetables and wavetable look-up oscillators

Wavetables are a classic method of sound synthesis. It works similarly to the BufRd of a Buffer above, but here we are creating a bespoke wavetable (which can often be visualised for manipulation) and using wavetable look-up oscillators to play the content of the wavetable back. In fact many of the oscillators of SuperCollider use wavetable look-up under the hood, SinOsc being a good example.

Let's start with creating a SynthDef with an Osc (which is a wavetable look-up oscillator). It expects to get a signal in the form of a SuperCollider Wavetable, which is a special format for interpolating oscillators.

{line-numbers=off}
~~~~~~~
(
SynthDef(\wavetable,{ arg out = 0, buffer;
	var signal;
	signal = Osc.ar(buffer, MouseX.kr(60,300)); // mouseX controlling pitch
	Out.ar(out, signal ! 2)
}).add
)

// we then allocate a Buffer with 512 samples (the buffer size must be the power of 2)
b = Buffer.alloc(s, 512, 1); 
b.sine1(1.0, true, true, true); // and we fill it with a sine wave

b.plot // notice something strange?
b.getToFloatArray(action: { |array|  { array[0, 2..].plot }.defer }); // check this

// let's listen to it
a = Synth(\wavetable, [\buffer, b])
a.free;

// You can hear that it sounds very different from a PlayBuf trying to play the same file (and here we get aliasing), since the PlayBuf is not band limited:

{PlayBuf.ar(1, b, MouseX.kr(-1, 10), loop:1)}.play;

// We can then create different waveforms
b.sine1(1.0/[1,2,3,4], true, true, true); //
b.getToFloatArray(action: { |array|  { array[0, 2..].plot }.defer }); // view the wave
a = Synth(\wavetable, [\buffer, b])
a.free;

// A saw wave
b.sine1(0.3/Array.series(90,1,1)*2, false, true, true);
b.getToFloatArray(action: { |array|  { array[0, 2..].plot }.defer });
a = Synth(\wavetable, [\buffer, b])
a.free;

// Random numbers
b.sine1(Array.fill(50, {1.0.rand}), true, true, true);
b.getToFloatArray(action: { |array|  { array[0, 2..].plot }.defer });

a = Synth(\wavetable, [\buffer, b])
a.free;

// We can also use an envelope to fill a buffer
a = Env([0, 1, 0.2, 0.3, -1, 0.3, 0], [0.1, 0.1, 0.1, 0.1, 0.1, 0.1], \sin);
a.plot; // view this envelope 

// But we need to turn the envelope into a signal and then into a wavetable
c = a.asSignal(256).asWavetable;
c.size; // the size of the wavetable is twice the size of the signal... 512

// now we neet to put this wavetable into a buffer:
b = Buffer.alloc(s, 512);
b.setn(0, c);

// play it
a = Synth(\oscplayer, [\bufnum, b.bufnum])
a.free;

// try to load the above without turning the data into a wavetable, i.e.,
a = Env([0, 1, 0.2, 0.3, -1, 0.3, 0], [0.1, 0.1, 0.1, 0.1, 0.1, 0.1], \sin);
c = a.asSignal(256);
b = Buffer.alloc(s, 512);
b.setn(0, c);
a = Synth(\oscplayer, [\bufnum, b.bufnum])

// and you will hear aliasing where the partials of the sound mirror back into the audio range
~~~~~~~

Above we saw how an envelope was turned into a Signal which was then converted to a Wavetable. Signals are a type of a numerical collection in SuperCollider that allows for various math operations. These can be useful for FFT manipulation of data arrays or simply writing data to a file, as in this example:

{line-numbers=off}
~~~~~~~
f = SoundFile.new;
f.openWrite( Platform.userAppSupportDir +/+ "sounds/writetest.wav");
d = Signal.fill(44100, { |i| // one second of sound  
	// 1.0.rand2;  // white noise
	// sin(i/10); // a sine wave
	sin(i/10).cubed;
});
f.writeData(d);
f.close;
~~~~~~~

Below we explore further how Signals can be used with wavetable oscillators.

{line-numbers=off}
~~~~~~~

x = Signal.sineFill(512, [0,0,0,1]);
// We can now operate in many ways on the signal
[x, x.neg, x.abs, x.sign, x.squared, x.cubed, x.asin.normalize, x.exp.normalize, x.distort].flop.flat.plot(numChannels: 9);

c = x.asWavetable;

b = Buffer.alloc(s, 512);
b.setn(0, c); // set the wavetable into the buffer so Osc can read it.

// play it
a = Synth(\wavetable, [\buffer, b])
a.free;

// And the following lines will load a different wavetable into the buffer
c = x.exp.normalize.asWavetable;
b.setn(0, c);
c = x.abs.asWavetable;
b.setn(0, c);
c = x.squared.asWavetable;
b.setn(0, c);
c = x.asin.normalize.asWavetable;
b.setn(0, c);
c = x.distort.asWavetable;
b.setn(0, c);

// try also COsc (Chorusing wavetable oscillator)
{COsc.ar(b, MouseX.kr(60,300))!2}.play

// OscN
{OscN.ar(b, MouseX.kr(60,300))!2}.play // works better with the non-asWavetable example above

// Variable OSC - which can morph between wavetables
b = {Buffer.alloc(s, 512)} ! 9;
x = Signal.sineFill(512, [0,0,0,1]);
[x, x.neg, x.abs, x.sign, x.squared, x.cubed, x.asin.normalize, x.exp.normalize, x.distort].do({arg signal, i; b[i].setn(0, signal.asWavetable)});

{ VOsc.ar(b[0].bufnum + MouseX.kr(0,7), [120,121], 0, 0.3) }.play

// change the content of the wavetables to something random
9.do({arg i; b[i].sine1(Array.fill(512, {1.0.rand2}), true, true, true); })

// VOsc3 
{ VOsc3.ar(b[0].bufnum + MouseX.kr(0,7), [120,121], 0, 0.3) }.play

~~~~~~~

People often want to draw their own sound in a wavetable. We can end this excursion into wavetable synthesis by creating a graphical user interface that allows for the drawing of wavetables.

{line-numbers=off}
~~~~~~~
(
var size = 512;
var canvas, wave, lastPos, lastVal;

w = Window("Wavetable", Rect(100, 100, 1024, 500)).front;
wave = Signal.sineFill(size, [1]);
b = Buffer.alloc(s, size * 2); // double the size for the wavetable

Slider(w, Rect(0, 5, 1024, 20)).action_({|sl| x.set(\freq, sl.value*1000)});  
  UserView(w, Rect(0, 30, 1024, 470))
    .background_(Color.black)
    .animate_(true)
    .mouseMoveAction_({ |me, x, y, mod, btn|
       var pos = (size * (x / me.bounds.width)).floor;
       var val = (2 * (y / me.bounds.height)) - 1;
       val = min(max(val, -1), 1);
       wave.clipPut(pos, val);
       if(lastPos != nil, {
           for(lastPos + 1, pos - 1, { |i|
               wave.clipPut(i, lastVal + (((i - lastPos) / (pos - lastPos)) * (val - lastVal)));
           });
           for(pos + 1, lastPos - 1, { |i|
               wave.clipPut(i, lastVal + (((i - lastPos) / (pos - lastPos)) * (val - lastVal)));
           });
       });
       lastPos = pos;
       lastVal = val;
       b.loadCollection(wave.asWavetable);
       })
       .mouseUpAction_({
           lastPos = nil;
          lastVal = nil;
       })
       .drawFunc_({ |me|
	         Pen.color = Color.white;
           Pen.moveTo(0@(me.bounds.height * (wave[0] + 1) / 2));
           for(1, size - 1, { |i, a|
               Pen.lineTo((me.bounds.width * i /size)@(me.bounds.height * (wave[i] + 1)/2))
           });
           Pen.stroke;
       });
b.loadCollection(wave.asWavetable);
x = {arg freq=440; Osc.ar(b, freq) *0.4 ! 2 }.play;
)
~~~~~~~





// 6) =========  Pitch and time changes  ==========


b = Buffer.read(s, "sounds/a11wlk01-44_1.aiff");

// The most common way
// here double rate (and pitch) results in half the length (time) of the file

(
SynthDef(\playBuf,{ arg out = 0, bufnum;
	var signal;
	signal = PlayBuf.ar(1, bufnum, MouseX.kr(0.2, 4), loop:1);
	Out.ar(out, signal ! 2)
}).add
)

x = Synth(\playBuf, [\bufnum, b.bufnum])
x.free



// we could use PitchShift to change the pitch without changing the time
// PitchShift is a granular synthesis pitch shifter (other techniques include Phase Vocoders)

(
SynthDef(\playBufWPitchShift,{ arg out = 0, bufnum;
	var signal;
	signal = PlayBuf.ar(1, bufnum, 1, loop:1);
	signal = PitchShift.ar(
		signal,	// stereo audio input
		0.1, 			// grain size
		MouseX.kr(0,2),	// mouse x controls pitch shift ratio
		0, 				// pitch dispersion
		0.004			// time dispersion
	);
	Out.ar(out, signal ! 2)
}).add
)

x = Synth(\playBufWPitchShift, [\bufnum, b.bufnum])
x.free


// for time streching check out the Warp0, Warp1 Ugens.

