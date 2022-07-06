# Chapter 3 - The Language


This chapter explores how we use the SuperCollider language to control the SC Server. From a certain perspective we could see the server with its synth definitions as an instrument and the language as the performer or the score.  The SuperCollider language is an interpreted object orientated and functional language written in C/C++ and deriving much of its inspiration from Smalltalk. In many ways it is similar to Python, Ruby, Lua or JavaScript, but these are all different languages, and for good reasons: there is no point in creating a programming language that's the same as another. 

SuperCollider is a powerful language, and as its author James McCartney writes in a 19xx paper:


> Different languages are based on different paradigms and lead to different types of approaches to solve a given problem. Those who use a particular computer language learn to think in that language and can see problems in terms of how a solution would look in that language. (McCartney 2003)


SuperCollider is very open and allows us to do things in multiple different ways. We could talk about different coding or compositional styles. And none are better than others. It depends on what people get used to and what practices are in line with how they already think or would like to think.

Music is a time-based art form. It is largely about scheduling events in time (which is a notational practice) or about performing those events yourself (which is an instrumental practice). SuperCollider is good for both practices and it provides the user with specific functionalities that make sense for a musical programming language, which might seem strange in a general language. This chapter and the next will introduce diverse wasy to control the server through automated loops, through patterns, Graphical User Interfaces, and other interface protocols such as MIDI or OSC.


## Tasks, Routines, forks and loops

We have learned to design synth-graphs with UGens, and wrap them into a SynthDef. We have started and stopped a Synth on the server, but we might ask: then what? How do we make music with SuperCollider? How do we schedule things to happen repeatedly in time?

The most basic way of scheduling things is to create a process that loops and runs the same code repeatedly. Such a process can count, so we can use the counter to access data from arrays and this allows us to use the counter as an index into an array into which we can write anything, perhaps a melody. But first let us look at a basic routine:

    Routine({
        inf.do({arg i;
            "iteration: ".post; i.postln;
            0.25.wait; 
        })
    }).play

This could also be written as:

    fork{
        inf.do({arg i;
            "iteration: ".post; i.postln;
            0.25.wait; 
        })
    }

or, in another example:

    a = fork{
      inf.do({arg i;
      "iteration: ".post; i.postln;
       0.25.wait; 
      })
    };

    a.stop; // run this line later

but the key thing is that we have a routine that serves like an engine that can be paused and woken up again after a certain wait. Try to run the do-loop without a fork:

    // this won't work, as there is no routine involved
    100.do({arg i; "iteration: ".post; i.postln; 0.25.wait; });
    // but this will work, as we are not asking the loop to wait:
    100.do({arg i; "iteration: ".post; i.postln; })

A routine can be played with different clocks (TempoClock, SystemClock, and AppClock) and we will explore them later in this chapter. But here is how we can ask different clocks to play the routines:

    (
    r = Routine.new({
        10.do({ arg a;
            a.postln;
            1.wait;
        });
        0.5.wait;
        "routine finished!".postln;
    });
    )

    SystemClock.play(r); // and then we run it
    r.reset // we have to reset the routine to start it again:
    AppClock.play(r); // here we tell AppClock to play routine r
    r.play(AppClock) // or we can use this syntax
    r.stop; // stop the routine
    r.play; // try to start the routine again... It won't work.

In the last line above we experience that we can't restart a routine after it has stopped. Here is where Tasks come in handy, but they are pauseable processes that behave like routines. (Check the Task helpfile). 

    (
    t = Task({
        inf.do({arg i;
            "iteration is: ".post; i.postln;
            0.25.wait;
        })
    });
    )

    t.play;
    t.pause;
    t.resume;
    t.stop;
    t.play;
    t.reset; 

Let's make some music with a Task. We can put some note values into an array and then ask a Task to loop through that array, repeating the melody we make. Let us create a SynthDef that we would like to use for this piece of music:

    SynthDef(\ch3synth1, {arg freq=333, amp=0.4, pan=0.0, dur=0.41; // the arguments
        var signal, env;
        env = EnvGen.ar(Env.perc(0.001, dur), doneAction:2); // doneAction kills the synth
        signal = LFTri.ar(freq, 0, amp) * env; // the envelope multiplies the signal
        signal = Pan2.ar(signal, pan);
        Out.ar(0, signal);
    }).add;

And here we use a task to create synths from the SynthDef. This is done in time:


    (
    m = ([ 0, 1, 5, 6, 10, 12 ]+48).midicps;
    m = m.scramble; // try to re-evaluate only this line
    t = Task({
        inf.do({arg i;
            Synth(\ch3synth1, [\freq, m.wrapAt(i)]);
            0.25.wait;
        })
    });
    t.play;
    )


In fact we could create a loop that re-evaluates the m.scramble line:


    f = fork{
        inf.do({arg i;	
            m = m.scramble; 
            "SCRAMBLING".postln;
            4.8.wait; // why did I choose 4.8 second wait.
        })
    }


## Patterns

Patterns are interesting methods for creating musical structures in an efficient way. Patterns are high-level abstractions of keys and values that can be 'bound' together to control synths. Patterns use the TempoClock of the language to send control messages to the server. Patterns are related to Events, but those are collections of keys and values that can be used to control synths. 

All this might seem very convoluted, but the key point is that we are operating with default values that can be used to control synths. A principal pattern to understand is the Pbind (a Pattern that **binds** keys to values, such as \frequency (a key) to 440 (a value)). 

    ().play; // run this Event and we observe the posting of default arguments
    Pbind().play; // the event arguments are used in the Pbind.

The Pbind is using the default arguments to play the 'default' synth (one that is defined by SuperCollider), a frequency of 261.6256, amplitude of 0.1, and so on.

    // here we have a Pattern that binds the frequency key to the value of 1000
    Pbind(\freq, 1000, \dur, 0.25).play;

The keys that the patterns play match the arguments of the SynthDef. Let's create a SynthDef that we can fully control with a pattern:

    // the synthdef has the conventional 'freq' and 'amp' arguments, but also our own 'cutoff'
    SynthDef(\patsynth1, { arg out=0, freq=440, amp=0.1,  pan=0, cutoff=1000, gate = 1;
        var signal = MoogFF.ar( Saw.ar(freq, amp), cutoff, 3);
        var env = EnvGen.kr(Env.adsr(), gate, doneAction: 2);
        Out.ar(out, Pan2.ar(signal, pan, env) );
    }).add;
    // we play our 'patsynth1' instrument, and control the cutoff parameter
    Pbind(\instrument, \patsynth1, \freq, 100, \cutoff, 300, \amp, 0.6).play;
    // try this as well:
    Pbind(\instrument, \patsynth1, \freq, 100, \cutoff, 3000, \amp, 0.6).play;

Patterns have some default timing mechanism, so we can control the **duration** until the next event, and we can also set the **sustain** of the note:

    Pbind(\instrument, \patsynth1, \freq, 100, \amp, 0.6, \dur, 0.5).play;
    Pbind(\instrument, \patsynth1, \freq, 100, \amp, 0.6, \dur, 0.5, \sustain, 0.1).play;

All this is quite musically boring, but here is where patterns start to get exciting. There are diverse list patterns that allow us to operate with lists, for example by going sequentially through the list (Pseq), picking random values from the list (Prand), shuffling the list (Pshuf), and so on:

    // here we format it differently, into pairs of keys and values
    Pbind(
        \instrument, \patsynth1, 
        \freq, Pseq([100, 200, 120, 180], inf), // sequencing frequency
        \amp, 0.6, 
        \dur, 0.5
    ).play;

    // we can use list patterns for values to any keys:
    Pbind(
        \instrument, \patsynth1, 
        \freq, Prand([100, 200, 120, 180], inf), 
        \amp, Pseq([0.3, 0.6], inf),
        \dur, Pseq([0.125, 0.25, 0.5, 0.25], inf), 
    ).play;

    Pbind(
        \instrument, \patsynth1, 
        \freq, Pseq([100, 200, 120, 180], inf), 
        \cutoff, Pseq([1000, 2000, 3000], inf), // only 3 items in the list - it loops 
        \amp, Pseq([0.3, 0.6], inf),
        \dur, Pseq([0.125, 0.25, 0.5, 0.25], inf), 
    ).play;

There will be more on patterns later, but at this stage it is a good idea to play with the pattern documentation files, for example the ones found under Streams-Patterns-Events. There is also a fantastic  Practical Guide to Patterns in the SuperCollider Documentation. Under 'Streams-Patterns-Events>A-Practical-Guide' [ TODO think this section has been renamed ? - Andy ]
 
To end this section on patterns, let's simply play a little with Pdefs:

    // here we put a pattern into a variable "a"
    (
    a = Pdef.new(\example1, 
            Pbind(\instrument, \patsynth1, // using our sine synthdef
                \freq, Pseq([220, 440, 660, 880], inf), // freq arg
                \dur, Pseq([0.25, 0.5, 0.25, 0.5], inf);  // dur arg
            )
    );
    )

    a.play;
    a.pause;
    a.resume;

    // but we don't need to:
    (
    Pdef(\example2, 
        Pbind(\instrument, \patsynth1, // using our sine synthdef
            \freq, Pseq.new([720, 770, 990, 880], inf), // freq arg
            \dur, Pseq.new([0.25, 0.5, 0.25, 0.5], inf);  // dur arg
        )
    );
    )

    Pdef(\example2).play;
    Pdef(\example2).pause;
    Pdef(\example2).resume;

    // Now, let's play them both together with a bit of timeshift

    (
    Pdef(\example1).quant_([2, 0, 0]);
    Pdef(\example2).quant_([2, 0.5, 1]); // offset by half a beat
    Pdef(\example1).play;
    Pdef(\example2).play;
    )

    // and without stopping we redefine the example1 pattern:
    (
    Pdef(\example1, 
        Pbind(\instrument, \patsynth1, // using our sine synthdef
            \freq, Pseq.new([
                Pseq.new([220, 440, 660, 880], 4),
                Pseq.new([220, 440, 660, 880], 4) * 1.5], // transpose the melody
                inf),
            \dur, Pseq.new([0.25, 0.125, 0.125, 0.25, 0.5], inf);  // dur arg
        )
    );
    )


## The TempoClock

TempoClock is one of three clocks available for timing organisation in SuperCollider. The others are SystemClock and AppClock. TempoClock is a scheduler like SystemClock, but it schedules in beats rather than milliseconds. AppClock is less accurate, but it can call GUI primitives and therefore to be used when GUI's need update from a clock controlled process.

Let's start by creating a clock, give it the tempo of 1 beat per second (that's 60 bpm), and schedule a function to be played in 4 beats time. The arguments of beats and seconds since SuperCollider was started are passed into the function, and we post those.

    t = TempoClock.new;
    t.tempo = 1;
    t.sched(4, { arg beat, sec; [beat, sec].postln; }); // wait for 4 beats (4 secs);

You will note that the beat is a fractional number. This is because the beat returns the appropriate beat time of the clock's thread. If you prefer to have the beats in whole numbers, you can use the schedAbs method:

    t = TempoClock.new;
    t.tempo = 4; // we make the tempo 240 bpm (240/60 = 4)
    t.schedAbs(4, { arg beat, sec; [beat, sec].postln; }); // wait for 4 beats (1 sec);

If we would like to schedule the function **repeatedly**, we add a number representing the next beat at the end of the function.

    t = TempoClock.new;
    t.tempo = 1;
    t.schedAbs(0, { arg beat, sec; [beat, sec].postln; 1}); 

And with this knowledge we can start to make some music:

    t = TempoClock.new;
    t.tempo = 1;
    t.schedAbs(0, { arg beat, sec; [beat, sec].postln; 1}); 
    t.schedAbs(0, { arg beat, sec; "_Scramble_".scramble.postln; 0.5});

We can try to make some rhythmic pattern with the tempoclock now. Let us just use a simple synth like the one we had above, but now we call it 'clocksynth'.

    // our synth
    SynthDef(\clocksynth, { arg out=0, freq=440, amp=0.5,  pan=0, cutoff=1000, gate = 1;
        var signal = MoogFF.ar( Saw.ar(freq, amp), cutoff, 3);
        var env = EnvGen.kr(Env.perc(), gate, doneAction: 2);
        Out.ar(out, Pan2.ar(signal, pan, env) );
    }).add;
    // the clock
    t = TempoClock.new;
    t.tempo = 2;
    t.schedAbs(0, { arg beat, sec; 
        Synth(\clocksynth, [\freq, 440]);
        if(beat%4==0, { Synth(\clocksynth, [\freq, 440/4, \amp, 1]); });
        if(beat%2==0, { Synth(\clocksynth, [\freq, 440*4, \amp, 1]); });
    1}); 

## TIP: Updating GUI from a TempoClock Process

When you get an error message that sounds like "... cannot be called from this process"
then you need to use an AppClock or put the function into a .defer function:

    {myfunction.value}.defer.

Yet another trick to play sounds in SuperCollider is to use "fork" and schedule a pattern through looping. If you look at the source of .fork (by hitting Cmd+I) you will see that it is essentially a Routine (like above), but it is making our lives easier by wrapping it up in a method of Function.

    (
    var clock, waitTime;
    waitTime = 2;
    clock = TempoClock(2, 0);

    { // a fork
        "starting the program".postln;
        { // and we fork again (play 10 sines)
            10.do({|i|
                Synth(\clocksynth, [\freq, 1000+(rand(1000))]);
                "synth nr : ".post; i.postln;
                (waitTime/10).wait; // wait for 100 milliseconds
            });
            "end of 1st fork".postln;
        }.fork(clock);
        waitTime.wait;
        "finished waiting, now we play the 2nd fork".postln;
        { // and now we play another fork where the frequency is lower
            20.do({|i|
                Synth(\clocksynth, [\freq, 100+(rand(1000))]);
                "synth nr : ".post; i.postln;
                (waitTime/10).wait;
            });
            "end of 2nd fork".postln;
        }.fork(clock);
        "end of the program".postln;
    }.fork(clock);
    )

Note that the interpreter reaches the end of the program before the last fork is finished playing. This is enough about the TempoClock at this stage. We will explore it in more depth later.


## GUI Control

Graphical user interfaces are a very common way for musicians to control their compositions. They serve like a control board for things that the language can do, and to control the server. In the next chapter we will explore interfaces in SuperCollider, but this example is provided in this chapter to give an indication of how the language works.

    // we create a synth (here a oscillator with 16 harmonics)
    (
    SynthDef(\simpleSynth, {|freq, amp|
        var signal, harmonics;
        harmonics = 16;
        signal = Mix.fill(harmonics, {arg i;
            SinOsc.ar(freq*(i+1), 1.0.rand, amp * harmonics.reciprocal/(i+1)) 
        });
        Out.ar(0, signal ! 2);
    }).add;
    )

    (
    var synth, win, freqsl, ampsl;
    // create a GUI window
    win = Window.new("simpleSynth", Rect(100, 100, 300, 90), false).front;
    // and place the frequency and amplitude sliders in the window
    StaticText.new(win, Rect(10,10, 160, 20)).font_(Font("Helvetica", 9)).string_("freq");
    freqsl = Slider.new(win, Rect(40,10, 160, 24)).value_(1.0.rand)
        .action_({arg sl; synth.set(\freq, sl.value*1000;) });
    StaticText.new(win, Rect(10,46, 160, 20)).font_(Font("Helvetica", 9)).string_("amp");
    ampsl = Slider.new(win, Rect(40,46, 160, 24)).value_(1.0.rand)
        .action_({arg sl; synth.set(\amp, sl.value) });
    // a button to start and stop the synth. If the button value is 1 we start it, else stop it
    Button.new(win, Rect(220, 10, 60, 60)).states_([["create"], ["kill"]])
        .action_({arg butt;
            if(butt.value == 1, {
                // the synth is created with freq and amp values from the sliders
                synth = Synth(\simpleSynth, [\freq, freqsl.value*1000, \amp, ampsl.value]);
            },{
                synth.free;
            });
        });
    )


