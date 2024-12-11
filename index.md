## Development Log 1 - December 10th 2024
I need to get my hands on a Raspberry Pi (preferably a Compute Module) and a compatible microphone/speaker to test the 'beep' functions. I've already built the framework for recording audio and transmitting it via a series of binary beep sounds at two distinct frequencies. These can be captured by a receiving device and decoded.

It's still early days, but here’s the progress so far: GitHub Repository

I may need to pause development until I acquire the necessary hardware. For now, I’ll try to continue working using a development VM. If that’s not feasible, I’ll focus on writing some of the audio processing logic in C++ in the meantime.

<img src=".\assets\img\techpriest.jpg" alt="TechPriest" class="inline"/>

## Development Log 0 - December 9th 2024
I’ve started work on an audio processing project. For a small Linux-based device, I explored several audio processing libraries and decided to work with ALSA in C (alsa/asoundlib.h).

So far, I’ve successfully configured the microphone to capture audio with a delay, allowing spoken words to be recorded. I’m prioritizing performance and real-time audio capture, especially given the device’s hardware constraints.

The code is in a private repo for now, but I plan to make it public once the project structure and README are polished.

If anyone has recommendations for optimizing real-time audio capture or knows of alternative libraries I should explore, I’d love to hear your thoughts!

More updates soon!

## Breaking Baldi - Part 1 : The Impossible Question
<img src=".\assets\img\p1\breaking_baldi.png" alt="Baldi" class="inline"/>

Baldi's Basics is a self-described "state-of-the-art, fully 3D interactive, fun-time educational game that teaches a slew of subjects!" It's also quite creepy in the creepiest-of-pasta sort of way! 

Unfortunately for you, Baldi really doesn’t like it when you mess up. Particularly in “Everyone’s favorite subject, Math!”.  If you’ve ever played the game, you may believe that Math is the only subject on the table in Baldi’s world. However, I’m about to turn it into a fun and educational reverse engineering experience! ;) 

Night time update:

A quick addition to this, I made an assumption about transmitting speech in audio that turned out to be difficult to do without a hefty processing price (or latency connecting to a speech recognition cloud service) which is going to be a big no no. Found a solution and will be testing that tomorrow.

##### What a Creep... Or, Welcome to Baldi's Basics!
<img src=".\assets\img\p1\baldi_enter.PNG" alt="Baldi" class="inline"/>


You quickly realize that things are not quite what they seem in Baldi’s educational horror show when one of the questions you’re required to answer suddenly becomes incomprehensible, a la Zalgo, and you are unable to appease Baldis extreme expectations. 
“Unfair!” I hear you shouting. How can we possible answer a question that is so jumbled that is clearly designed for us to fail?
Well, anything is possible through the magic of Reverse Engineering, memory manipulation, and debugging!

For this exercise, we will be using Cheat Engine. In this specific case, the use of Cheat Engine just made sense as I needed to have built in Mono Dissection tools (for Unity C#) and the ability to quickly manipulate instantiated C# classes in memory.

To begin to discover the answer to the Impossible Question, we need to attach the Cheat Engine software to the game executable (BALDI.EXE), and activate the available Mono Features.

##### Attaching to the game executable:
<img src=".\assets\img\p1\baldi_exe.PNG" alt="Baldi" class="inline"/>

##### Activating the features:
<img src=".\assets\img\p1\mono_dissect.PNG" alt="MonoDissect" class="inline"/>

After activating the Mono Features of the Cheat Engine, we will begin to search for where the C# Classes for Gameplay are stored. For games made with Unity and Mono, we can navitgate to the Assembly-CSharp class. Note for those of you that are new to this that the memory address will not be the same on your host if you are following along. This is due to the way that modern Operating Systems virtualize memory and randomize memory addresses. Security++

##### Mono Dissector:
<img src=".\assets\img\p1\assembly_csharp.PNG" alt="Assembly" class="inline"/>

Briefly clawing our way through the C# classes available to us, we come to the Class that is going to help us answer the Impossible Question, MathGameScript. Expanding this class provides us a handy list of fields and methods available to the MathGameScript class.

##### MathGameScript Class:
<img src=".\assets\img\p1\MathGameScript_Func.PNG" alt="MathGameScript" class="inline"/>

Notice the hex values next to each of the "fields" in the class, as this will be important later!

Now would be a good idea to ask ourselves some questions about why the "Impossible Question" can't answered:

* Is the jumbled text the problem?
* Is the text itself even relevant?
* Will the answer be wrong no matter what? 
* Will answering the question correctly break gameplay?
* If it does break, how can this benefit us?

Now that we have a few questions in the back of our mind, let's progress a bit in the game. Since we are trying to answer the Impossible Question we should take a good, thorough look at what a normal question looks like as an instantiated C# class.

Let's head to the first "Notebook", click on it, and begin the Math Game. Now head to the second "Notebook" and begin the second Math Quiz. The third question in this quiz _should_ be impossible to answer and look like a jumbled mess.

##### Notebook:
<img src=".\assets\img\p1\first_noteboo.PNG" alt="Notebook2" class="inline"/>

##### Math Game:
<img src=".\assets\img\p1\mathtime_1.PNG" alt="Mathtime1" class="inline"/>

A new MathGameScript class should have been instantiated once we started the Math quiz. We can right-click on the MathGameScript class in the "Mono Dissector" and "Find instances of this class". 

##### MathGameScript Class:
<img src=".\assets\img\p1\class_instances.PNG" alt="ClassInstances" class="inline"/>

We'll need to review the fields in each instance to determine if we're currently working out of that Class instance. 

For the above, we'll look for the following values:

* problem - 1
* num1 - 6
* num2 - 9
* solution - 15

Once we've identified the proper class instance, continue on to question three. You'll notice a difference between the first and third questions right away.

##### I Can't Even! (or Odd):
<img src=".\assets\img\p1\impossible_q.PNG" alt="NoWayJose" class="inline"/>

##### Current Instance (MathGameScript):
<img src=".\assets\img\p1\Instance.PNG" alt="Instance" class="inline"/>

There are important details in this screenshot that we need to address. We first ensure that we are on Problem 3 in the current instance by validating that problem field is set to 3.

Next, we will verify as much as possible the numbers in fields num1 - num3. Then, we need to record the solution so that we can enter it into the answer field. _HOWEVER_, even if we place the answer into the field in the classes current running state it will still give us an incorrect answer penalty and make Baldi very angry.

There is a Boolean that has changed from 0 to 1 on the third question that was not present in the previous two, the "impossibleMode" value.

Remember what I said about the hex values next to each field in the "Mono Dissector" class explorer being important? That's because these are the memory offsets! What does this mean? Well, it's going to point us to exactly the location in memory where this "impossibleMode" boolean is set, so we can turn it off.

First, we take the base address of the class instance, which is the Hex value from the "Instances of MathGameScript" class we are working out of. Then, we add the value of the offset to get the exact memory location we need to edit.

In my case, the base memory address of the current MathGameScript instance is 04C5F000. The offset of the impossibleMode field is A5. _So_:

04C5F000 + A5 = 4C5F0A5

Now we need to explore this region in memory!

From the main Cheat Engine window, select the "Add Address Manually" button. Once added, right click on the memory address and select the "Browse this memory region" option.

##### Adding the Class Base Address:
<img src=".\assets\img\p1\add_addr.PNG" alt="AddressAdd" class="inline"/>

##### Exploring Memory:
<img src=".\assets\img\p1\browse.PNG" alt="BrowseMemory" class="inline"/>

Once we start to browse, the debugger conveniently places the first item we can view at the base memory location of 04C5D000. We can use the keyboard to navigate across the memory addresses until we reach the offset memory address, 04C5F0A5, which conveniently appears to hold a 01 value (boolean) that we can switch to 00!

##### I see you, you sneaky flag:
<img src=".\assets\img\p1\found_it.PNG" alt="Foundit" class="inline"/>

Go ahead and set that value to zero. There is no save feature here since what we just did was edit the value in memory, in real time!

Open up the Class Instance window again "Instances of MathGameScript", and collapse the class (shrink it so it closes). Once that's done, reopen the Class (expand) and see if the impossibleMode boolean value has been sucessfully set to 0.

##### Impossible Mode <OFF>:
<img src=".\assets\img\p1\impossible_mode_off.PNG" alt="WeDidIt" class="inline"/>

Well, it looks like the impossibleMode boolean has been successfully changed in memory from 1 to 0, making the Impossible Question much less impossible and possibly much more possible! POSSIBLY!

Enter the solution from the solution field of the class into the answer block in-game and we'll see what happens.

##### WOW! YOU EXIST!:
<img src=".\assets\img\p1\i_did_it.PNG" alt="W00T" class="inline"/>

I can tell by the third green check mark, the celebration message buried deep in the problem text, and Baldis... happy... expression that we're off the hook!

Three notebooks later and Baldi still seems pretty happy with me. Maybe this wasn't a educational game for Math, but for Reverse Engineering?

##### 3Deepy4Me:
<img src=".\assets\img\p1\3_notebooks_deep.PNG" alt="BOO!" class="inline"/>

### The End? 

I'm going to keep attempting to break this game down and document a few interesting and quirky little hacks we can use on NPCs other than Baldi and gameplay in general, as this was actually a lot of fun to do. Cheat Engine will be my go-to unless I make a quick shift off to another debugger/decompiler combo!

I don't plan on making a trainer for this from the Cheat Engine software, not until I really dig deeper into the game code anyway.

I hope you enjoyed my writeup!

~ Marhtini
