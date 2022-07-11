# Chapter 4

SuperCollider is a very open environment. It can be used for practically anything sound related, whether that is scientific study of sound, instrument building, DJing, generative composition, or creating interactive installations. For these purposes we often need real-time interaction with the system and this can be done through various ways, but typically through screen-based or hardware interaction. This section will introduce the most common ways of interacting with the SuperCollider language.

## MIDI - Musical Instrument Digital Interface

*MIDI: A popular 80s technology (SC2 Documentation)*

MIDI is one of the most common protocols for hardware and software communication. It is a simple protocol that has proven valuable, <del>although it is currently seen to have gone past its prime.</del> [ I don't think so? The spec had its first update in my lifetime since you wrote this. - Andy ] The key point of using MIDI in SuperCollider is to be able to interact with hardware controllers, synthesizers, and other software. SuperCollider has a strong MIDI implementation and should support everything you might want to do with MIDI.

    // we initialise the MIDI client and the post window will output your devices
    MIDIClient.init;
    // the sources are the input devices you have plugged in
    MIDIClient.sources;
    // the destinations are the devices that can receive MIDI
    MIDIClient.destinations;

### Using MIDI Controllers (Input)

Let's start with exploring MIDI controllers. The MIDI methods that you will use will depend on what type of controller you've got. The following are the input methods of MIDIIn:

* noteOn
* noteOff
* control
* bend
* touch
* polyTouch
* program
* sysex
* sysrt
* smpte

If you were to use a relatively good MIDI keyboard, you would be able to use most of these methods. In the following example we will explore the interaction with a simple MIDI keyboard.

> Virtual MIDI Keyboard
>
> If you don't have a hardware MIDI keyboard you could download a cross platform and open source software MIDI keyboard from here:  http://vmpk.sourceforge.net

    MIDIIn.connectAll; // we connect all the incoming devices
    MIDIFunc.noteOn({arg ...x; x.postln; }); // we post all the args

On the device I'm using whilst writing this (Korg NanoKEY), I get an array formatted thus [127, 60, 0, 1001096579], where the first item is the velocity (how hard I hit the key), the second is the MIDI note, the third is the MIDI channel, and the fourth is the device number (so if you have different devices, you can differentiate between them using this ID).

For the example below, we will use the convenient MIDIdef class to register the definition we want to use for the incoming MIDI messages. Making such definitions is common in SuperCollider, as we make SynthDefs and OSCdefs as well. (XXX HIDdefs?) Let's hook the incoming note and velocity values up to the freq and amp values of a synth that we create. Note that the MIDIdef contains two things, its name and *the function it will trigger on every incoming MIDI note on*! We simply create a Synth inside that function.

    //First we create a synth definition for this example:
    SynthDef(\midisynth1, {arg freq=440, amp=0.1;
        var signal, env;
        signal = VarSaw.ar([freq, freq+2], 0, XLine.ar(0.7, 0.9, 0.13));
        env = EnvGen.ar(Env.perc(0.001), doneAction:2); // this envelope will die out
        Out.ar(0, signal*env*amp);
    }).add;

    Synth(\midisynth1) // let's try it

    // and now we can play the synth
    MIDIdef.noteOn(\mydef, {arg vel, key, channel, device; 
        Synth(\midisynth1, [\freq, key.midicps, \amp, vel/127]);
        [key, vel].postln; 
    });

But the above is not a common synth-like behaviour. Typically you'd hold down the note and it would not be released until you release your finger off the keyboard key. We therefore need to use an [ADSR envelope](https://en.wikipedia.org/wiki/Synthesizer#Attack_Decay_Sustain_Release_.28ADSR.29_envelope).

    //First we create a synth definition for this example:
    SynthDef(\midisynth2, {arg freq=440, amp=0.1, gate=1;
        var signal, env;
        signal = VarSaw.ar([freq, freq+2], 0, XLine.ar(0.7, 0.9, 0.13));
        env = EnvGen.ar(Env.adsr(0.001), gate, doneAction:2);
        Out.ar(0, signal*env);
    }).add;
    // since we added default freq and amp arguments we can try it:
    a = Synth(\midisynth2) // playing 440 Hz
    a.release // and the synth will play until we release it (gate = 0)
    // the adsr envelope in the synth keeps the gate open as long as note is down

    // now let's connect the MIDI
    MIDIIn.connectAll; // we connect all the incoming devices
    MIDIdef.noteOn(\mydef, {arg vel, key, channel, device; 
        Synth(\midisynth2, [\freq, key.midicps, \amp, vel/127]);
        [key, vel].postln; 
    });

What's going on here? The synth definition uses a common trick to create a slight detuning in the frequency in order to make the sound more "analogue" or imperfect. We use a VarSaw that can change the saw waveform and we do change it with the XLine UGen. The synth def has an amp argument for the volume and a gate argument that keeps the synth playing until we tell it to stop. 

But what happened? We play and we get a cacophony of sound. The notes are piling up on top of each other as they are not released. How would you solve this? 

You could put the note into a variable:

    MIDIdef.noteOn(\myOndef, {arg vel, key, channel, device; 
        a = Synth(\midisynth2, [\freq, key.midicps, \amp, vel/127]);
        [key, vel].postln; 
    });
    MIDIdef.noteOff(\myOffdef, {arg vel, key, channel, device; 
        a.release;
        [key, vel].postln; 
    });

And it will release the note when you release your finger. However, now the problem is that if you press another key whilst holding down the first one, the second key will be the Synth that is put into variable 'a', so you have lost the reference to the first one. You can't release it! There is no access to it. Here is where SuperCollider excels as a programming language and makes things so simple and easy compared to data-flow programming environments like Pd or Max/MSP. We just create an array and put our synths into it. Here every note has a slot in the array and we turn the synths on and off depending on the MIDI message:

    a = Array.fill(127, { nil });
    g = Group.new; // we create a Group to be able to set cutoff of all active notes
    c = 6;
    MIDIdef.noteOn(\myOndef, {arg vel, key, channel, device; 
        // we use the key as index into the array as well
        a[key] = Synth(\moog, [\freq, key.midicps, \amp, vel/127, \cutoff, c], target:g);

    });
    MIDIdef.noteOff(\myOffdef, {arg vel, key, channel, device; 
        a[key].release;
    });
    MIDIdef.cc(\modulation, { arg val; c=val.linlin(0, 127, 6, 20); g.set(\cutoff, c) });


### MIDI Communication (Output)

> TIP: Free MIDI Synthesizers
> 
> Some free synths you might want to use in the examples below:
> - OSX : [SimpleSynth]: (http://notahat.com/simplesynth)
> - Linux : [ZynAddSubFX]: (http://zynaddsubfx.sourceforge.net)
> - Windows :[VirtualMIDISynth]: (http://coolsoft.altervista.org/en/virtualmidisynth)

It is equally easy to control external hardware or software with SuperCollider's MIDI functionality. Just as above we initialise the MIDI client and check which devices are available:

    // we initialise the MIDI client and the post window will output your devices
    MIDIClient.init;
    // the destinations are the devices that can receive MIDI
    MIDIClient.destinations;

    // the default device is selected
    m = MIDIOut(0); 
    // or select your own device from the list of destinations
    m = MIDIOut(0, MIDIClient.destinations[0].uid); 
    // we now have a MIDIOut object stored in variable 'm'.
    // now we can use the object to send out MIDI messages:
    m.latency = 0; // we put the latency to 0 (default is 0.2)
    m.noteOn(0, 60, 100); // note on
    m.noteOff(0, 60, 100); // note off

And you could control your device using Patterns:

    Pbind(
        \type, \midi, 
        \midiout, m, 
        \midinote, Prand([60, 62, 63, 66, 69], inf), 
        \chan, 1, 
        \amp, 1, 
        \dur, 0.25
    ).play;

or for example a Task:

    a =[72, 76, 79, 71, 72, 74, 72, 81, 79, 84, 79, 77, 76, 77, 76];
    t = Task({
        inf.do({arg i; // i is the counter and wrapAt can wrap the array
        m.noteOff(0, a.wrapAt(i-1), 100); // note off
        m.noteOn(0, a.wrapAt(i), 100); // note on
        0.25.wait;
        })
    }).play; 

You might have recognised the beginning of a Mozart melody there, but perhaps not, as the note lengths were not correct. How would you solve that? Try to fix the timing of the notes as an exercise. Tip: create a duration array (in var 'd' for example) and put that instead of "0.25.wait;" above. Use the wrapAt(i) to get at the correct duration slot.


## OSC - Open Sound Control

Open Sound Control has become the principal protocol replacing MIDI in the 21st century. It is fast and flexible network protocol that can be used to communicate between applications (like SC Server and sc-lang), between computers (on a local network or the internet), or to hardware (that supports OSC). It is used by musicians and media artists all over the world and it has become so popular that commercial software companies are now supporting it in their software. In many ways it could have been called OMC (Open Media Control) as it is used in graphics, video, 3D software, games, and robotics as well.

OSC is a protocol of communication (how to send messages), but it does not define a standard of **what** to communicate (that's the *open* bit). Unlike MIDI, it can send all kinds of information through the network (integers, floats, strings, arrays, etc.), and the user can define the message names (or address spaces as they are also called). 

There are two things that the user needs to know: the computer's IP address, and the listening Port.

* IP address: Typically something like "194.81.199.106" or locally "127.0.0.1" (localhost)
* Port: You can use any port, but ideally choose a port above 10000.

You have already used OSC in the SendTrig example of chapter 2, but there it was 'under the hood', so to speak, as the communication took place in the SuperCollider classes. 

    n = NetAddr("127.0.0.1", 57120);
    a = OSCdef(\test, { arg msg, time, addr, recvPort; msg.postln; }, '/hello', n);
    n.sendMsg('/hello', 4000.rand); // run this line a few times
    n.sendMsg('/hola', 4000.rand); // try this, but it won't work. Why not?
    a.free;

OSC messages make use of Unix-like address spaces. You know what that is already, as you are used to how the internet uses '/' to indicate a folder down in web addresses. For example here this OSCdef.html document lies in a folder called 'Classes': http://doc.sccode.org/Classes/OSCdef.html together with lots of other documents. The address above is '/hello' (not '/hola'). 

The idea here is that we can send messages directly deep into the internals of our synthesizers or systems, for example like this:

'/synth1/oscillator2/lowpass/cutoff', 12000
'/synth1/oscillator2/frequency', 300
'/light3/red/intensity', 10
'/robot/leftarm/upper/xdegrees', 90

and so on. We are here giving direct messages that are human-readable as well as specific for  the machine. This is very different from how people used to use MIDI where you have no way of naming things, you have to resolve to mapping your things with only 16 channels and often constrained in messaging with numbers from 0-127. 

Try to open [Pure Data](https://puredata.info/downloads) and create a new patch with the following in it:

[dumpOSC 12000]
|
|
[print]

Then send the messages to Pd with this code in SC:

    n = NetAddr("127.0.0.1", 12000);
    n.sendMsg('/hello', 4000.rand); // Pure Data will print this message

Try to do the same with another computer on the same network, but then send it to that computer's IP address:

    n = NetAddr(other_computer_IP_address, 12000);
    n.sendMsg('/hello', 4000.rand);

Use the same Pd patch on that computer, but then run the following lines in SuperCollider:

    a = OSCdef(\test, { arg msg, time, addr, recvPort; msg.postln; }, '/hello', nil);

You notice that there is now 'nil' in the sender address. This allows **any** computer on the network to send to your computer. If you would limit that to a specific net address (for example NetAddr("192.9.12.199", 3000)), it would only be able to receive OSC messages from that specific address/computer. 

Hopefully you have now been able to send OSC messages to another software on your computer, to Pd on another computer, and to SuperCollider on another computer. These examples were on the same network. You might have to change settings in your firewall for this to work over networks, and if you are on an institutional network (such as a University network) you might even have to ask the system administrators to open up for a specific port if the incoming message is coming from **outside** the network (Internally it works without admin changes).

We could end this section by creating a little program that is typical for who people use OSC over networks on the same or different computers. Here below we have the listener:

    // synth definition used in this example
    SynthDef(\osc, {arg freq=220, cutoff=1200;
        Out.ar(0, LPF.ar(Saw.ar(freq, 0.5), cutoff));
    }).add;
    // the four OSC defs, that represent the program functionality
    OSCdef(\createX, { arg msg, time, addr, recvPort; 
        x = Synth(\osc);
    }, '/create', nil); 
    OSCdef(\releaseX, { arg msg, time, addr, recvPort;
        x.free; 
        }, '/free', nil);
    OSCdef(\freqX, { arg msg, time, addr, recvPort; 
        x.set(\freq, msg[1]);
        }, '/freq', nil);
    OSCdef(\cutoffX, { arg msg, time, addr, recvPort;
        x.set(\cutoff, msg[1]);
        }, '/cutoff', nil);

And the other system (another software or another computer) will send something like this:

    n = NetAddr("127.0.0.1", 57120);
    n.sendMsg('/create')
    n.sendMsg('/freq', rrand(100, 2000))
    n.sendMsg('/cutoff', rrand(100, 2000))
    n.sendMsg('/free')

The messages could be wrapped into functionality that is plugged to some GUI, a hardware sensor (pressure sensor and motion tracker for example), or perhaps algorithmically generated together with some animated graphics.


## GUI - Graphical User Interfaces

Show GUIS,
Discuss sending and receiving
Show Pen examples


## HID - Human Interface Devices

SuperCollider has good support for using joysticks, game pads, drawing tablets and other interfaces that work with the HID protocol (A subset of the USB protocol and using the USB port of the computer). 



## Hardware - serial Arduino info

