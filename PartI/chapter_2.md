# The SuperCollider Server

The SuperCollider Server, or SC Synth as it's also known, is an elegant and great sounding audio engine. As mentioned earlier, SuperCollider is traditionally separated between a server and a client, that is, an audio server (the SC Synth) and the SuperCollider language client (sc-lang). When the server is booted, it connects to the default audio device (such as internal or external audio cards), but you can set it to any audio device available to your computer (for example using virtual audio routing software like Jack). 

The SC Synth renders audio and has an elegant structure of Busses, Groups, Synths and multitude of UGens, and it works a bit like a modular synth, where the output of certain chain of oscillators and filters can be routed into another module. The audio is created through creating graphs called Synth Definitions. These are definitions of synths, but in a wide sense as they can do practically anything audio related (for example performing audio analysis rather than synthesis).

The SC Synth is a program that runs independently from the SuperCollider IDE or language. You can use any software to control it, like C/C++, Java, Python, Lua, Pure Data, Max/MSP or any other.

This chapter will introduce the SuperCollider server for the most basic purposes of getting started with this amazing engine for audio work. This section will be fundamental for the succeeding chapters.

## Booting the Server

When you "boot the server", you are basically starting a new process on your computer that does not have a GUI (Graphical User Interface). If you observe the list of running processes of your computer, you will see that when you boot the server, a new process will appear (try typing "top" into a Unix Terminal). The server can be booted through a menu command (Menu-> XXX), or through a command line. 

## The 's' Variable

The 's' variable is a unique variable in SuperCollider, as there is a convention that the SC Server has been assigned to this variable. So never assign anything else to this variable.

    // let us explore the 's' variable, that stands for the synth:
    s.postln; // we see that it contains a localhost synth
    s.addr // the address of the synth (IP address and Port)
    s.name // the localhost server is the default server (see Main.sc file)
    s.serverRunning // is it running?
    s.avgCPU // how much CPU is it using right now?
    
    // Let's boot the server. Look at the post window
    s.boot

We can explore creating our own servers with specific ports and IP addresses:

    n = NetAddr("127.0.0.1", 57200); // IP (get it from whatsmyip.org) and port
    p = Server.new("hoho", n); // create a server with the specific net address
    p.makeWindow; // make a GUI window
    p.boot; // boot it

    // try the server:
    {SinOsc.ar(444)}.play(p);
    // stop it
    p.quit;

From the above you might start to think about possibilities of having the server running on a remote computer with various clients communicating to it over network, and yes, that is precisely one of the innovative ideas of SuperCollider 3. You could put any server (with a remote IP address and port) into your server variable and communicate to it over a network. Or have many servers on diverse computers, instructing each of them to render audio. All this is common in SuperCollider practice, but the most common setup is using the SuperCollider IDE to write SC Lang code to control a localhost audio server (localhost meaning "on the same computer"). And that is what we will focus on for a while.


## The Unit Generators

Unit Generators have been the key building blocks of digital synthesis systems, since Max Matthews' Music N systems in the 1960s. Written in C/C++ and compiled as plugins for the SC Server, they encapsulate complex calculations into a simple black box that returns to us - the synth builders or musicians - what we are after, namely an output that could be in the form of a wave or a filter. The Unit Generators, or UGens as they are commonly called, are modular and the output of one can be the input of another. You can think of them like units in a modular synthesizer, for example the Moog:

![A Moog Modular Synth](images/ch2_moog.png)

UGens typically have audio rate (.ar) and control rate (.kr) methods. Some have initialization rate as well. The difference here is that an **audio rate** UGen will output as many samples as the sample rate per second. A computer with 44.1kHz sample rate will require each UGen to calculate 44100 samples per second. **Control rate** is of much lower rate than the sample rate and gives the synth designer the possibility of saving computational power (or CPU cycles) if used wisely. 

## Control Rate

The control rate frequency can be found out by dividing the sample rate with the server's block size. The block size is the number of samples that the server calculates in one go, typically 64 samples, but this can be increased or decreased. So, for example, if your sample rate is 44.1 kHz, and the block size is 64, your control rate will be ~689 Hz (i.e., 44100/64 = 689.0625). This also means that the latency of audio processing using this block size is ~1.5 ms (i.e. 64/44100).

Control rate is used where we are controlling parameters that do not need to be audio rate. For example the frequency of an oscillator. It is perfectly adequate to control frequency changes with control rate, since our ear would not detect any difference whether the resolution was audio rate or not.


    // Here is a sine wave unit generator
    // it has an audio rate method (the .ar)
    // and its argument order is frequency, phase and multiplication
    {SinOsc.ar(440, 0, 1)}.play 
    // now try to run a SinOsc with control rate:
    {SinOsc.kr(440, 0, 1)}.play // and it is inaudible

The control rate SinOsc is inaudible, but it is running fine on the server. We use control rate UGens to **control** other UGens, for example frequency, amplitude, or filter frequency. Let's explore that a little:


    // A sine wave of 1 Hz modulates the 440 Hz frequency
    {SinOsc.ar(440*SinOsc.kr(1), 0, 1)}.play 
    // A control rate sine wave of 3 Hz modulates the amplitude
    {SinOsc.ar(440, 0, SinOsc.kr(3))}.play 
    // An audio rate sine wave of 3 Hz modulates the amplitude
    {SinOsc.ar(440, 0, SinOsc.ar(3))}.play
    // and as you can hear, there is no difference
     
    // 2 Hz modulation of the cutoff frequency of a Low Pass Filter (LPF)
    // we add 1002, so the filter does not go into negative range
    // which might blow up the filter
    {LPF.ar(Saw.ar(440), SinOsc.kr(2, 0, 1000)+1002)}.play 


The beauty of UGens is how one can connect the output of one to the input of another. Oscillator UGens typically output values between -1 and 1, in a certain pattern (e.g., sine wave, saw wave, or square wave) and in a certain frequency. Other UGens such as filters or FFT processing do calculations on an incoming signal and output a new signal. Let's explore one more example of connected UGens that demonstrates their modular power:


    {
    	// we create a slow oscillator in control rate
    	a = SinOsc.kr(1);
    	// the output of 'a' is used to multiply the frequency of a saw wave
    	// resulting in a frequency from 440 to 660. Why?
    	b = Saw.ar(220*(a+2), 0.5);
    	// and here we use 'a' to control amplitude (from -0.5 to 0.5)
    	c = Saw.ar(110, a*0.5);
    	// we add b and c, and use a to control the filter cutoff frequency
    	// we simply added a .range method to a so it now outputs
    	// values between 100 and 2000 at 1 Hz
    	d = LPF.ar(b+c, a.range(100, 2000));
    	Out.ar(0, Pan2.ar(d, 0));
    }.play

This is a simple case study of how UGens can be added (b+c), and used in any calculation (such as a*0.5 - which is an amplitude modulation, creating a tremolo effect) of the signal. For a bit of fun, let's try to use a microphone and make a little effect of your voice:


    {
    	// we take sound in from the sound card
    	a = SoundIn.ar(0);
    	// and we ring modulate using the mouse to control frequency
    	b = a * SinOsc.ar(MouseX.kr(100, 3000));
    	// we also use the mouse (vertical) to control delay
    	c = b + AllpassC.ar(b, 1, MouseY.kr(0.001, 0.2), 2);
    	// and here, instead of Pan2, we simply use an array [c, c]
    	Out.ar(0, [c, c]);
    }.play

A good way to explore UGens is to browse them in the documentation.

    UGen.browse; // XXX check if this works

## The SynthDef

Above we explored UGens by wrapping them in a function and call .play on that function ({}.play). What this does is to turn the function (indicated by {}, as we learned in the chapter 1) into a synth definition that is sent to the server and then played. The {}.play (or Function:play, if you want to peek into the source code – by highlighting "Function:play" and hit, Cmd+I – and explore how SC compiles the function into a SynthDef under the hood) is how many people sketch sound in SuperCollider and it's good for demonstration purposes, but for all real synth building, we need to create a synth definition, or a SynthDef. 

A SynthDef is a pre-compiled graph of unit generators. This graph is written to a binary file and sent to the server over OSC (Open Sound Control - See chapter XXX). This file is stored in the "synthdefs" folder on your system. In a way you could see it as your own VST plugin for SuperCollider, and you don't need the source code for it to work (although it does not make sense to throw that away). 

It is recommended that the SynthDef help file is read carefully and properly understood. The SynthDef is a key class of SuperCollider and very important. It adds synths to the server or writes synth definition files to the disk, amongst many other things. Let's start by exploring how we can turn a unit generator graph function into a synth definition:


    // this simple synth
    {Saw.ar(440)}.play
    // becomes this synth definition
    SynthDef(\mysaw, {
    	Out.ar(0, Saw.ar(440));
    }).add;

You notice that we have done two things: given the function a name (\mysaw), and we've wrapped our saw wave in an 'Out' UGen which defines which 'Bus' the audio is sent to. If you have an 8 channel sound card, you could send audio to any bus from 0 to 7. You could also send it to bus number 20, but we would not be able to hear it then. However, we could put another synth there that routes the audio back onto audio card busses, for example 0-7.

    // you can use the 'Out' UGen in Function:play
    {Out.ar(1, Saw.ar(440))}.play // out on the right speaker

NOTE: There is a difference in the Function-play code and the SynthDef, in that 
we need the Out Ugen in a synth definition to tell the server
which audiobus the sound should go out of. (0 is left, 1 is right)

But back to our SynthDef, we can now try to instantiate it, and create a Synth. (A Synth is an instantiation (child) of a SynthDef). This synth can then be controlled if we reference it with a variable.

    // create a synth and put it into variable 'a'
    a = Synth(\mysaw);
    // create another synth and put it into variable 'b'
    b = Synth(\mysaw);
    a.free; // kill a
    b.free; // kill b

This is obviously not a very interesting synth. It is 'hardcoded', i.e., the parameters in it (such as frequency and amplitude) are static and we can't change them. This is only done in very specific situations, as normally we would like to specify the values of our synth both when initialising the synth and after it has been started. 

In order to open the SynthDef up for specified parameters and enabling it to be changed, we need to put arguments into the UGen function graph. Remember in chapter 1 how we created a function with arguments:

    f = {arg a, b; 
    	c = a + b; 
    	postln("c is now: " + c)
    };
    f.value(2, 3);

Note that you can't write 'f.value', as you will get an error trying to add 'nil' to 'nil' ('a' and 'b' are both nil in the arg slots in the function. To solve that we can give them default values:

    f = {arg a=2, b=3; 
    	c = a + b; 
    	postln("c is now: " + c)
    };
    f.value(22, 33);
    f.value;

So we add the arguments for the synthdef, and we add a Pan2 UGen that enables us to pan the sound from the left (-1) to the right (1). The centre is 0:


    SynthDef(\mysaw, { arg freq=440, amp=0.2, pan=0;
    	Out.ar(0, Pan2.ar(Saw.ar(freq, amp), pan));
    }).add;
    // this now allows us to create a new synth:
    a = Synth(\mysaw); // explore the Synth help file
    // and control it, using the .set, method of the Synth:
    a.set(\freq, 220);
    a.set(\amp, 0.8);
    a.set(\freq, 555, \amp, 0.4, \pan, -1);

This synth definition could be written better and more understandable. Let's say we were to add a filter to the synth, it might look like this:

    SynthDef(\mysaw, { arg freq=440, amp=0.2, pan=0, cutoff=880, rq=0.3;
    	Out.ar(0, Pan2.ar(RLPF.ar(Saw.ar(freq, amp), pan), cutoff, rq));
    }).add;

But this is starting to be hard to read. Let us make the SynthDef easier to read (although for the computer it is the same, as it only cares about where the semicolons (;) are).


    // the same as above, but more readable
    SynthDef(\mysaw, { arg freq=440, amp=0.2, pan=0, cutoff=880, rq=0.3;
    	var signal, filter, panned;
    	signal = Saw.ar(freq, amp);
    	filter = RLPF.ar(signal, cutoff, rq);
    	panned = Pan2.ar(filter, pan);
    	Out.ar(0, panned);
    }).add;


This is roughly how you will write and see other people write synth definitions from now on. The individual parts of a UGen graph are typically put into variables to be more human readable and easier to understand. The exception are SuperCollider tweets (#supercollider) where we have the 140 character limit. We can now explore the synth definition a bit more:


    a = Synth(\mysaw); // we create a synth with the default arguments
    b = Synth(\mysaw, [\freq, 880, \cutoff, 12000]); // we pass arguments
    a.set(\cutoff, 500);
    b.set(\freq, 444);
    a.set(\freq, 1000, \cutoff, 1200);
    b.set(\cutoff, 4000);
    b.set(\rq, 0.1);

## Observing server activity (Poll, Scope and FreqScope)

SuperCollider has various ways to explore what is happening on the server, in addition to the most obvious one: sound itself. Due to the separation between the SC server and the sc-lang, this means that data has to be sent from the server and back to the language, since it's the language that prints or displays the data. The server is just a lean mean sound machine and doesn't care about anything else. Firstly we can try to poll (get) the data from a UGen and post it to the post window:

    // we can explore the output of the SinOsc
    {SinOsc.ar(1).poll}.play // you won't be able to hear this
    // and compare to white noise:
    {WhiteNoise.ar(1).poll}.play // the first arg of noise is amplitude
    // we can explore the mouse:
    {MouseX.kr(10, 1000).poll}.play // nothing to hear
    
    // we can poll the frequency of a sound:
    {SinOsc.ar(LFNoise2.ar(1).range(100, 1000).poll)}.play
    // or we poll the amplitude of it
    {SinOsc.ar(LFNoise2.ar(1).range(100, 1000)).poll}.play
    // and we can add a label (first arg is poll rate, second is label)
    {SinOsc.ar(LFNoise2.ar(1).range(100, 1000).poll(10, "freq"))}.play


People often use poll to explore what is happening in the synth, to debug, or try to understand why something is not working. But it is typically not used in a concrete situation (XXX rephrase?). Another way to explore the server state is to use scope:

    // we can explore the output of the SinOsc
    {SinOsc.ar(1)}.scope // you won't be able to hear this
    // and compare to white noise:
    {WhiteNoise.ar(1)}.scope // the first arg of noise is amplitude
    // we can scope the mouse state (but note the control rate):
    {MouseX.kr(-1, 1)}.scope // nothing to hear
    // the range method maps the output from -1 to 1 into 100 to 1000
    {SinOsc.ar(LFNoise2.ar(1).range(100, 1000))}.scope;
    // same here, we explore the saw wave form at different frequencies
    {Saw.ar(220*SinOsc.ar(0.5).range(1, 10))}.scope


The scope shows amplitude over time, that is: the horizontal axis is **time** and the vertical axis is **amplitude**. This is often called a time-domain view of the signal. But we can also explore the frequency content of the sound, a view we call frequency-domain view. This is achieved by performing an FFT analysis of the signal which is then displayed to the scope (don't worry, this happens 'under the hood' and we'll learn about this in chapter XXX). Now let's explore the freqscope:

    // we see the wave at 1000 Hz, with amplitude modulated
    {SinOsc.ar(1000, 0, SinOsc.ar(0.25))}.freqscope
    // some white noise again:
    {WhiteNoise.ar(1)}.freqscope // random values throughout the spectrum
    // and we can now experienc the power of the scope
    {RLPF.ar(WhiteNoise.ar(1), MouseX.kr(20, 12000), MouseY.kr(0.01, 0.99))}.freqscope
    // we can now explore various wave forms:
    {Saw.ar(440*XLine.ar(1, 10, 5))}.freqscope // check the XLine helpfile
    // LFTri is a non-bandlimited UGen, so explore the mirroring or 'aliasing'
    {LFTri.ar(440*XLine.ar(1, 10, 25))}.freqscope

Futhermore, there is a Spectrogram Quark that shows a spectrogram view of the audio signal, but this is not part of the SuperCollider distribution. However, it's easy to install and we will cover this in the chapter on the Quarks.

## A quick intro to busses and multichannel expansion

Chapter XXX will go deeper into busses, groups, and how to route the audio signals through the SC Server. However, it is important at this stage to understand how the server works in terms of channels (or busses). Firstly, all oscillators are mono. Many newcomers to SuperCollider find it strange that they only hear a signal in their left ear when using headphones running a SinOsc. Well, it would be strange to have it in stereo, quadrophonic, 5.1 or any other format, unless we specifically ask for that! We therefore need to copy the signal into the next bus if we want stereo. The image below shows a rough sketch of how the sc synth works.

![A sketch illustrating busses in the SC Synth](images/ch2_busses.png)

We can see that by default SuperCollider has 8 output channels, 8 input channels, and 112 private audio bus channels (where we can run effects and other things). This means that if you have an 8 channel sound card, you can send a signal out on any of the first 8 busses. If you have a 16 channel sound card, you need to enter the ServerOptions class and change the 'numOutputBusChannels' variable to 16. More on that later, but let's now look at some examples:

    // sound put out on different busses
    { Out.ar(0, LFPulse.ar(220, 0, 0.5, 0.3)) }.play; // left speaker (bus 0)
    { Out.ar(1, LFPulse.ar(220, 0, 0.5, 0.3)) }.play; // right speaker (bus 1)
    { Out.ar(2, LFPulse.ar(220, 0, 0.5, 0.3)) }.play; // third speaker (bus 2)
    
    // Pan2 makes takes the signal and converts it into an array of two signals
    { Out.ar(0, Pan2.ar(PinkNoise.ar(1), 0)) }.scope(8)
    // or we can play it out on bus 6 (and you probably won't hear it)
    { Out.ar(0, Pan2.ar(PinkNoise.ar(1), 0)) }.scope(8)
    // but the above is the same as:
    { a = PinkNoise.ar(1); Out.ar(0, [a, a]) }.scope(8)
    // and (where the first six channels are silent):
    { a = PinkNoise.ar(1); Out.ar(0, [0, 0, 0, 0, 0, 0, a, a]) }.scope(8)
    // however, it's not the same as:
    { Out.ar(0, [PinkNoise.ar(1), PinkNoise.ar(1)]) }.scope(8)
    // why not? -> because we now have TWO signals rather than one

It is thus clear how the busses of the server are represented by an array containing signals (as in: [signal, signal, signal, signal, etc.]). We can now take a mono signal and 'expand' it into other busses. This is called multichannel expansion:

    { SinOsc.ar(440) }.scope(8)
    { [SinOsc.ar(440), SinOsc.ar(880)] }.scope(8)
    // same as:
    { SinOsc.ar([440, 880]) }.scope(8)
    // a trick to 'expand into an array'
    { SinOsc.ar(440) ! 2 }.scope(8)
    // if that was strange, check this:
    123 ! 30

Enough of this. We will explore busses and audio signal routing in chapter XXX later. However, it is important to understand this at the current stage.


## Getting values back to the language

As we have discussed, the SuperCollider language and server are two separate applications. They communicate through the OSC protocol. This means that the communication between the two is **asynchronous**, or in other words, that you can't know precisely how long it takes for a message to arrive. If we would like to do something with audio data in the language, such as visualising it, posting it, or such, we need to send a message to the server and wait for it to respond back. This can happen in various ways, but a typical way of doing this is to use the SendTrig Ugen:


    // this is happening in the language
    OSCdef(\listener, {arg msg, time, addr, recvPort; msg.postln; }, '/tr', n);
    // and this happens in the server
    {
    	var freq;
    	freq = LFSaw.ar(0.75, 0, 100, 900);
    	SendTrig.kr(Impulse.kr(10), 0, freq);
    	SinOsc.ar(freq, 0, 0.5)
    }.play 

What we see above is the SendTrig, sending 10 messages every second to the language (the Impulse triggers those messages). It sends a '/tr' OSC message to port 57120 locally. (Don't worry, we'll explore this later in a chapter on OSC). The OSCdef then has a function that posts the message from the server.

    // this is happening in the language
    OSCdef(\listener, {arg msg, time, addr, recvPort; msg.postln; }, '/tr', n);
    // and this happens on the server
    {
    	var freq;
    	freq = LFSaw.ar(0.75, 0, 100, 900);
    	SendTrig.kr(Impulse.kr(10), 0, freq);
    	SinOsc.ar(freq, 0, 0.5)
    }.play 

A little bit more complex example might involve a GUI (Graphical User Interfaces are part of the language) and synthesis on the server:

    (
    // this is happening in the language
    var win, freqslider, mouseslider;
    win = Window.new.front;
    freqslider = Slider(win, Rect(20, 10, 40, 280));
    mouseslider = Slider2D(win, Rect(80, 10, 280, 280));
    
    OSCdef(\sliderdef, {arg msg, time, addr, recvPort; 
    	{freqslider.value_(msg[3].linlin(600, 1400, 0, 1))}.defer; 
    }, '/slider', n); // the OSC message we listen to
    OSCdef(\sliderdef2D, {arg msg, time, addr, recvPort; 
    	{ mouseslider.x_(msg[3]); mouseslider.y_(msg[4]); }.defer; 
    }, '/slider2D', n); // the OSC message we listen to
    	
    // and this happens on the server
    {
    	var mx, my, freq;
    	freq = LFSaw.ar(0.75, 0, 400, 1000); // outputs 600 to 1400 Hz. Why?
    	mx = LFNoise2.kr(2).range(0,1);
    	my = LFNoise2.kr(2).range(0, 1);
    	SendReply.kr(Impulse.kr(10), '/slider', freq); // sending the OSC message 
    	SendReply.kr(Impulse.kr(10), '/slider2D', [mx, my]); 
    	(SinOsc.ar(freq, 0, 0.5)+RLPF.ar(WhiteNoise.ar(0.3), mx.range(100, 3000), my))!2 ;
    }.play;
     )

We could also write values to a control bus on the server, from which we can read in the language. Here is an example:


    b = Bus.control(s,1); // we create a control bus
    {Out.kr(b, MouseX.kr(20,22000))}.play // and we write the output of some UGen to the bus
    b.get({arg val; val.postln;}); // we poll the puss from the language
    // or even:
    fork{loop{ b.get({arg val; val.postln;});0.1.wait; }}

Check the source of Bus (by hitting Cmd+I) and locate the .get method. You will see that the Bus .get method is using an OSCresponder (XXX is that the case in 3.6?) underneath. It is therefore "asynchronous", meaning that it will not happen in the linear order of your code. (The language is asking server for the value, and the server then sends back to language. This takes time).

Here is a program that demonstrates the asynchronous nature of b.get. The {}.play from above has to be running. Note how the numbered lines of code appear in the post window "in the wrong order"! (Instead of a synchronous posting of 1, 2 and 3, we get the order of 1, 3 and 2). It takes between 0.1 and 10 milliseconds to get the value on a 2.8 GHz Intel computer.

```json
(
x = 0; y= 0;
b = Bus.control(s,1); // we create a control bus
{Out.kr(b, MouseX.kr(20,22000))}.play;
t = Task({
	inf.do({
		"1 - before b.get : ".post; x = Main.elapsedTime.postln;
		b.get({|val| 	
			"2 - ".post; val.postln; 
			y = Main.elapsedTime.postln;
        	"the asynchronious process took : ".post; 
        	(y-x).post; 
        	" seconds".postln;
        	}); //  this value is returned AFTER the next line
    		"3 - after b.get : ".post;  Main.elapsedTime.postln;
    		0.5.wait;
    	})
    }).play;
)
```
This type of communication from the server to the language is not very common. The other way (from language to server) is however. This section is therefore not vital for your work in SuperCollider, but you will at some point stumble into the question of synchronous and asynchronous communication with the server and this section should prepare you for that.


## ProxySpace

SuperCollider is an extremely wide and flexible language. It is profoundly deep and you will find new things to explore for years to come. Typically SC users find their own way of working in the language and then explore new areas when they find they need so, or are curious. 

ProxySpace is one such area. It makes live coding and various on line coding extremely flexible. Effects can be routed in and out of proxies, and source changed. Below you will find a quick examples that are useful when testing UGens or making prototypes for synths that you will write as synthdefs later. ProxySpace is also often used in live coding. Evaluate the code below line by line:

    p= ProxySpace.push(s.boot)
    
    ~signal.play;
    ~signal.fadeTime_(2) // fading in and out in 2 secs
    ~signal= {SinOsc.ar(400, 0, 1)!2}
    ~signal= {SinOsc.ar([400, 404], 0, LFNoise0.kr(4))}
    ~signal= {Saw.ar([400, 404],  LFNoise0.kr(4))}
    ~signal= {Saw.ar([400, 404],  Pulse.ar(2))}
    ~signal= {Saw.ar([400, 404],  Pulse.ar(Line.kr(1, 30, 20)))}
    ~signal= {LFSaw.ar([400, 404],  LFNoise0.kr(4))}
    ~signal= {Pulse.ar([400, 404],  LFNoise0.kr(4))}
    ~signal= {Blip.ar([400, 404],  12, Pulse.ar(2))}
    ~signal= {Blip.ar([400, 404],  24, LFNoise0.kr(4))}
    ~signal= {Blip.ar([400, 404],  4, LFNoise0.kr(4))}
    ~signal= {Blip.ar([400, 404],  MouseX.kr(4, 40), LFNoise0.kr(4))}
    ~signal= {Blip.ar([200, 204],  5, Pulse.ar(1))}
    
    // now let's try to add some effects 
    
    ~signal[1] = \filter -> {arg sig; (sig*0.6)+FreeVerb.ar(sig, 0.85, 0.86, 0.3)}; // reverb
    ~signal[2] = \filter -> {arg sig; sig + AllpassC.ar(sig, 1, 0.15, 1.3 )}; // delay
    ~signal[3] = \filter -> {arg sig; (sig * SinOsc.ar(2.1, 0, 5.44, 0))*0.5}; // tremolo
    ~signal[4] = \filter -> {arg sig; PitchShift.ar(sig, 0.008, SinOsc.ar(2.1, 0, 0.11, 1))}; // pitchshift
    ~signal[5] = \filter -> {arg sig; (3111.33*sig.distort/(1+(2231.23*sig.abs))).distort*0.2}; // distort
    ~signal[1] = nil;
    ~signal[2] = nil;
    ~signal[3] = nil;
    ~signal[4] = nil;
    ~signal[5] = nil;

Another ProxySpace example:

    p = ProxySpace.push(s.boot);
    ~blipper = { |freq=20, nHarm=30, amp=0.1| Blip.ar(freq, nHarm, amp)!2 };
    ~blipper.play;
    ~lfo = { MouseX.kr(10, 100, 1) };
    ~blipper.map(\freq, ~lfo);
    ~blipper.set(\nHarm, 50)
    ~lfn = { LFDNoise3.kr(15, 30, 40) };
    ~blipper.map(\nHarm, ~lfn);
    ~lfn = 30;
    ~blipper.set(\nHarm, 50);

## Ndef

(XXX fill out this section) 

and: jitlib_basic_concepts_02
there is a help-file for Ndef. However, many essential parts are documented in NodeProxy-help which is the common basis for Ndef as well as ProxySpace.
