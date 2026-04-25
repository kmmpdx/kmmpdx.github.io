---
layout: post
title: "RPi C++ Getting Started With SDL3"
author: mike
excerpt_separator: <!--more-->
---

The Simple DirectMedia Layer (SDL3) graphics library provides an easy way to add both simple and extremely complex graphics features to programs written in C++ on the Raspberry Pi and other Linux computers.  While the library is popular for 3D game design, in this series, we'll be mostly concerned with using the library to build control panels and GUI's for real-time data display, analysis and control.


In this article, we'll cover the basics of getting started with both the library and C++ on a Raspberry Pi 4/5 with a desktop display, by installing the library and creating a basic skeleton C++ program using a main loop to display an empty graphics window.  Other articles introduce the basics of 2D graphics and user input events.
<br/> <br/>
<img src="/assets/blogImages/2026-01-03/SDL3-MainBasics1.png" alt="empty SDL3 graphics window" width="500" />
<!--more-->
<br/> <br/> <br/>
*The usual disclaimer:  This is not expert or professional advice - it is provided for educational purposes only.  Always back up your files before installing anything.  Always understand why you're doing something.  Always understand what you're doing.  If this breaks something, it's your responsibility.*
<br/> <br/> <br/>
We'll start by coding a minimal skeleton C++ program that is a good starting point for writing applications using the <a href="https://wiki.libsdl.org/SDL3/FrontPage/" style="color: blue">SDL3 Library</a>.   This series of articles is mainly aimed at engineers and hobbyists who want to be able to add simple graphical interfaces to their programs - especially for real-time signal processing and control.

A single text file created in any plain-text editor will be used for the source code, and compilation of the program will be done from the command line - this is typically only useful for very small programs or testing, but it is an easy way to get started with C++ programming on the Raspberry Pi or most any Linux distribution.

Since the Raspberry Pi Operating System includes the GNU C/C++ compiler, and the repository has a recent release of the SDL3 library, getting started is quite easy.  A working knowledge of C/C++ is useful, but we'll attempt to make this program an easy step-by-step introduction to C++.




<br/>
### Update/Install the Library Packages from the Repository

First, install/update the apps and development libraries we'll need from the Raspberry Pi repository - on the command line enter:

<pre class="monoHighlight">
    sudo apt update
    sudo apt install libsdl3-dev
    sudo apt install build-essential
    sudo apt install geany
    sudo ldconfig
</pre>
<br/>


### About C++ Libraries

We'll be using dynamic libraries for this program, which are reusable pieces of pre-compiled executable code, installed on your computer, that your program looks for when it starts up and then can use as part of it's operation.  For example, the program will start by opening a graphics window on the screen - your program doesn't need to deal with the intricacies of handling the interface to the window manager/renderer - it just uses a piece of the SDL3 library that does it for you.

Library files typically have a name of the form libxxx-dev where the 'xxx'  will typically be the name of the library - for example <code>libsdl3-dev</code> above.  Dynamic libraries, when installed, include 2 files types - the 'header' file(s), which typically have a .h extension and the library executable file(s) which will have a .so extension.

The locations and names of library files can be located using the <code>dpkg</code> command - for example, on the command line type:
<pre class="monoHighlight">
    dpkg -L libsdl3-dev
</pre>
which will output the list of files that were installed with the library, along with where they were put.

For example, in the case of the <code>libsdl3-dev</code> library, the installer will put the .h files in the main header directory <code>/usr/include/SDL3</code> and the .so executable file was put in the <code>/usr/lib/x86_64-linux-gnu</code> directory on my desktop Debian machine, and under /usr/lib/aarch64-linux-gnu on the Raspberry Pi.

It's important to know where the files are located - in particular, the .h header file contains the definitions of functions and variables your program needs to interface with the library - sometimes the only documentation for a library will be included in the header file.

The other key command involved with installing libraries is the <code>ldconfig</code> command - when a new library is installed, it needs to be run to update the operating system paths used by the compiler so it will know where to find the library files when your program attempts to use them.
<br/><br/>



### Open a New C++ Source File

We'll just use a single text file created in any plain text editor for the source code, and compile the program from the BASH command line - this is typically only useful for very small programs or testing, but is an easy way to get started with C++ programming on the Raspberry Pi.

First, make a directory for the program, change to it, and use the geany text editor to open a new file to use for the C code:

<pre class="monoHighlight">
    mkdir ~/projectSDL3-mainbasics
    cd ~/ProjectSDL3-mainbasics
    geany SDL3-mainbasics.cpp
</pre>
<br/>



### Step-By-Step ...

If you're already familiar with writing and executing C++ programs, you can skip over the step-by-step instructions below and go to the end of the article for the complete skeleton program.  The SDL3 library is written in C, but works directlly with C++.  The code that follows is stylistically C, but we'll take advantage of several useful C++ features.
<br/><br/>



### Starting with 'Hello World'

Typically, when starting on a new system, the first program to be written and run is a simple 'Hello World' program which will test the file editing process, compiler command(s), linking to the library files and correct header files and file locations - in this case, we'll print out the version number of the SDL3 dynamic library.

Copy or type the code below into your SDL3-mainbasics.cpp file and save it (note, you need to always save the program after an edit - otherwise the compiler won't use your current edits).

The first 2 lines tell the compiler to include the headers of the standard C++ <code>iostream</code> input/output library and the <code>SDL3/SDL.h</code> library, which provide pre-defined definitions of the functions from the SDL3 library that we'll need in this program.

The <code>main()</code> function is the entry point for any stand-alone C++ program - it's where your program will start execution when launched from the command line.

The large multi-line statement after the main() declaration is the only instruction in this program - it uses the standard C++ console output <code>std::cout</code> to print out the version of the SDL3 dynamic library that is installed on your computer and being used by the program.  The library function <code>SDL_GetVersion()</code> returns a string with the entire version in it, and the functions <code>SDL_VERSIONNUM_MAJOR/MINOR/MICRO()</code> parse the string into seperate small sub-versions strings to output to the console so that we can insert a '.' between the major, minor and micro version numbers for readability.



<div class="shadowbox"><pre class="monoBackground">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;

int main() {                                                                     // program entry point
    
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                      // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;

}
</pre> </div> <br/>
To compile the program, bring up a new command line in the directory you created and enter :
<pre class="monoHighlight">
    g++ SDL3-mainbasics.cpp -oSDL3-mainbasics -lSDL3 -Wall
</pre>


This will start the GNU compiler - the source file you wrote your program into is the <code>SDL3-mainbasics.cpp</code> file, and the name following the <code>-o</code> will become the name of the executable file.  The <code>-lSDL3</code> tells the compiler to 'link' to the SDL3 library file.  And the <code>-Wall</code> tells the compiler to print out any warnings it might find.

If there are no errors or warnings, it will just return a command prompt, indicating all is well and that it has successfully compiled the program - if there is an error, it will print a message indicating the problem that you'll have to fix - typically a misspelling or a missing '{}' or ';' ...

You can now run the program by entering:

<pre class="monoHighlight">
    ./SDL3-mainbasics
</pre>


The program will print out the library version number on the console, and then close:

<pre class="monoHighlight">
    SDL3 Library Version 3.2.10
</pre>

It doesn't look very exciting, but it indicates a lot of things have happened correctly - if you'd like to see the vast amount of detail that occurs when running the compiler, add the verbose flag -v to the end of the g++ command:

<pre class="monoHighlight">
    g++ SDL3-mainbasics.cpp -oSDL3-mainbasics -lSDL3 -Wall -v
</pre>





<br/>
### Bring Up a Graphics Window

Now that you've got a bare-bones program compiling and running, let's bring up a graphics window - add the code below in blue to your file:

<div class="shadowbox"><pre class="monoBackground">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;

int main() {                                                                      // program entry point
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                       // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;
</pre><pre class="monoHighlight">
    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;        // Window and Renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                              // Initialize the library
        !SDL_CreateWindowAndRenderer("SDL3 Main Loop Basics",                     // Create Window with Renderer
                                    1000, 400, 0, &window, &renderer)) {
        std::cout << SDL_GetError() << std::endl;
        return 1;                                                                 // program exit
    }

    SDL_RenderClear(renderer);                                                    // clear window
    SDL_RenderPresent(renderer);                                                  // updates the screen
</pre><pre class="monoBackground">
    return 0;                                                                     // program exit
}
</pre> </div> <br/>
First, two pointer variables of SDL3 library-defined types are declared:  <code>window</code> and <code>renderer</code>, which will be used to keep track of the window and renderer that will be created, so they can be referred to later in the program.

The program then calls the library initialization function <code>SDL_Init()</code>, which, if all goes well, returns a boolean value of true, which allows the program to continue, otherwise an error will be fetched from the library and printed, and the program will exit.

The two pointer variables that were defined are then passed into the <code>SDL_CreateWindowAndRenderer()</code> function along with parameters to define our window as 1000 pixels wide by 400 pixels high - again, if all goes well, it returns a boolean value of true, allowing the program to continue.  The '0' parameter in the function is for additional <a href="https://wiki.libsdl.org/SDL3/SDL_WindowFlags" style="color: blue">WindowFlags</a> - we're just using the default here.

The next two function calls to the library <code>SDL_RenderClear()</code> and <code>SDL_RenderPresent()</code> clear the new window and draws it to the screen.

Save, compile and run your program - you should see a window that appears in the middle of your display and then disappears immediately as your program prints out the version line to the console, and then ends, as before.






<br/>
### Make It Stay - a Main Loop

To prevent the program from continuing on to the end of the program after showing the window, we'll add an "infinite loop" - often called a "Main Loop" in programming parlance:

<div class="shadowbox"><pre class="monoBackground">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;

int main() {                                                                    // program entry point
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                     // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;

    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;      // Window and Renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                            // Initialize the library
        !SDL_CreateWindowAndRenderer("SDL3 Main Loop Basics",                   // Create Window with Renderer
                                    1000, 400, 0, &window, &renderer)) {
        std::cout << SDL_GetError() << std::endl;
        return 1;                                                               // program exit
    }

    SDL_RenderClear(renderer);                                                  // clear window
    SDL_RenderPresent(renderer);                                                // updates the screen
</pre><pre class="monoHighlight">
    while (true) {
        ;
    }
</pre><pre class="monoBackground">
    return 0;                                                                   // program exit
}
</pre> </div> <br/>
Once program execution reaches the <code>while (true)</code> statement, there is nothing to stop it from executing forever, so from the users perspective, the program will 'freeze', executing the same infinite loop, over and over - which means the window we drew will stay up on the screen 'forever' - unfortunately, our program is 'hung' and not doing anything.

Go ahead and edit, save, compile and run the program - this time, the window frame will be drawn, and stay, and your command line will not return a prompt to indicate that the program ended.

Take a look at the window - it looks normal and can be moved around - but if you minimize it, and then re-expand, it will become see through, and you'll see whatever is 'underneath' the frame of the window - which will then move with the window.

<br/>
<img src="/assets/blogImages/2026-01-03/SDL3-MainBasics1.png" alt="empty SDL3 graphics window" width="500" />
<br/> <br/>


After admiring your not-very-functional window, you can shut down the program by either:
- clicking on the 'X' close button of the window and waiting a couple of seconds, and then clicking on 'yes' when the operating system notices your program isn't responding and asks if you'd like it ended, or
- clicking on your command window that launched the program to regain focus, then hitting Ctrl Z to suspend the process, then typing "jobs -l" to get a list of PIDs that are stopped, then typing "kill -9 xxxxx" where xxxxx is the PID of your stuck program as reported by "jobs", or
- clicking on your command window that launched the program to regain focus, then hitting Ctrl Z to suspend the process, then typing "killall -9 SDL3-mainbasics".





<br/>
### Make It Stop


Let's modify our Main Loop so our program can start to be useful - beginning with having the program deliberately stop when the 'X' button on the window is clicked - change the 'while loop' code to:

<div class="shadowbox"><pre class="monoBackground">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;

int main() {                                                                    // program entry point
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                     // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;

    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;      // Window and Renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                            // Initialize the library
        !SDL_CreateWindowAndRenderer("SDL3 Main Loop Basics",                   // Create Window with Renderer
                                    1000, 400, 0, &window, &renderer)) {
        std::cout << SDL_GetError() << std::endl;
        return 1;                                                               // program exit
    }

    SDL_RenderClear(renderer);                                                  // clear window
    SDL_RenderPresent(renderer);                                                // updates the screen

</pre><pre class="monoHighlight">
    bool endProgram = false;
    while (!endProgram) {                                                       // Main Loop
        SDL_Event event;
        if (SDL_PollEvent(&event) && (event.type == SDL_EVENT_QUIT))            // window was closed
            endProgram = true;
    }                                                                           // end of Main Loop

    SDL_DestroyRenderer(renderer);          SDL_DestroyWindow(window);          // Shut down the library
    SDL_Quit();
</pre><pre class="monoBackground">
    return 0;                                                                   // program exit
}
</pre> </div> <br/>
This version of the loop puts a boolean variable <code>endProgram</code> in control of the Main Loop, and uses a library function <code>SDL_PollEvent()</code> to test for user actions.  The function will return true if the user has taken some action in/on our window, and will fill in appropriate values into the library defined <code>SDL_Event</code> structure <code>event</code> that was passed into the function.

In this case, if the window's close button is clicked, the <code>event.type</code> element of the structure will be set equal to the library defined value of <code>SDL_EVENT_QUIT</code> and when we see it, we can set the <code>endProgram</code> boolean variable to true, which will negate the while loop, allowing the code to fall out of the Main Loop and end the program cleanly.

Since we're now ending the program by choice, there's a few things we should clean up before closing.  After detecting that the user wants to end the program and falling out of the Main Loop, the following code uses the library functions <code>SDL_DestroyRenderer()</code> and <code>SDL_DestroyWindow()</code> to free up memory and resources, and then closes down the interface to the library and graphics system with <code>SDL_Quit()</code>:


Again, save, compile and run your program:
<pre class="monoHighlight">
    g++ SDL3-mainbasics.cpp -oSDL3-mainbasics -lSDL3 -Wall
    ./SDL3-mainbasics
</pre>
Now, when you click the window's close button, the program will shut down the window and exit cleanly to the command line.
<br/><br/>




### Clear the Window

The window still won't behave correctly with minimizing - the SDL library programing model assumes an infinite running Main Loop that starts by clearing a working memory 'canvas' that is cleared to a background color at the start of the loop, is then changed and populated by drawing whatever image/graphics/etc that we need in our application, which is then transferred to the display area on the screen once the drawing is complete - which is repeated as long as the program is still operating.

So far we've created the stable window frame that can be closed - next step is to clear it's background and transfer it to the display screen - if we simply move the <code>SDL_RenderClear()</code> and <code>SDL_RenderPresent()</code> function calls inside the Main Loop, they will be done continuously, instead of only once:

<div class="shadowbox"><pre class="monoBackground">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;

int main() {                                                                    // program entry point
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                     // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;

    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;      // Window and Renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                            // Initialize the library
        !SDL_CreateWindowAndRenderer("SDL3 Main Loop Basics",                   // Create Window with Renderer
                                    1000, 400, 0, &window, &renderer)) {
        std::cout << SDL_GetError() << std::endl;
        return 1;                                                               // program exit
    }

    bool endProgram = false;
    while (!endProgram) {                                                       // Main Loop
</pre><pre class="monoHighlight">
        SDL_SetRenderDrawColorFloat(renderer, 0.f, 0.f, 0.f, 1.f);              // black
        SDL_RenderClear(renderer);                                              // clear window
        SDL_RenderPresent(renderer);                                            // updates the screen
</pre><pre class="monoBackground">
        SDL_Event event;
        if (SDL_PollEvent(&event) && (event.type == SDL_EVENT_QUIT))            // window was closed
            endProgram = true;
    }                                                                           // end of Main Loop

    SDL_DestroyRenderer(renderer);          SDL_DestroyWindow(window);          // Shut down the library
    SDL_Quit();

    return 0;                                                                   // program exit
}
</pre> </div> <br/>
In the code above, we also added another function call to set the window's drawing color to black with <code>SDL_SetRenderColorFloat()</code>, which is used by the <code>SDL_RenderClear()</code> function - the default drawing color is black, so it will work in this case, but once you start changing the color to draw other things, it will need to be set.

There are several ways of selecting and defining colors in the SDL library, but in this case, we're using the floating point RGBA format (Red-Green-Blue-Alpha) default type, where color is specified by combining three primary colors at different intensities (red, green and blue - RGB), along with a transparency value (Alpha - A) that defines how clear or opaque the color is.  The library function <code>SDL_SetRenderDrawColorFloat()</code> specifies the level of each of the 4 RGBA values, in order, with each having a range of 0.0 to 1.0 in floating point, where 0.0 is off, and 1.0 is fully on.  In the case of transparency (Alpha), 0.0 is fully transparent, while 1.0 is opaque.

With Alpha set to opaque, setting the RGB values to 0.0/0.0/0.0 gives black, while setting them to 1.0/1.0/1.0 gives white.  A useful tool for specifying other colors is a <a href="https://www.rgbcolorpicker.com/" style="color: blue">'color picker'</a>, which allows you to graphically 'pick' a color off a reference colored surface, reading out the RGB values for the color.  Note that many color pickers will provide an 8-bit value for each of the colors in the range of 0-255 (0x00-0xFF) - if so, simply divide the value by 255 to get a floating point result in the range of 0.0 - 1.0.

Once you save, compile and run the program, you'll have a functional window with a black background that will behave correctly when moved around, minimized and closed.
<br/><br/>



### Add Another Loop

To generalize the skeleton program a bit more, instead of the single <code> if </code> statement we used above to check for the 'window close' event from the library, the code below adds an Event Loop, which embeds the test for the <code>SDL_EVENT_QUIT</code> in a while loop that calls the <code>SDL_PollEvent()</code> function repeatedly until it indicates that no other events are available by returning false.

The reason for the change is that the library keeps track of all the user's actions for your program and puts them in a queue, where they stay until you access them, one at a time, with the <code>SDL_PollEvent()</code> function.  The loop allows all the events to be purged from the queue, while only looking for the ones (or in this case, one) your program needs to use - the rest are ignored.


<div class="shadowbox"><pre class="monoBackground">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;

int main() {                                                                    // program entry point
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                     // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;

    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;      // Window and Renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                            // Initialize the library
        !SDL_CreateWindowAndRenderer("SDL3 Main Loop Basics",                   // Create Window with Renderer
                                    1000, 400, 0, &window, &renderer)) {
        std::cout << SDL_GetError() << std::endl;
        return 1;                                                               // program exit
    }

    bool endProgram = false;
    while (!endProgram) {                                                       // Main Loop
        SDL_SetRenderDrawColorFloat(renderer, 0.f, 0.f, 0.f, 1.f);              // black
        SDL_RenderClear(renderer);                                              // clear window
        SDL_RenderPresent(renderer);                                            // updates the screen

        SDL_Event event;
</pre><pre class="monoHighlight">
        while (SDL_PollEvent(&event) != 0) {                                    // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                   // window was closed
                endProgram = true;
        }                                                                       // end of Event Loop
</pre><pre class="monoBackground">
    }                                                                           // end of Main Loop

    SDL_DestroyRenderer(renderer);          SDL_DestroyWindow(window);          // Shut down the library
    SDL_Quit();

    return 0;                                                                   // program exit
}
</pre> </div> <br/>


### About Time

The main loop calls the drawing routines as fast as the program can run, which is actually very fast - that speed is seldom required for a user-interface and uses excess CPU resources that can be better spent on your own processing or used by other programs running on your computer - the following changes to the program will help you characterize your programs timing - add the variable declarations above the Main Loop and the block of code at the top of the Main Loop:

<div class="shadowbox"><pre class="monoBackground">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;

int main() {                                                                    // program entry point
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                     // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;

    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;      // Window and Renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                            // Initialize the library
        !SDL_CreateWindowAndRenderer("SDL3 Main Loop Basics",                   // Create Window with Renderer
                                    1000, 400, 0, &window, &renderer)) {
        std::cout << SDL_GetError() << std::endl;
        return 1;                                                               // program exit
    }

    bool endProgram = false;
</pre><pre class="monoHighlight">
    uint64_t time=0, zTime=0, dTime=0, loopCntr=1;
</pre><pre class="monoBackground">
    while (!endProgram) {                                                       // Main Loop
</pre><pre class="monoHighlight">
        if ((dTime=((time=SDL_GetTicksNS())-zTime)) > 1E9) {                    // more than one second
            std::cout << "\rLoop Time(mS) = " << 1E-6*dTime/loopCntr
                                  << "      " << std::flush;
            zTime = time;
            loopCntr = 0;
        }
        ++loopCntr;
</pre><pre class="monoBackground">
        SDL_SetRenderDrawColorFloat(renderer, 0.f, 0.f, 0.f, 1.f);              // black
        SDL_RenderClear(renderer);                                              // clear window
        SDL_RenderPresent(renderer);                                            // updates the screen

        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                    // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                   // window was closed
                endProgram = true;
        }                                                                       // end of Event Loop
    }                                                                           // end of Main Loop

    SDL_DestroyRenderer(renderer);          SDL_DestroyWindow(window);          // Shut down the library
    SDL_Quit();

    return 0;                                                                   // program exit
}
</pre> </div> <br/>


Tracking time in programs requires access to a high-speed and high-accuracy counter maintained by the CPU hardware - the SDL library provides two easy to use functions that will provide your program with the number of counter counts ('ticks') since the program started, in either milliseconds (mS) or nanoseconds (nS).  Similar functions are available in the C++ standard libraries, but are syntactically a bit more difficult to use.

In the code above, each pass through the Main Loop, the <code>dTime</code> variable is calculated as the difference between the previous stored tick count in <code>zTime</code> and the current running nanosecond tick counter from the library code>SDL_GetTicksNS()</code> function.

When the program has executed enough cycles through the Main Loop such that the elapsed count exceeds one second, it's divided by the number of Main Loop passes it took (<code>loopCntr</code>), which gives the average number of nanoseconds it takes to execute the Main Loop code.  The value is scaled by a million to give a fractional loop time in milliseconds and output to the console.

Now, when you compile and run the program, the code will print out the approximate amount of time, in milliseconds (mS), it takes to make a pass through the Main Loop code - and in this case, since the program is not yet doing any drawing operations other than clearing the window, it's fairly fast - typically about 450 microseconds (0.45mS) on a Pi5.
<br/><br/>

### Next Steps

Next up - we'll begin with the skeleton program below and start using the library's drawing primitives in <a href="{% post_url 2026-01-04-Rpi-C++-SDL3-Drawing-Primitives %}" style="color: blue">C++ SDL3 2D Drawing Primitives</a>, show how to handle user interaction with your program in <a href="{% post_url 2026-01-05-Rpi-C++-SDL3-Event-Basics %}" style="color: blue">C++ SDL3 Event Basics</a>, and look into getting started with C++ Objects and TrueType text fonts in <a href="{% post_url 2026-01-06-RPi-C++-SDL3-Text-And-Objects %}" style="color: blue">C++ Text and Objects With SDL3</a>.
<br/><br/>




### SDL3 Main Loop Skeleton

The code below is a completed version of the above step-by-step code, with the addition of a Main Loop thread sleep function call <code>SDL_Delay()</code>, the output of an additional carriage return output to the command line to move the prompt back to the left, and addition of the traditional argc and argv arguments in the main function declaration that can be used to access command line parameters.

The sleep function was included because for many programs, an update rate of 20 times-per-second (50mS) is plenty fast enough to feel responsive - letting the program release it's thread back to the operating system for 50mS provides processing time that can be used by other threads/programs.  Note that the accuracy of the sleep function is determined by the operating system, and may not be particularly accurate or consistent - it's just guaranteed to be at least the specified time.
<br/>



<div class="shadowbox"><pre class="monoBackground">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;
// compile with:   g++ SDL3-mainbasics.cpp -oSDL3-mainbasics -lSDL3 -Wall

int main(int argc, char *argv[]) {                                              // program entry point
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                     // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;

    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;      // Window and Renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                            // Initialize the library
        !SDL_CreateWindowAndRenderer("SDL3 Main Loop Basics",                   // Create Window with Renderer
                                    1000, 400, 0, &window, &renderer)) {
        std::cout << SDL_GetError() << std::endl;
        return 1;                                                               // program exit
    }

    bool endProgram = false;
    uint64_t time=0, zTime=0, dTime=0, loopCntr=1;

    while (!endProgram) {                                                       // Main Loop
        SDL_Delay(50);                                                          // sleep for 50mS
        
        if ((dTime=((time=SDL_GetTicksNS())-zTime)) > 1E9) {                    // more than one second
            std::cout << "\rLoop Time(mS) = " << 1E-6*dTime/loopCntr
                                  << "      " << std::flush;
            zTime = time;
            loopCntr = 0;
        }
        ++loopCntr;

        SDL_SetRenderDrawColorFloat(renderer, 0.f, 0.f, 0.f, 1.f);              // black
        SDL_RenderClear(renderer);                                              // clear window

        // window drawing goes here

        SDL_RenderPresent(renderer);                                            // updates the screen

        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                    // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                   // window was closed
                endProgram = true;
        }                                                                       // end of Event Loop
    }                                                                           // end of Main Loop

    SDL_DestroyRenderer(renderer);          SDL_DestroyWindow(window);          // Shut down the library
    SDL_Quit();
    std::cout << std::endl;                                                     // extra CR/LF

    return 0;                                                                   // program exit
}
</pre> </div> <br/>





