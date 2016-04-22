# Tuning Systems and Scales





// 1) ========= The SynthDefs ==========




(
// let's make a synth definition with as pure note as possible 
// and we include two envelopes to choose from.
SynthDef(\pure, {arg freq=440, pan=0.0, vol=0.5, envdur=0.5, envType=0;
	var signal, envArray, env;
	env = EnvGen.ar(Env.perc(0.01, envdur), doneAction:2);
	signal = Pan2.ar(SinOsc.ar(freq), pan) * env  * vol;
	Out.ar(0, signal);
}).add;

// and another one almost identical that plays a sample
SynthDef(\puresample, {arg bufnum, rate=1, pan=0.0, vol=0.5, envdur=0.5, envType=0;
	var signal, envArray, env;
	envArray = [	EnvGen.kr(Env.linen(0.05, envdur, 0.1, 1), doneAction:2), 
				EnvGen.kr(Env.perc(0.01, envdur), doneAction:2)
			];
	env = Select.kr(envType, envArray);
	signal = Pan2.ar(PlayBuf.ar(1, bufnum, rate), pan) * env * vol;
	Out.ar(0, signal);
}).add;
)



/*
Tuning systems are generally called "temperaments". There are many
different temperaments, but the equal temperament is the most common
and is used more frequently all over the world.

For a bibliography and further information, visit:
http://www.huygens-fokker.org/scala/

The scales can be found here:
http://www.huygens-fokker.org/docs/scales.zip

A good source for microtonal theory is the Tonalsoft Encyclopedia of Microtonal Music Theory:
http://tonalsoft.com/enc/
*/

// NOTE: Tuning systems are not scales. We can have scales in different tuning systems. 
// NOTE 2: There are various ways of calculating these tunings and opinions differ.










// 2) ========= Equal Temperament ==========





/*
Equal temperament is the most common tuning system in Western music. The octave is divided
logarithmically into series of equal steps, most commonly the twelve tone octave. Other systems
are also used such as the nineteen tone equal temperament (19-TET) or the thirty one tone equal
temperament (31-TET).

Indian and Arabic music often uses a twenty four tone equal temperament (24-TET), although
the instruments are frequently tuned using just intonation.
Javanese Gamelan music is mainly tuned in a 5-TET

About the cent:
The cent is a logaritmic unit (of equal steps) where 1200 represent an octave.
In a 12-TET system, the half-note (of two adjacent keys on a keyboard) is 100 cents.
*/

// the logarithmic formula for pitch (exponential) for equal temperament
// the following contains all 12 notes in an octave
// the formula : fundFreq * 2.pow(n/12);  (or roughly 1.05946309)

// for Equal temperament of 12 notes in an octave these are the values we multiply 
// the fundamental key with:

Array.fill(12, {arg i; 2.pow(i/12);})

// -> returns :	[ 1, 1.0594630943593, 1.1224620483094, 1.1892071150027, 1.2599210498949, 1.33483985417, 1.4142135623731, 1.4983070768767, 1.5874010519682, 1.6817928305074, 1.7817974362807, 1.8877486253634 ]

(
var freq, n_TET;
n_TET = 25; // try, 5 19, 24, 31, 72...
freq = 440;
~eqTempFreqlist = Array.fill(n_TET, {arg i; freq * 2.pow(i/n_TET);});

~eqTempFreqlist = ~eqTempFreqlist.add(freq*2); // let's add the octave finally

[\freqlist, ~eqTempFreqlist].postln;

Task({
	~eqTempFreqlist.do({ arg freq, i; // first arg = item in the list, next arg = the index (i)
		Synth(\pure, [\freq, freq]);
		0.5.wait;
	});
}).start;
)

// now compare the list we've got (in a 12-TET)
~eqTempFreqlist.size
// to this:
[69,70,71,72,73,74,75,76,77,78,79,80].midicps // the midi notes in an octave starting with A

// and further... you can check the MIDI notes of a say 19-TET equal temperament:
~eqTempFreqlist.cpsmidi // where we get floating point MIDI notes




// NOTE (SC LANG): If you are wondering about the ~freqlist.do compare this:

a = [111,222,333,444,555,666,777,888];

a.do({arg item, i; [\item, item, \i, i].postln;}) // a is the array

// to this:

a.size.do({arg i; [\item, a[i], \i, i].postln;}) // a.size is an integer.












// 3) ========= Just Intonation ==========



/*
Just intonation is a very natural system frequently used by vocalists or 
instrumentalists who can easily tune the pitch. Instruments tuned in just
intonation will have to be retuned in order to play in a different scale.
This is the case with the Hapsichord for example.

Just intonation is a method of tuning intervals based exclusively on rational
numbers (integers). It is based on the intervals of the harmonic series. 
Depending on context, the ratio might be different for the same note. 
(e.g. 9/8 and 10/9 for the major second)
Any interval tuned as ratio of whole numbers is a just interval, but usually
it is only ratios with small numbers.

Examples of intervals:

2/1 = octave
3/2 = fifth
4/3 = fourth
5/4 = major third
6/5 = minor third

Some composers (e.g. La Monte Yong and Terry Riley) choose to compose for
just intonation tuned instruments.
*/

// from http://en.wikipedia.org/wiki/Mathematics_of_musical_scales

// a major scale
~justIntFreqlist8 = [1, 9/8, 5/4, 4/3, 3/2, 5/3, 15/8]; 
// a whole 12 note octave
~justIntFreqlist = [1/1, 135/128, 9/8, 6/5, 5/4, 4/3, 45/32, 3/2, 8/5, 27/16, 9/5, 15/8, 2/1];
// and we put in a fundamental note (A)
~justIntFreqlist = ~justIntFreqlist * 440


(

Task({
	~justIntFreqlist.do({ arg freq, i; // 1st arg is the item in the list, 2nd is the index (i)
		Synth(\pure, [\freq, freq]);
		0.7.wait;
	});
}).start;
)

///////// testing some versions

~justIntFreqlist1 = [1/1, 135/128, 9/8, 6/5, 5/4, 4/3, 45/32, 3/2, 8/5, 27/16, 9/5, 15/8, 2/1];
~justIntFreqlist2 = [1/1, 16/15, 9/8, 6/5, 5/4, 4/3, 45/32, 3/2, 8/5, 5/3, 16/9, 15/8, 2/1]
~justIntFreqlist3 = [1/1, 25/24, 9/8, 6/5, 5/4, 4/3, 45/32, 3/2, 8/5, 5/3, 9/5, 15/8, 2/1];
~justIntFreqlist4 = [1/1, 16/15, 9/8, 6/5, 5/4, 4/3, 17/12, 3/2, 8/5, 5/3, 9/5, 15/8, 2/1];

~justIntFreqlist1 = ~justIntFreqlist1 * 440
~justIntFreqlist2 = ~justIntFreqlist2 * 440
~justIntFreqlist3 = ~justIntFreqlist3 * 440
~justIntFreqlist4 = ~justIntFreqlist4 * 440


(
Task({
	13.do({ arg freq, i; // first arg is the item in the list, next arg is the index (i)
		Synth(\pure, [\freq, ~justIntFreqlist1[i], \envdur, 1.56]);
		Synth(\pure, [\freq, ~justIntFreqlist2[i], \envdur, 1.56]);
		//Synth(\pure, [\freq, ~justIntFreqlist3[i], \envdur, 1.56]); // try these as well
		//Synth(\pure, [\freq, ~justIntFreqlist4[i], \envdur, 1.56]);
		1.4.wait;
	});
}).start;
)










// 4) ========= Pythagorean tuning ==========



/*
Pythagorean tuning was invented by the Greek Philosopher Pythagoras in
the 6th century BC. He was interested in harmony, geometry and beans.
The Pythagorean tuning is based on perfect fifths, fourths and octaves.

*/


~pythFreqlist8 = [1, 9/8, 81/64, 4/3, 3/2, 27/16, 243/128, 2/1]; // a major scale

~pythFreqlist = [1, 256/243, 9/8, 32/27, 81/64, 4/3, 729/512, 3/2, 128/81, 27/16, 16/9, 243/128, 2/1];

~pythFreqlist = ~pythFreqlist * 440;

(

Task({
	~pythFreqlist.do({ arg freq, i; // first arg is the item in the list, next arg is the index (i)
		Synth(\pure, [\freq, freq]);
		0.7.wait;
	});
}).start;
)


// now let's compare Equal Temperament to the Pythagorean tuning.
// first we make the equal temperament scale array
~eqTempFreqlist = Array.fill(12, {arg i; 440 * 2.pow(i/12);});
~eqTempFreqlist = ~eqTempFreqlist.add(440*2); // let's add the octave finally

(
Task({
	12.do({ arg freq, i; // first arg is the item in the list, next arg is the index (i)
		Synth(\pure, [\freq, ~pythFreqlist[i], \envdur, 1]);
		Synth(\pure, [\freq, ~eqTempFreqlist[i], \envdur, 1]);
		1.4.wait;
	});
}).start;
)

// and here we compare Just Intonation with Pythagorean tuning.

(
Task({
	12.do({ arg freq, i; // first arg is the item in the list, next arg is the index (i)
		Synth(\pure, [\freq, ~pythFreqlist[i], \envdur, 1]);
		Synth(\pure, [\freq, ~justIntFreqlist[i], \envdur, 1]);
		1.4.wait;
	});
}).start;
)










// 5) ========= Scales ==========




/*
Scales are usually but not necessarily designated for an octave - so they repeat themselves
over all octaves. There are countless scales with different note count, the most common 
in Western music is the diatonic scale. Other common scales (defined by note count) are chromatic
(12 notes), whole tone (6 notes), pentatonic (5 notes) and octatonic (8 notes)

*/


////////////////////////// A DICTIONARY OF SCALES

// James McCartney wrote this dictionary of scales:
// (they are MIDI notes - no microtones and all are equal tempered)
(
z = (
// 5 note scales
	minorPentatonic: [0,3,5,7,10],
	majorPentatonic: [0,2,4,7,9],
	ritusen: [0,2,5,7,9], // another mode of major pentatonic
	egyptian: [0,2,5,7,10], // another mode of major pentatonic
	
	kumoi: [0,2,3,7,9],
	hirajoshi: [0,2,3,7,8],
	iwato: [0,1,5,6,10], // mode of hirajoshi
	chinese: [0,4,6,7,11], // mode of hirajoshi
	indian: [0,4,5,7,10],
	pelog: [0,1,3,7,8],
	
	prometheus: [0,2,4,6,11],
	scriabin: [0,1,4,7,9],
	
// 6 note scales
	whole: (0,2..10),
	augmented: [0,3,4,7,8,11],
	augmented2: [0,1,4,5,8,9],
	
	// hexatonic modes with no tritone
	hexMajor7: [0,2,4,7,9,11],
	hexDorian: [0,2,3,5,7,10],
	hexPhrygian: [0,1,3,5,8,10],
	hexSus: [0,2,5,7,9,10],
	hexMajor6: [0,2,4,5,7,9],
	hexAeolian: [0,3,5,7,8,10],
	
// 7 note scales
	ionian: [0,2,4,5,7,9,11],
	dorian: [0,2,3,5,7,9,10],
	phrygian: [0,1,3,5,7,8,10],
	lydian: [0,2,4,6,7,9,11],
	mixolydian: [0,2,4,5,7,9,10],
	aeolian: [0,2,3,5,7,8,10],
	locrian: [0,1,3,5,6,8,10],
	
	harmonicMinor: [0,2,3,5,7,8,11],
	harmonicMajor: [0,2,4,5,7,8,11],
	
	melodicMinor: [0,2,3,5,7,9,11],
	bartok: [0,2,4,5,7,8,10], // jazzers call this the hindu scale
	
	// raga modes
	todi: [0,1,3,6,7,8,11], // maqam ahar kurd
	purvi: [0,1,4,6,7,8,11],
	marva: [0,1,4,6,7,9,11],
	bhairav: [0,1,4,5,7,8,11],
	ahirbhairav: [0,1,4,5,7,9,10],
	
	superLocrian: [0,1,3,4,6,8,10],
	romanianMinor: [0,2,3,6,7,9,10], // maqam nakriz
	hungarianMinor: [0,2,3,6,7,8,11],	
	neapolitanMinor: [0,1,3,5,7,8,11],
	enigmatic: [0,1,4,6,8,10,11],
	spanish: [0,1,4,5,7,8,10],
	
	// modes of whole tones with added note:
	leadingWhole: [0,2,4,6,8,10,11],
	lydianMinor: [0,2,4,6,7,8,10],
	neapolitanMajor: [0,1,3,5,7,9,11],
	locrianMajor: [0,2,4,5,6,8,10],
	
// 8 note scales
	diminished: [0,1,3,4,6,7,9,10],
	diminished2: [0,2,3,5,6,8,9,11],
	
// 12 note scales
	chromatic: (0..11)
);
)
z.at('chromatic').postln;

// now we try one of those scales

(
x = z.at('hirajoshi').copy; // test the scales above by replacing the name
x = x.add(12); // add the octave
x = x.mirror;

Task({
	x.do({ arg ratio, i; // first arg is the item in the list, next arg is the index (i)
		Synth(\pure, [\freq, (69+ratio).midicps, \envdur, 0.94]);
		0.41.wait;
	});
}).start;
)

(
// do we get a nice melody?
Task({
	x.mirror.do({ arg ratio, i; // first arg is the item in the list, next arg is the index (i)
		Synth(\pure, [\freq, (69+x.choose).midicps, \envdur, 0.84]);
		0.18.wait;
	});
}).start;
)










// 6) ========= The Scala Library ==========



/*
For a proper exploration of scales we will use the Scala project and the SCL class
written in SuperCollider to use the Scala files.

The Scale Archive can be found here (with over 3000 scales):
http://www.huygens-fokker.org/docs/scales.zip

And the SuperCollider class that interfaces with the archive is written by Jascha Narveson 
and can be found here: http://jnarveson.web.wesleyan.edu/sc/SCL/SCL.zip

*/
// note that you have to provide the path to where you install your Scala libaray
// for example "~/scwork/scl/"
// which means SCL.new("~/scwork/scl/degung5.scl".standardizePath, 440);


x = SCL.new("degung5.scl".standardizePath, 440);
x.name
x.steps // how many notes are there in the scale
x.getRatios

z = x.getRatios.mirror;

(
Task({
	z.do({ arg ratio, i; // first arg is the item in the list, next arg is the index (i)
		Synth(\pure, [\freq, 440*ratio]);
		0.3.wait;
	});
}).start;
)


(
x = SCL.new("cairo.scl".standardizePath, 440);
z = x.getRatios.mirror;

Task({
	z.do({ arg ratio, i; // first arg is the item in the list, next arg is the index (i)
		Synth(\pure, [\freq, 440*ratio]);
		0.3.wait;
	});
}).start;
)


(
x = SCL.new("kayolonian_s.scl".standardizePath, 440);
z = x.getRatios.mirror;

Task({
	z.do({ arg ratio, i; // first arg is the item in the list, next arg is the index (i)
		Synth(\pure, [\freq, 440*ratio]);
		0.3.wait;
	});
}).start;
)











// 7) ========= Using Samples ==========


// First we load a sound:
// we get a sound with a simple tone (replace this sound with your own)
b = Buffer.read(s, "sounds/xylo/02.aif");



/// The pythagorean scale:

~pythFreqlist8 = [1, 9/8, 81/64, 4/3, 3/2, 27/16, 243/128, 2/1]; // a major scale

~pythFreqlist8 = ~pythFreqlist8.mirror;

(
{
	~pythFreqlist8.do({arg item;
		Synth(\puresample, [\bufnum, b.bufnum, \rate, item, \envType, 1]);
		Synth(\pure, [\freq, 752*item]); // 752 is just rough freq of the 02.aif sample
		0.5.wait;
	});
}.fork
)





x = SCL.new("degung5.scl".standardizePath, 440);

x = SCL.new("diaconv6144.scl".standardizePath, 440);

x = SCL.new("bagpipe1.scl".standardizePath, 440);

x.name
x.steps // how many notes are there in the scale
x.getRatios


Synth(\puresample, [\bufnum, b.bufnum, \rate, 1, \envType, 0]);
Synth(\pure, [\freq, 752]);


// p is our scale
p = x.getRatios
p.size

p = p.mirror // up and down again! (See Array helpfile for .mirror)

(
{
	p.do({arg item;
		Synth(\puresample, [\bufnum, b.bufnum, \rate, item, \envType, 1]);
		Synth(\pure, [\freq, 752*item]); // 752 is just rough freq of the 02.aif sample
		0.5.wait;
	});
}.fork
)










// 8) ========= The Scale and Tuning Classes ==========



/*
A recent addition (2008) to the power of SuperCollider are the Scale and Tuning classes.
They make encapsulate and simplify the things we have done above in easy to use
methods of the Scale and Tuning libraries.
*/

// an example

// we choose a minor scale:

a = Scale.minor;
a.degrees; 
a.semitones;	
a.cents;	
a.ratios;	

d = Pdef(\minor_et12, Pbind(\scale, a, \degree, Pseq((0..7) ++ (6..0), inf), \dur, 0.5, \amp, 0.1)).play;

// we choose a tuning:
t = Tuning.just; // just intonation
a.tuning_(t);

e = Pdef(\minor_just, Pbind(\scale, a, \degree, Pseq((0..7) ++ (6..0), inf), \dur, 0.5, \amp, 0.1)).play;


// So let's listen to equal tempered and just intonation together. You can hear the beating
(
a = Scale.minor;
t = Tuning.et12; // equal tempered tuning 
a.tuning_(t);
d = Pdef(\minor_et12, Pbind(\scale, a, \degree, Pseq((0..7) ++ (6..0), inf), \dur, 0.5, \amp, 0.1)).play;

b = Scale.minor;
t = Tuning.just; // just intonation
b.tuning_(t);
e = Pdef(\minor_just, Pbind(\scale, b, \degree, Pseq((0..7) ++ (6..0), inf), \dur, 0.5, \amp, 0.1)).play;
)


// check the scale directory

Scale.directory

// and the available tunings

Tuning.directory




