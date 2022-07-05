# The SuperCollider language

This chapter will introduce the fundamentals for creating and running a simple SuperCollider program. It will introduce the basic concepts needed for further exploration and it will be the only silent chapter in the book. We will learn the basic key orientation practices of SuperCollider, that is how to run code, post into the post window and use the documentation system. We will also discuss the key fundamental things needed to understand and write SuperCollider code, namely: variables, arrays, functions and basic data flow syntax. Having grasped the topics introduced in this chapter, you should be able to write practically anything that you want, although later we will go into Object Orientated Programming, which will make things considerably more effective and perhaps easy. 

## The semicolon, brackets and running a program

The semicolon ";" is what divides one instruction from the next. It defines a *line* of code. After the semicolon, the interpreter looks at next line. There has to be semicolon after each line of code. Forgetting it will give you errors printed in the post console.

This code will work fine if you evaluate only this line:

    "Hello World".postln

But not this, if you evaluate both lines (by highlighting both and evaluating them with Shift+Return):

    "Hello World".postln
    "Goodbye World".postln;

Why not? Because the interpreter (the SC language) will not understand 

    "Hello World".postln"Goodbye World".postln; 

However, this will work:

    "Hello World".postln; "Goodbye World".postln; 

It is up to you how you format your code, but you'd typically want to keep it readable for yourself in the future and other readers too. There is however a style of SC coding used for Tweeting, where the 140 character limit introduces interesting constraints for composers. Below is a Twitter composition by Tim Walters, but as you can see, it is not good for human readability although it sounds good (The language doesn’t care about human readability, but we do):

    play{HPF.ar(({|k|({|i|SinOsc.ar(i/96,Saw.ar(2**(i+k))/Decay.ar(Impulse.ar(0.5**i/k),[k*i+1,k*i+1*2],3**k))}!6).product}!32).sum/2,40)}

It can get tiring to having to select many lines of code and here is where brackets come in handy as they can create a scope for the interpreter. So this following code:

    var freq = 440;
    var amp = 0.5;
    {SinOsc.ar(freq, 0, amp}.play;

will not work unless you highlight all three lines. Imagine if these were 100 lines: you would have to do some tedious scrolling up and down the document. So using brackets, you can simply double click after or before a bracket, and it will highlight all the text between the matching brackets. 

    (
    var freq = 440;
    var amp = 0.5;
    {SinOsc.ar(freq, 0, amp}.play;
    )

## Matching brackets

Often when writing SuperCollider code, you will experience errors whose origin you can’t figure out.   Double clicking between brackets and observe whether they are matching properly is one of the key methods of debugging SuperCollider code.

    (
    "you ran the program and ".post; 
    (44+77).post; " is the sum of 44 and 77".postln;
    "the next line - the interpreter posts it twice as it's the last line".postln;
    )

The following will not work. Why not? Look at the post window.

    (
    (44+77).postln
    55.postln;
    )

Note that the • sign is where the interpreter finds the error.

## The post window

You have already posted into the post window (many other languages use a “print” and “println” for this purpose). But let’s explore the post window a little further. 

    (
    "hello".post; // post something
    "one, two, three".post;
    )


    (
    "hello there".postln; // post something and make a line break
    "one, two, three".postln;
    )

    1+4; // returns 5

    Scale.minor.degrees // returns an array with the degrees in the minor scale

You can also use postf:

    "the first value is %, and the second one is % \n".postf(1111, 9999);

If you are posting a long list you might not get the whole content using .postln, as SC is lazy and doesn’t like printing too long data structures, like lists.

For this purpose use the following:

    Post << "hey"

Example

    Array.fill(1000, {100.rand}).postln; // you see you get ...etc...

Whereas,

    Post << Array.fill(1000, {100.rand}) // you get the whole list

### The Documentation system (The help system)

The documentation system in SuperCollider is a good source for information and learning. It includes introduction tutorials, overviews and documentation for almost every class in SuperCollider. The documentation files typically contain examples of how to use the specific class/UGen, and thus serves as a great source for learning and understanding. Many SC users go straight into the documentation when they start writing code, using it as a template and copy-paste the examples into their projects. 

So if you highlight the word Array in an SC document and hit Cmd+d or Ctrl+d (d for documentation), you will get the documentation for that class. You will see the superclasses/subclasses and learn about all the methods that the Array class has.

Also, if you want to read and browse all the documentation, you can open a help browser:
Help.gui or simply Cmd+D or Ctrl+D (uppercase D).

## Comments

Comments are information written for humans, but ignored by the language interpreter. It is a good practice to write comments where you think you might forget what a certain block of code will do. It is also a communication to another programmer who might read your code. Feel free to write as many comments as you want, but often it might be a better practice to name your variables and function names (we’ll learn later in this section what these words mean) such that you don’t need to add a comment. 

    // This is a comment

    /*
    And this is 
    also a comment
    */

Comments are red by default, but can be any colour (in the Format menu choose ‘syntax colorize’)


## Variables

Here is a mantra to memorise: Variables are containers of some value. They are names or references to values that could change (their value can vary). So we could create a variable that is a property of yourself called age. Every year this variable will increase by one integer (a whole number). So let us try this now:

    var age = 33;
    age = age + 1; // here the variable 'age' gets a new value, or 33 + 1
    age.postln; // and it posts 34

SuperCollider is not strongly typed so there is no need to declare the data type of variables. Data types (in other languages) include : integer, float, double, string, custom objects, etc... But in SuperCollider you can create a variable that contains an integer at one stage, but later contains reference to a string or a float. This can be handy, but one has to be careful as this can introduce bugs in your code.

Above we created a variable ‘age’, and we *declared* that variable by writing ‘var’ in front of it. All variables have to be declared before you can use them. There are two exceptions, all lowercase letters from a to z (note that ’s’ is a special variable that is by default used as a reference to the SC Server) can be used without declaration, and so can ‘environmental’ variables (which can be considered global within a certain context) and they start with the ‘~’ symbol. More on that later. 

    a = 3; // we assign the number 3 to the variable "a"
    a = "hello"; // we can also assign a string to it.
    a = 0.333312; // or a floating point number;
    a = [1, 34, 55, 0.1, "string in a list", \symbol, pi]; // or an array with mixed types

    a // hit this line and we see in the post window what "a" contains

SuperCollider has scope, so if you declare a variable within a certain scope, such as a function, they can have a local value within that scope. So try to run this code (by double clicking behind the first bracket).

    (
    var v, a;
    v = 22;
    a = 33;
    "The value of a is : ".post; a.postln;
    )
    "The value of a is now : ".post; a.postln; // then run this line 


So ‘a’ is a global variable. This is good for prototyping and testing, but not recommended as a good software design. A variable with the name ‘myvar’ could not be global – only single lowercase characters.

If we want longer variable names, we can use environmental variables (using the ~ symbol): they can be seen as global variables, accessible from anywhere in your code

    ~myvar = 333;

    ~myvar // post it;

But typically we just declare the variable (var) in the beginning of the program and assign its value where needed. Environmental variables are not necessary, although they can be useful, and this book will not use them extensively. 

But why use variables at all? Why not simply write the numbers or the value wherever we need it? Let’s take one example that should demonstrate clearly why they are useful:


    {
        // declare the variables
        var freq, oscillator, filter, signal;
        freq = 333; // set the frequency variable
        // create a Saw wave oscillator with two channels
        oscillator = Saw.ar([freq, freq+2]);
        // use a resonant low pass filter on the oscillator
        filter = RLPF.ar(oscillator, freq*4, 0.25);
        // multiply the signal by 0.5 to lower the amplitude 
        signal = filter * 0.5;
    }.play;

As you can see, the ‘freq’ variable is used in various places in the above synthesizer. You can now change the value of the variable to something like 500, and it the frequency will ‘automatically’ be turned into 500 Hz in the left channel, 502 Hz in the right, and the cutoff frequency will be 2000 Hz. So instead of changing these variables throughout the code, you change it in one place and its value magically plugged into every location where that variable is used.

## Functions

Functions are an important feature of SuperCollider and most other programming languages. They are used to encapsulate algorithms or functionality that we only want to write once, but use in different places at various times. They can be seen as a black box or a factory that takes some input, parses it, and returns some output. Just as a sophisticated coffee machine might take coffee beans and water as input, it then grounds the beans, boils the water, brews the coffee, and finally outputs a lovely drink. The key point is that you don’t need (or want) to know precisely how all this happens. It’s enough to know where to fill up the beans and water, and then how to operate the buttons of the machine (strength, number of cups, etc.). The coffee machine is a [black box] (http://en.wikipedia.org/wiki/Black_box).

Functions in SuperCollider are notated with curly brackets ‘{}’

Let’s create a function that posts the value of 44. We store it in a variable ‘f’, so we can *call* it later.

    f = { 44.postln };

When you run this line of code, you see that the SuperCollider post window notifies you that it has been given a function. It does *not* post 44 into the post window. For that we have to call the function, i.e., to ask it to perform its calculation and return some value to us.


    f.value // to call the function we need to get its value

Let us write a more complex function:

    f = {
	    69 + ( 12 * log( 220/440 ) / log(2) )
    };
    f.value // returns the MIDI note 57 (the MIDI note for 220 Hz)

This is a typical function that calculates the midi note of a given frequency in Hz (or cycles per second). Most electronic musicians know that the MIDI note 60 is C, and that 69 is A, and that a is 440 Hz. But how is this calculated? Well the function above does return the MIDI note of 220 Hz. But this is a function without any input (or *argument* as it is called in the lingo). Let’s open up this input channel, by drilling a hole into the black box, and let’s name this argument ‘freq’ as that’s what we want to put in.


    f = { arg freq;
	    69 + ( 12 * log( freq/440 ) / log(2) )
    }

We have now an *input* into our function, an argument named ‘freq’. Note that this argument has been put into the right position inside the calculation. We can now put in any frequency and get the relevant MIDI note.

    f.value(440) // returns 69
    f.value(880) // returns 81
    f.value(261) // returns 59.958555396543 (a fractional MIDI note, close to C (or 60))


The above is a good example of why functions are so great. The algorithm of calculating the MIDI note from frequency is somewhat complex (or nasty?), and we don’t really want to memorise it or write it more than once. We have simply created a black box that we put in to the ‘f’ variable and now we can call it whenever we want without knowing what is inside the black box.

We will be using functions all the time in the coming chapters. It is vital to understand how they receive arguments, process the data, and return a value. 

The final thing to say about functions at this stage is that they can have default values in their arguments. This means that we don’t have to *pass* in all the arguments of the function.


    f = { arg salary, tax=20;
	    var aftertax;
	    aftertax = salary - (salary * (tax/100) )
    }

So here above is a function that calculates the pay after tax, with the default tax rate set at 20%. Of course we can’t be sure that this is the tax rate forever, or in different countries, so this needs to be an argument that can be set in the different contexts. 

    f.value(2000) // here we use the default 20% tax rate
    f.value(2000, 35) // and here the tax percentage has become 35%

##  Tip

SuperCollider contains quite a lot of examples of “syntax sugar”, i.e., where you can write things slightly differently for the sake of brevity (or perhaps aesthetics?). We will explore more of this sugar later in the book, but for now it suffices to mention some related to the function.

You will see the following

    f = { arg string; string.postln; } // we will post the string that comes into the function
    f.value("hi there") // and here we call the function passing "hi there" as the argument.


Often written in this form:

    f = {|string| string.postln;} // arguments can be defined within two pipes '|'
    f.("hi there") // and you can skip the .value and just write a dot (.)

## Arrays, Lists and Dictionaries

Arrays are one of the most useful things to understand and use in computer music. This is where we can store bunch of data (whether pitches, scales, synths, or any other information you might want to reference). A common thing a novice programmer typically does is to create lots of variables for data that could be stored in an array, so let’s dive straight into learning how to use arrays and lists.

An array can be seen as a storage space for things that you need to use later. Like a bag or a box where you keep your things. We typically keep the reference to the array in a variable so we can access it anywhere in our code:

    a = [11, 22, 33, 44, 55]; // we create an array with these five numbers

You will see that the post window posts the array there when you run this line. Now let us try to play a little with the array:

    a[0]; // we get at the first item in the array (most programming languages index at zero)
    a[4] // returns 55, as index 4 into the array contains the value 55
    a[1]+a[4] // returns 77 as 22 plus 55 equals 77
    a.reverse // we can reverse the array
    a.maxItem // the array can tell us what is the highest value


and so on. The array we created above had five defined items in it. But we can create arrays differently, where we fill it algorithmically with any data we’d be interested in:

    a = Array.fill(5, { 100.rand }); // create an array with five random numbers from 0 to 100

What happened here is that we tell the Array class to fill a new array with five items, but then we pass it a function (introduced above) and the function will be evaluated five times. Compare that with:


    a = Array.fill(5, 100.rand ); // create an array with ONE random number from 0 to 100

We can now play a little bit with that function that we pass to the array creation:


    a = Array.fill(5, { arg i; i }); // create a function with the iterator ('i') argument
    a = Array.fill(5, { arg i; (i+1)*11 }); // the same as the first array we created
    a = Array.fill(5, { arg i; i*i });
    a = Array.series(5, 10, 2); // a new method (series). 
    // Fill the array with 5 items, starting at 10, adding 2 in every step.

You might wonder why this is so fantastic or important. The fact is that arrays are used everywhere in computer music. The sound file you will load in later in this book will be stored in an array, with each sample in its own slot in an array. Then you can jump back and forth in the array, scratching, cutting, break beating or whatever you would like to do, but the fact is that this is all done with data (the samples of your soundfile) stored in an array. Or perhaps you want to play a certain scale.


    m = Scale.minor.degrees; // the Scale class will return the degrees of the minor scale

m is here an array with the following values: [ 0, 2, 3, 5, 7, 8, 10 ]. So in a C scale, 0 would be C, 2 would be D (two half notes above C), 3 would be E flat, and so on. We could represent those values as MIDI notes, where 60 is the C note (~ 261Hz). And we could even look at the actual frequencies in Hertz of those MIDI notes. (Those frequencies would be passed to the oscillators as they are expecting frequencies and not MIDI notes as arguments).

    m = Scale.minor.degrees; // Scale class returns the degrees of the minor scale
    m = m.add(12); // you might want to add the octave (12) into your array 
[ TODO : In SuperCollider 3.12.2 this doesn't work at this point, you get ERROR: Primitive '_ArrayAdd' failed. Attempted write to immutable object. - after running one of the lines below it then works. I don't understand why well enough to explain, but have an inkling (the array contains a refernce to Scale.minor.degrees rather than the values [ 0, 2, 3, 5, 7, 8, 10 ] until something forces it to, right ? ) - Andy ]

    m = m+60 // here we simply add 60 to all the values in the array
    m = m.midicps // and here we turn the MIDI notes into their frequency values
    m = m.cpsmidi // but let's turn them back to MIDI values for now

We could now play with the ‘m’ array a little. In an algorithmic composition, for example, you might want to pick a random note from the minor scale

    n = m.choose; // choose a random MIDI note and store it in the variable 'n'
    x = m.scramble; // we could create a melody by scrambling the array
    x = m.scramble[0..3] // scramble the list and select the first 4 notes
    p = m.mirror // mirror the array (like an ascending and descending scale)

You will note that in ‘x = m.scramble’ above, the ‘x’ variable contains an array with a scrambled version of the ‘m’ array. The ‘m’ array is still intact: you haven’t scrambled that one, you’ve simply said “put a scrambled version of ‘m’ into variable ‘x’.” So the original ‘m’ is still there. If you really wanted to scramble ‘m’ you would have to do:
  
    m = m.scramble; // a scrambled version of the 'm' array is put back into the 'm' variable
    // But now it's all scrambled up. Let's sort it into ascending numbers again:
    m = m.sort

Arrays can contain anything, and in SuperCollider, they can contain values of mixed types, such as integers, strings, floats, and so on.


    a = [1, "two", 3.33, Scale.minor] // we mix types into the array.
    // This can be dangerous as the following
    a[0]*10 // will work
    a[1]*10 // but this won't, as you cant multiply the word "two" with 10 


Arrays can contain other arrays, containing other arrays of any dimensions.

    // a function that will create a 5 item array with random numbers from 0 to 10
    f = { Array.fill(5, { 10.rand }) }; // array generating function 
    a = Array.fill(10, f.value);  // create another array with 10 items of the above array
    // But the above was evaluated only once. Why? 
    // Because, you need to pass it a function to get a different array every time. Like this:
    a = Array.fill(10, { f.value } );  // create another array with 10 items of the above array
    // We can get at the first array and see it's different from the second array
    a[0]
    a[1]
    // We could put a new array into a[0] (that slot contains an array)
    a[0] = f.value
    // We could put a new array into a[0][0] (an integer)
    a[0][0] = f.value

Above we added 12 to an array, the minor scale in the previous instance.

    m = []    // Start with an new, empty array
    m.add(12) // but try to run this line many times, the array won't grow forever

### Lists

It is here that the List class becomes useful. 

    l = List.new;   // Start with an new, empty list
    l.add(100.rand) // try to run this a few times and watch the list grow

Lists are like arrays - and implement many of the same methods - but the are slightly more expensive than arrays. In the example above you could simply do ‘a = a.add(100.rand)’ if ‘a’ was an array, but many people like lists for reasons we will discuss later.

### Dictionaries

A dictionary is a collection of items where *keys* are mapped to *values*. Here, keys are keywords that are identifiers for slots in the collection. You can think of this like names for values. This can be quite useful. Let's explore two examples:

    a = Dictionary.new
    a.put(\C, 60)
    a.put(\Cs, 61)
    a.put(\D, 62)
    a[\Ds] = 63 // same as .put
    // and now, let's get the values
    a.at(\D)
    a[\Ds] // same as .at

    a.keys
    a.values
    a.getPairs
    a.findKeyForValue(60)

Imagine how you would do this with an Array. One way would be 


    a = [\C, 60, \Cs, 61, \D, 62, \Ds, 63]
    // we find the slot of a key:
    x = a.indexOf(\D) // 4
    a[x+1]
    // or simply
    a[a.indexOf(\D)+1]

but using an array you need to keep track of the how things are organised and indexed.

Another Dictionary example:

    b = Dictionary.new
    b.put(\major, [ 0, 2, 4, 5, 7, 9, 11 ])
    b.put(\minor, [ 0, 2, 3, 5, 7, 8, 10 ])
    b[\minor]

## Methods?

We have now seen things as 100.rand and a.reverse. How does .rand and .reverse work? Well, SuperCollider is an [Object Orientated language](https://en.wikipedia.org/wiki/Object-oriented_programming) and these are *methods* of the respective classes. So an integer (like 100), has methods like .rand, .midicps, or .neg. It does not have a .reverse method. Why not? Because you can’t reverse a number. However, an array (like [11,22,33,44,55]) can be reversed or added to. We will explore this later in the chapter about Object Orientated programming in SC, but for now it is enough to think that the object (an instantiation of the class) has relevant methods. Or to use an analogy: let’s say we have a class called Car. This class is the information needed to build the car. When we build a Car, we instantiate the class and we have an actual Car. This car can then have some methods, for instance: start, drive, turn, putWipersOn. And these methods could have arguments, like speed(60), or turn(-60). You could think about the object as the noun, the method as the verb, and the argument as the adjective. (As in: John (object) walks (method) fast (adjective)).

    // we create a new car. 4 indicating for example number of seats
    c = Car.new(4); 
    c.start;
    c.drive(40); // the car drives 40 miles per hour
    c.turn(-60); // the car turns 60 degrees to the left

So to really understand a class like Array or List you need to read the documentation and explore the methods available. Note also that the Array is subclassing (or getting methods from its superclass) the ArrayedCollection class. This means that it has all the methods of its superclass. Like a class “Car” might have a superclass called “Vehicle” of which a “Motorbike” would also be a subclass (a sibling to “Car”). You can explore this by peeking under the hood of SC a little:

    Array.openHelpFile // get the documentation of the Array class
    Array.dumpInterface // get the interface or the methods of the Array class
    Array.dumpFullInterface // get the methods of Array's superclasses as well.

You can see that in the .dumpFullInterface method will tell you all the methods Array *inherits* from its superclasses.

Now, this might give you a bit of a brainache, but don’t worry, you will gradually learn this terminology and what it means for you in your musical or sound practice with SuperCollider. Wikipedia is good place to start reading about [Object Oriented Programming](https://en.wikipedia.org/wiki/Object-oriented_programming).

## Conditionals, data flow and control

The final thing we should discuss before we start to make sounds with SuperCollider is how we control data and take decisions. This is about logic, about human thinking, and how to encode such decisions in the form of code. Such logic the basic form of all clever systems, for example in artificial intelligence. In short it is about establishing conditions and then decide what to do with them. For example: if it is raining and I’m going out, I take my umbrella with me, else I leave it at home. It’s about basic logic that humans do all the time throughout the day. And programming languages have ways formalise such conditions, most typically with an if-else statement.

In pseudocode it looks like this:
    if( condition, { then do this }, { else do this });
as in:
    if( rain, { umbrella }, { no umbrella });

So the condition represents a state that is either true or false. If it is true (there is rain), then it evaluates the first function, if false (no rain) it evaluates the second condition.

Another form is a simple if statement where you don’t need to specify what to do if it’s false:
if( hungry, { eat } );

So let’s play with this:


    if( true, { "condition is TRUE".postln;}, {"condition is FALSE".postln;});
    if( false, { "condition is TRUE".postln;}, {"condition is FALSE".postln;});

You can see that true and false are keywords in SuperCollider. They are so called Boolean values. You should not use those as variables (well, you can’t). In digital systems, we operate in binary code, in 1s and 0s. True is associated with 1 and false with 0. 

    true.binaryValue;
    false.binaryValue;

Boolean logic is named after George Boole who wrote an important paper in 1848 (“The Calculus of Logic”) on expressions and reasoning. In short it involves the statements AND, OR, and not.

A simple boolean truth table might look like this
    
    true AND true = true
    true AND false = false
    false AND false = false
    true OR true = true
    true OR false = true
    false or false = false

And also

    true AND not false = true

etc. Let’s try this in SuperCollder code and observe the post window. But first we need to learn the basic syntax for the Boolean operators:

== 	stands for *equal*
!= 	stands for *not equal*
&&	stands for *and*
||	stands for *or*

And we also use comparison operators

">"  stands for *more than*  
"<"  stands for *less than*  
">="  stands for *more than or equal to*  
"<="  stands for *less than or equal to*  


    true == true // returns true
    true != true // returns false (as true does indeed equal true)
    true == false // returns false
    true != false // returns true (as true does not equal false)
    3 == 3 // yes, 3 equals 3
    3 != 4 // true, 3 does not equal 4
    true || false // returns true, as one of the elements are true
    false || false // returns false, as both of the elements are false
    3 > 4 // false, as 3 is less than 4
    3 < 4 // true
    3 < 3 // false
    3 <= 3 // true, as 3 is indeed less than or equal to 3

You might not realise it yet, but knowing what you now know is very powerful and it is something you will use all the time for synthesis, algorithmic composition, instrument building, sound installations, and so on. So make sure that you understand this properly. Let's play with this a bit more in if-statements:

    if( 3==3, { "condition is TRUE".postln;}, {"condition is FALSE".postln;});
    if( 3==4, { "condition is TRUE".postln;}, {"condition is FALSE".postln;});
    // and things can be a bit more complex:
    if( (3 < 4) && (true != false), {"TRUE".postln;}, {"FALSE".postln;});

What happened in that last statement? It asks: is 3 less than 4? Yes. AND is true not equal to false? Yes. Then both conditions are true, and that's what it posts. Note that of course the values in the string (inside the quotation marks) could be anything, we're just posting now. So you could write:

    if( (3 < 4) && (true != false), {"VERDAD".postln;}, {"FALSO".postln;}); 

in Spanish if you'd like, but you could not write this:

verdad == verdad

as the SuperCollider language is in English.

But what if you have lots of conditions to compare? Here you could use a *switch* statement:

    (
    a = 4.rand; // a will be a number from 0 to 4;
    switch(a)
    {0} {"a is zero".postln;} // runs this if a is zero
    {1} {"a is one".postln;} // runs this if a is one
    {2} {"a is two".postln;} // etc.
    {3} {"a is three".postln;};
    )

Another way is to use the case statement, and it might be faster than the switch.

    (
    a = 4.rand; // a will be a number from 0 to 4;
    case
    {a == 0} {"a is zero".postln;} // runs this if a is zero
    {a == 1} {"a is one".postln;} // runs this if a is one
    {a == 2} {"a is two".postln;} // etc.
    {a == 3} {"a is three".postln;};
    )

Note that both in switch and case, the semicolon is only after the last testing condition. (so the line evaluation goes from "case...... to that semicolon" )

## Looping and iterating

The final thing we need to learn in this chapter is looping. Looping is one of the key tricks used in programming. Say we want to generate 1000 synths at once. It would be tedious to write and evaluate 1000 lines of code one after another, but it's easy to loop one line of code 1000 times!

In many programming languages this is done with a [for-loop] (http://en.wikipedia.org/wiki/For_loop):

    for(int i = 0; i > 10, i++) {
    	println("i is now" + i);		
    }

The above code will work in Java, C, JavaScript and many other languages. But SuperCollider is a fully object orientated language, where everything is an object - which can have methods - so an integer can have a method like .neg, or .midicps, but also .do (the loop).

So in SuperCollider we can simply do:


    10.do({ "SCRAMBLE THIS 10 TIMES".scramble.postln; })

What happened is that it loops through the command 10 times and evaluates the function (which scrambles and posts the string we wrote) every time. We could then make a counter:

    (
    var counter = 0;
    10.do({ 
    	counter = counter + 1;
    	"counter is now: ".post; 
    	counter.postln; 
    })
    )

But instead of such counter we can use the argument passed into the function in a loop:

    10.do({arg counter; counter.postln;});
    // you can call this argument whatever you want:
    10.do({arg num; num.postln;});
    // and the typical convention is to use the character "i" (for iteration):
    10.do({arg i; i.postln;});

Let's now try to make a small program that gives us all the prime numbers from 0 to 10000. There is a method of the Integer class that is called isPrime which comes in handy here. We will use many of the things learned in this chapter, i.e., creating a List, making a do loop with a function that has a iterator argument, and then we'll ask if the iterator is a prime number, using an if-statement. If it is (i.e. true), we add it to the list. Finally we post the result to the post window. But note that we're only posting after we've done the 10000 iterations.

    (
    p = List.new;
    10000.do({ arg i; // i is the iteration from 0 to 10000
    	if( i.isPrime, { p.add(i) }); // no else condition - we don't need it
    });
    Post << p;
    )

We can also loop through an Array or a List. Then the do-loop will pick up up all the items of the array and pass it into the function that you write inside the do loop. Additionally, it will add an iterator. So we have two arguments to the function:

    (
    [ 11, 22, 33, 44, 55, 66, 77, 88, 99 ].do({arg item, counter; 
    	item.post; " is in the array at slot: ".post; counter.postln;
    });
    )

So it posts the slot (the counter/iterator always starts at zero), and the item in the list. You can call the arguments whatever you want of course. Example:

    [ 11, 22, 33, 44, 55, 66, 77, 88, 99 ].do({arg aa, bb; aa.post; " is in the array at slot: ".post; bb.postln });

Another looping technique is to use the for-loop:


    for(startValue, endValue, function); // this is the syntax
    for(100, 130, { arg i; i = i+10; i.postln; }) // example

We might also want to use the forBy-loop:

    forBy(startValue, endValue, stepValue, function); // the syntax
    forBy(100, 130, 4, { arg i; i = i+10; i.postln; }) // example


While is another type of loop:

    while (testFunc, bodyFunc); // syntax
    (
    i = 0;
    while ({ i < 30 }, {  i = i + 1; i.postln; });
    )

This is enough about the language. Now is the time to dive into making sounds and explore the synthesis capabilities of SuperCollider. But first let us learn some tricks of peeking under the hood of the SuperCollider language:

## Peaking under the hood

Each UGen or Class in SuperCollider has a class definition in a class file. These files are compiled every time SuperCollider is started and become the application environment we are using. SC is an "interpreted" language. (As opposed to a "compiled" language like C or Java). If you add a new class to SuperCollider, you need to *recompile* the language (there is a menu item for that), or simply start again. 

XXX FIX THIS:
- For checking the sourcefile, type Apple + i (or cmd + i) when a class is highlighted (say SinOsc)
- For checking the implementations of a method (which classes support it), type Apple + Y - poll
- For checking references to a method (which classes support it), type Shift + Apple + Y - poll

    UGen.dumpSubclassList // UGen is a class. Try dumping LFSaw for example

    UGen.browse  // examine methods interactively in a GUI (OSX)
    
    SinOsc.dumpFullInterface  // list all methods for the classhierarchically
    SinOsc.dumpMethodList  // list instance methods alphabetically
    SinOsc.openHelpFile


