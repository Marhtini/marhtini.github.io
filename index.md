## Breaking Baldi - Part 1 : The Impossible Question
<img src=".\assets\img\p1\breaking_baldi.png" alt="Baldi" class="inline"/>

Baldi's Basics is a self-described "state-of-the-art, fully 3D interactive, fun-time educational game that teaches a slew of subjects!" It's also quite creepy in the creepiest-of-pasta sort of way! 

Unfortunately for you, Baldi really doesn’t like it when you mess up. Particularly in “Everyone’s favorite subject, Math!”.  If you’ve ever played the game, you may believe that Math is the only subject on the table in Baldi’s world. However, I’m about to turn it into a fun and educational reverse engineering experience! ;) 

You quickly realize that things are not quite what they seem in Baldi’s educational horror show when one of the questions you’re required to answer suddenly becomes incomprehensible, a la Zalgo, and you are unable to appease Baldis extreme expectations. 
“Unfair!” I hear you shouting. How can we possible answer a question that is so jumbled that is clearly designed for us to fail?
Well, anything is possible through the magic of Reverse Engineering, memory manipulation, and debugging!

For this exercise, we will be using Cheat Engine. In this specific case, the use of Cheat Engine just made sense as I needed to have built in Mono Dissection tools (for Unity C#) and the ability to quickly manipulate instantiated C# classes in memory.

To begin to discover the answer to the Impossible Question, we need to attach the Cheat Engine software to the game executable (BALDI.EXE), and activate the available Mono Features.

Attaching to the game executable:
<img src=".\assets\img\p1\baldi_exe.png" alt="Baldi" class="inline"/>

Activating the features:
<img src=".\assets\img\p1\mono_dissect.png" alt="MonoDissect" class="inline"/>



