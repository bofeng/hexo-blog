---
title: '编程兔学Scratch'
date: 2015-12-30 05:32:22
tags: [scratch]
published: true
hideInList: false
feature: 
---
<div>
<link href='https://fonts.googleapis.com/css?family=Short+Stack' rel='stylesheet' type='text/css'>
<style>.handwriting {font-family: "Short Stack", cursive; font-size:16px}</style>
</div>

Today Tutu finished reading the book《Learn to Program with Scratch》, he wrote some notes:

<!-- more -->


<div class="handwriting">

<h3> Getting Started</h3>

<p>Stage size is 480 x 360.<p>

<p>You can copy blocks from one sprite to another, just drag blocks from the Sprite area of the source sprite to the thumbnail of the destination sprite in the Sprite list</p>

<p>Costumes: a sprite can "wear" only one at a time</p>

<p> Scratch can read only MP3 and WAV sound files</p>

<p> Backdrop usage example: if you are creating game, you might show one backdrop with instructions to begin and then switch to another when the user starts the game</p>

<p> The "Can drag" in player checkbox indicates whether or not the sprite can be dragged in Presentation mode</p>

<p> In "edit" menu, choose "Turbo mode" will increase the speed of some blocks</p>

<p> <mark>Block:</mark> [set x to [mouse x]] </p>

<p> <mark>Block:</mark> [if touching paddle then [pick random -30 to 30]] </p>

<p> <mark>Block:</mark> [if touching floor then [stop all]] </p>

<p>"Pick random" block allow you input a range, Scratch will only choose values b/t the two limits *inclusive* <p>

<h3>Motion and Drawing</h3>

<p>"Glide to" block like "go to", but it lets you set how long the Sprite to the target position.</p>

<p>Motion commands work with Reference to a sprite's center, so the center position is important.</p>

<p>Scratch pointing direction: 0 is up, 90 is right, 180 is down and -90 is left.</p>

<p>Direction for a sprite by default is 90 (right)</p>

<p>Each sprite has an invisible pen, which can be either up or down </p>

<p><mark>Block:</mark> [reset timer] [wait until [timer > 3]] </p>

<p> Scratch maintains a timer that records how much time has passed since Scratch was started. When you start Scratch in a Web browser, the timer will be set to 0, and it will count up by tenths of a second as long as you keep Scratch open. The timer block (in the Sensing palette) holds the current value of the timer. The checkbox next to the block allows you to show/hide the block’s monitor on the Stage. The reset timer block resets the timer to 0, and the time starts ticking up again immediately. The timer continues to run even when the project stops running. </p>

<p> A cloned sprite inherits the original’s state at the time it is cloned — that is, the original’s current position and direction, costume, visibility status, pen color, pen size, graphic effects, and so on.</p>

<h3>Looks and Sound</h3>

<p>The commands in the Looks palette will let you create animations and apply graphic effects like whirl, fisheye, ghost, and so on to costumes and backgrounds.</p>

<p>You can use the switch backdrop to command to change scenes in a story, switch levels in a game, and so on. Any sprite in your project can use the when backdrop switches to block to detect when the Stage has switched to a certain costume and act accordingly.</p>

<p> Change layer's z-index: "go to front" (always draw a sprite on top) and "go back layers".</p>

<p>You can add background music to your application by playing an audio file repeatedly. The easiest way to do this is to use play sound until done to let the file to play completely, and then restart it.</p>

<p>Tempo is measured in beats per minute (bpm). The higher the tempo, the faster the notes and drums will play.</p>

<h3>Procedures</h3>

<p>You can create your own custom blocks. After you make a custom block, it should appear in the More Blocks palette, where you can use it as you would any other Scratch block.</p>

<p> When you define customized block, there is an option called "Run without screen refresh", if you selected it, the blocks will run without pausing to refresh the screen, allowing the procedure to run much faster. The screen will refresh after Scratch executes the entire procedure.</p>

<p> You can have one procedure call another procedure (nested procedure).</p>

<p> In Scratch, different data type has different block shape, for example, numbers are rectangle with rounded corner, strings rectangle with shape corner.</p>

<p> Scratch automatically tries to convert b/t data types as needed.</p>

<p> When creating a variable, Choosing For this sprite only creates a variable that can be changed only by the sprite that owns it. Other sprites can still read and use the variable’s value, but they can’t write to it.</p>

<p> If scratch tries to convert a string to number, for example, "blahblah" to number, it will fail, however, rather than showing an error msg, Scratch will silently set the converted value to be 0.</p>

<p> <mark>Cloud Variable:</mark> Anyone who views a project you’ve shared on the Scratch website can read the cloud variables in the project. For example, if you share a game, you can use a cloud variable to track the highest score recorded among all the players. The score cloud variable should update almost immediately for everyone interacting with your game. Because these variables are stored on Scratch servers, they keep their value even if you exit your browser. Cloud variables make it easy to create surveys and other projects that store numbers over time. </p>

<p> <mark>Variables and Clone:</mark> Every sprite has a list of properties associated with it, including its current x-position, y-position, direction, and so on. You can imagine that list as a backpack holding the current values of the sprite’s attributes. When you create a variable for a sprite with a scope of For this sprite only, that variable gets added to the sprite’s backpack. When you clone a sprite, the clone inherits copies of the parent sprite’s attributes, including its variables. An inherited property starts out identical to the parent’s property at the time the clone is created. But after that, if the clone’s attributes and variables change, those changes don’t affect the parent. Subsequent changes to the parent sprite don’t affect the clone’s properties, either.</p>

<p> A variable's monitor can have different types: normal readout (default), large readout, and slider control</p>

<p> A variable’s monitor also indicates its scope. If a variable belongs to one sprite, its monitor should show the sprite name before the variable name. For example, the monitor Cat speed 0 indicates that speed belongs to Cat. If the speed variable were a global variable, its monitor would only say speed 0. </p>

<p> The "Light" sprite executes the "set ghost effect to" command to change its transparency level to simulate visual effect like a actual bulb.</p>

<p> <mark>Block:</mark> for user input, [ask and wait] </p>

<p>After executing the ask and wait command, the calling script waits for the user to press the ENTER key or click the check mark at the right side of the input box. When this happens, Scratch stores the user’s input in the answer block and continues execution at the command immediately after the ask and wait block.</p>

<p> <mark>Block:</mark> Use [join] block to concatenate strings.</p>

<h3> Making Decisions</h3>

<p> Check my color sensor game:</p>

<iframe allowtransparency="true" width="485" height="402" src="https://scratch.mit.edu/projects/embed/92807707/?autostart=false" frameborder="0" allowfullscreen></iframe>

https://scratch.mit.edu/projects/embed/92807707/?autostart=false

<p> <mark>Idea:</mark> Play Rock, Paper, Scissors game against the computer. The player makes a selection by clicking one of three buttons that represent rock, paper, or scissors. The computer makes a random selection. The winner is selected according to the following rules: Paper beats (wraps) rock, rock beats (breaks) scissors, and scissors beat (cut) paper. </p>

<h3> Repetition </h3>

<p> Procedures that can call themselves with recursion. </p>

<p> Use [stop] block to end active scripts: 1, stop this script: stop the script that invoked this block 2, stop all: stop all scripts in the applicaiton 3, stop other scripts in sprite: stop all scripts in a sprite except the one that invoked this block </p>

<h3> String Processing </h3>

<p><mark>Block:</mark> [letter [] of []], in Scratch, string's index starts from 1, not 0.</p>

<p><mark>Idea:</mark> Counts how many vowels are in an input string. </p>

<p>Scratch's string could be multi-byte character, like Chinese, "letter 1 of Chinese sentence" will show first Chinese character correctly.</p>

<p><mark>Idea:</mark> Check if a string is palindrome.</p>

<p>In Scratch, you can't alter the characters in a string, so the only way to change a string is to create a new one. </p>

<p><mark>Idea:</mark> Binary to decimal converter.</p>

<h3> Lists </h3>

<p>A list is like a container where you can store and access multiple values. You can think of it as a dresser with many drawers, with each drawer storing a single item. When you create a list, you name it just as you would a variable.</p>

<p>List's index starts from 1, not 0, like string </p>

<p>List supported functions: add, delete , insert, replace, contains, item at index, length </p>

<p> Trying to access an element past the boundaries of a list is, technically, an error. Rather than display an error message or abruptly terminate your program, however, Scratch silently tries to do something sensible with the offending block, for example, access out-of-range index, Scratch will return empty. Insert or delete element at out-of-range index, Scratch will ignore this command.</p>

<p><mark>Idea:</mark> Ask user to input number to a list, then find the min, max and average; search and sorting; find median.</p>

<p><mark>Idea:</mark> Math Wizard, US Map Quiz</p>



</div>
