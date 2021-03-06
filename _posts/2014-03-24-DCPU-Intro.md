---
layout: post
title: DCPU - Intro
categroy: posts
---

For this years major project season at [CSH](csh.rit.edu) I've decided
to build what I'm calling a "discrete" CPU. The processor is for now completely
virtual, later I will try to build it out of the same components it's design calls
for. The design goals of it are as follows:

1.  A word size of 8 bits or greater 
2.  Capable of the following calculations:
    * Addition
    * Subtraction
    * Multiplication
    * Division
3. Capable of addressing a words worth of physical memory
4. Arbitrarty amount of registers
5. Capable of executing programs from physical memory

More may be added to this list as time goes on, however I see this as
a fit list for my project. I'm not going to worry power and other physical
implementations, as the primary educational value of this is the discrete
mathematical and logical value. I'm currently taking a Discrete Mathematics
course, and thought it would be neat to employ and develop some of the skills
I learned through applied means.

The DCPU articles are living documents, they may change over time to reflect the
current state of the project. 

---
Development Tools
---
I'm using the following tools for developement:

* [Git](git-scm.com) - Git is a Source control management tool designed primarily for code development, however it can be used for any text or binary file to manage revisions and changes. The 'source' for this project is in XML format, and while I am not editing it directly, git will keep a catalogue of all the changes I have made and enable me to restore my work to arbitrary points in time.
* [Google Code](https://code.google.com/) - Supplementing my use of git, google code allows me to 'push' my code to a remote location for safe keeping. Besides backing up my files, it allows the source for the file to be open to the public, under the terms of the MIT license.
* [Logisim](http://ozark.hendrix.edu/~burch/logisim/docs/2.3.0/guide/subcirc/using.html) - Logisim is in the strictest sense a circuit design tool. It can however be used to simulate and test circuitry 'live' with a series of options and configurations. I use this as the primary design tool, and is what generates my circuitry files (.circ) that I consider to be the 'source' files.

---
Resources
===
I'm not building this out of the blue, I'm using various sources for this project:

* [Wikipedia](http://www.wikipedia.org/) - Specifically I've looked at many implementation of various components like adders and flip-flop types. I've used their designs for many of my components, and have intregated them with the overall design.
* [Wikibooks](http://www.wikibooks.org/) - In the same realm as wikipedia, wikibooks provides entire books dedicated to many of the topics I am working with. It has both the detail and abstraction that wikipedia often fails to provide.

---
Sample
===
Here's an excerpt from Logisim of the registers:

![Registers](/images/Registers.png)