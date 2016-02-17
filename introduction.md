# Introduction

Embarking upon learning SuperCollider can be daunting at first. The environment the user is faced with can seem confusing, but let’s be assured that this feeling will be quickly overcome. Learning SuperCollider is in many ways similar to learning an acoustic instrument and it takes hours of practice to reach excellence. However, it should be noted here immediately at the beginning that such excellence need not necessarily be the goal. Indeed, one can write some very good music knowing only a few chords on a guitar or the piano! 

The SuperCollider IDE (Integrated Development Environment) is the same on the Linux, Mac, and Windows operating systems. There might be minor differences, but it looks roughly like this (and this picture contains some labels for further explanation):

![A screenshot of the SuperCollider IDE](images/SCIDE.png)

You will see a coding window on the left, a documentation window, and a post window where the SuperCollider language informs you what it is up to. So let’s dive straight into the first exercise, the famous “Hello World” print into a console (the post window). Simply type “Hello world”.postln; (or “Hola Mundo”.postln; if you like) into the coding window, highlight that text and hit Shift + Return (or go to the XXX menu and select “Interpret code XXX”). If you look at the post window, a “Hello World” has been posted there. Now try to write the same with a spelling mistake, such as “Hello World”.possstln; and you will see an error message appearing. 

## A Tip on the Post Window

    The post window will become your good friend in the future 
    as it will tell you where the errors are, for example 
    spelling mistakes.

SuperCollider is case sensitive, which means that it understands “SinOsc” but has no clue what “Sinosc” means. You will also notice the semicolon (;) at the end of every line written. This is for the SuperCollider interpreter (the language parser) to understand that the current line has ended. The SuperCollider environment consists of three different elements (or processes): the IDE (the text editor that you see in front of you), the language (sclang that is the programming language), and the synth (the audio server that will generate the sound). Later chapters will explain how different languages (such as Java, C++, or Pd) can communicate to the audio server.

Now, let’s dive straight into making some sound, as that’s really the reason you are reading this book. First boot the audio server from the XXX menu and then type:

    {SinOsc.ar(440)}.play;

into the code window and evaluate that line (and if your code is only one line, you can simply place the cursor somewhere in that line and hit Shift+Return). You will hear a sound in the left speaker of your system (yes all oscillators are mono by nature). It might be loud, and you will need to stop it. Hit Cmd+. or Ctrl+. to stop the sound. There is also a menu item to stop the sound, but it is recommended that you simply write these key commands into the motor memory of your fingers.

Let us play a little with this code (hit Cmd/Ctrl+period (Cmd+.) to stop the sound after every line):

    // Octave higher
    {SinOsc.ar(880, 0, 1)}.play;
    // Half the amplitude
    {SinOsc.ar(880, 0, 0.5)}.play;
    // Add another oscillator to multiply the frequency
    {SinOsc.ar(880 * SinOsc.ar(2), 0, 0.5)}.play;
    // Or multiply the the amplitude
    {SinOsc.ar(880, 0, 0.5 * SinOsc.ar(2) )}.play;


What happened here? We are listening to a sine wave oscillator of 880 Hz, or cycles per second. The sine wave oscillator is what is called in most sound programming languages a “unit generator” and it outputs samples according to specific algorithms. So a SinOsc will output samples in different way than a Saw. Furthermore, in the code above we are using the output of one oscillator to multiply parameters of another. But the question arises: which parameters? What is that comma after 880 and the stuff appearing after it?

T>## Using the Documentation
T>
T> In order to understand what these arguments to the SinOsc are, highlight the 
T> SinOsc word and hit Cmd+d. This will call up the documentation file of this unit generator. You will see that the first argument is the frequency (freq), the second is the phase, and the third is multiplication (mul), or amplitude (since if you multiply a signal with a number you are changing its amplitude).

Finally, what we have listened to is a sine wave of 880 Hz, with respective amplitudes of 1 and 0.5. And this is important: signals sent to the sound card of your computer are typically consisting of samples of values between −1 and 1 in amplitude. If the signal is above 1 or below −1, you typically get what is called “clipping” and the sound most likely becomes distorted.

You might also have noticed the information given to you at the bottom of the IDE, that you have used a number of synths (s), ugens (u), groups (g), SynthDefs (sd XXX check). This will be explained in the following chapters, but for now: congratulations with having made some sound in SuperCollider!

## About the Installation

You have now installed and explored SuperCollider on your system. This book does not cover how to install SuperCollider on the different operating systems, but we should note that on any SuperCollider installation, a user specific area is created where you can install your classes, find the synth definitions you have created, and install SC-plugins. This is in your user directory:
Mac: ~/Library/Application Support/SuperCollider
Linux: 
Windows:
