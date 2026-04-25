---
layout: post
title: "RPi C++ Event Handling With SDL3"
author: mike
excerpt_separator: <!--more-->
---

The first part of this series on the Simple DirectMedia Layer (SDL3) covered getting started by loading the library and building a basic C++ program to display an empty graphics window.  In this part, we'll continue by adding basic event handling to the window using the libraries event management functions for mouse and keyboard actions.

<br/>
<img src="/assets/blogImages/2026-01-05/SDL3-MainEvents1.png" alt="command line window showing user outputs" width="500" />
<!--more-->
<br/> <br/> <br/>
*The usual disclaimer:  This is not expert or professional advice - it is provided for educational purposes only.  Always back up your files before installing anything.  Always understand why you're doing something.  Always understand what you're doing.  If this breaks something, it's your responsibility.*
<br/><br/>



As mentioned in the <a href="{% post_url 2026-01-03-Rpi-C++-SDL3-Getting-Started %}" style="color: blue">Getting Started</a> article, the library keeps track of the user's actions for your program and puts them in a queue to be made available to your program via the function call <code>SDL_PollEvent()</code>.  If any events have occurred, the function returns 'true' along with the event from the queue via the <code>SDL_Event</code> pointer <code>event</code> that was passed in with function call.  Note that the event 'while loop' will go through all pending events that need to be handled until the <code>SDL_PollEvent()</code> function returns 'false'.

The event pointer can take on many forms, depending on the type of event that has occurred - in this article, we'll only look at several of the events you're likely to need when building programs for user interfaces, but the library has many hundreds of different event types, sorted into several dozen categories - see <a href="https://wiki.libsdl.org/SDL3/SDL_Event" style="color: blue">SDL3 Library Events</a> and <a href="https://wiki.libsdl.org/SDL3/SDL_EventType" style="color: blue">SDL3 Event Types</a>.
<br/><br/>




### Open a New C++ Source File

As in the <a href="{% post_url 2026-01-03-Rpi-C++-SDL3-Getting-Started %}" style="color: blue">RPi C++ SDL3 Getting Started</a> article, we'll just use a single text file created in any plain text editor for the source code, and compile the program from the BASH command line - this is typically only useful for very small programs or testing, but is an easy way to get started with C++ programming on the Raspberry Pi.

Assuming you've made a directory to hold your test program from the Getting Started article, change to it and use the command line to create a new file in the geany text editor to use for these 2D Primitives code examples:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    cd ~/ProjectSDL3-mainbasics
    geany SDL3-eventBasics.cpp
</pre>
<br/>


### Starting With a Mouse Click

The following code starts with the final Main Loop skeleton program from the <a href="{% post_url 2026-01-03-Rpi-C++-SDL3-Getting-Started %}" style="color: blue">RPi C++ SDL3 Getting Started</a> article, and adds the changes in blue.  The window title is changed in the <code>SDL_CreateWindowAndRederer()</code> function call, along with adding the <code>SDL_WINDOW_RESIZABLE</code> flag to make the window resizable.

Down in the Event Loop in the Main Loop, a new <code>else</code> clause is added to test for a mouse click event - <code>SDL_EVENT_MOUSE_BUTTON_DOWN</code>, which if it is detected, prints out a click message to the console with the mouse button number and position in window coordinates.





<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;
// compile with:   g++ SDL3-eventBasics.cpp -oSDL3-eventBasics -lSDL3 -Wall

int main(int argc, char *argv[]) {                                              // program entry point
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                     // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;

    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;      // Window and Renderer
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                            // Initialize the library
        !SDL_CreateWindowAndRenderer("SDL3 Event Basics",                       // Create Window with Renderer
                                    1000, 400, 0, &window, &renderer)) {
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
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
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
            else if (event.type == SDL_EVENT_MOUSE_BUTTON_DOWN) {               // mouse button pressed
                std::cout << "Mouse Click button= " <<
                             (int) event.button.button <<                       // 0=left,1=center,2=right
                             " at x,y= " << event.button.x << "," <<            // x window position
                             event.button.y << std::endl;                       // y window position
            }
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        }                                                                       // end of Event Loop
    }                                                                           // end of Main Loop

    SDL_DestroyRenderer(renderer);          SDL_DestroyWindow(window);          // Shut down the library
    SDL_Quit();
    std::cout << std::endl;                                                     // extra CR/LF

    return 0;                                                                   // program exit
}
</pre> </div><br/>




Each of the <a href="https://wiki.libsdl.org/SDL3/SDL_Event" style="color: blue">event types</a> that is returned by the poll event function has an associated union structure containing any relevant data that is associated with the event - for example, when the <code>event.type</code> has the value of <code>SDL_EVENT_MOUSE_BUTTON_DOWN</code> as above, the relevent information for that event is carried in the structure <a href="https://wiki.libsdl.org/SDL3/SDL_MouseButtonEvent" style="color: blue">SDL3_MouseButtonEvent</a>, as shown below:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    typedef struct SDL_MouseButtonEvent {
        SDL_EventType type;                    /**< SDL_EVENT_MOUSE_BUTTON_DOWN or SDL_EVENT_MOUSE_BUTTON_UP */
        Uint32 reserved; Uint64 timestamp;     /**< In nanoseconds, populated using SDL_GetTicksNS() */
        SDL_WindowID windowID;                 /**< The window with mouse focus, if any */
        SDL_MouseID which;                     /**< The mouse instance id in relative mode, SDL_TOUCH_MOUSEID for touch events, or 0 */
        Uint8 button;                          /**< The mouse button index */
        bool down;                             /**< true if the button is pressed */
        Uint8 clicks;                          /**< 1 for single-click, 2 for double-click, etc. */
        Uint8 padding;
        float x;                               /**< X coordinate, relative to window */
        float y;                               /**< Y coordinate, relative to window */
    } SDL_MouseButtonEvent;
</pre>

Each element of the above structure can be accessed by appending the element name to the <code>event</code> return value, along with the union event name that's defined for the return type - in this case, <code>event.button.*</code>, where  <code>button</code> is the union event name for the returned type and  <code>*</code> is one of the elements in the structure shown above.


In this case, when one of the user's mouse buttons is clicked and the library returns the <code>SDL_EVENT_MOUSE_BUTTON_DOWN</code> event type, the mouse button number <code>event.button.button</code> and position <code>event.button.x</code>, <code>event.button.y</code> can be accessed and output to the console.

The available return types and their associated union event names and structures are documented here in the <a href="https://wiki.libsdl.org/SDL3/SDL_Event" style="color: blue">SDL Wiki</a> down under Event and Union Relationships.


Copy the code into your editor, save the file, and compile from the command line with the new filename, and run:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    g++ SDL3-eventBasics.cpp -oSDL3-eventBasics -lSDL3 -Wall
    ./SDL3-eventBasics
</pre>

When you click in the programs window area with the left, middle and right mouse buttons, you'll get the mouse button number and position that the mouse was at when the button was pressed in the output on your console:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    mike@debian:~/projectSDL3-mainbasics$ ./SDL3-eventBasics
    SDL3 Library Version 3.2.10
    Loop Time(mS) = 50.8075      Mouse Click button= 1 at x,y= 241.084,147.79
    Loop Time(mS) = 50.8096      Mouse Click button= 2 at x,y= 356.26,157.543
    Loop Time(mS) = 50.8584      Mouse Click button= 3 at x,y= 741.535,159.105
    Loop Time(mS) = 50.8824 
</pre><br/>

 

### Mouse Motion

As with clicks, the library generates events to reflect mouse movement.  As the user moves the mouse through the window, the library will generate events as the mouse changes position - if movement stops, or if the mouse is moved outside of the window, the events also stop.  Add the following additional else clause to the end of the <code>event.type</code> comparisons to print out the position information on the console:

<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                   // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                  // window was closed
                endProgram = true;
            else if (event.type == SDL_EVENT_MOUSE_BUTTON_DOWN) {              // mouse button pressed
                std::cout << "Mouse Click button= " <<
                             (int) event.button.button <<                      // 0=left,1=center,2=right
                             " at x,y= " << event.button.x << "," <<           // x window position
                             event.button.y << std::endl;                      // y window position
            }
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
            else if (event.type == SDL_EVENT_MOUSE_MOTION) {                   // mouse moving in window
                std::cout << "Mouse Move at x,y= " <<
                             event.motion.x << "," <<                          // x window position
                             event.motion.y << "  button= " <<                 // y window position
                             (int) event.motion.state << std::endl;            // 1=left,2=center,4=right (0=none)
            }
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        }                                                                      // end of Event Loop
    }                                                                          // end of Main Loop
</pre> </div><br/>

In the above code, the structure returned with the SDL_EVENT_MOUSE_MOTION event type is accessed via <code>event.motion.*</code> - in this case we're using the x,y position in <code>event.motion.x</code>, <code>event.motion.y</code>, and the mouse button state during movement in <code>event.motion.which</code>.  The button information is bit-mapped, where the least-significant three bits in the variable indicate if one or more of the three mouse buttons is being held down while being moved.  See <a href="https://wiki.libsdl.org/SDL3/SDL_MouseMotionEvent" style="color: blue">SDL3_MouseMotionEvent</a> for the associated structure definitions for the event type.

After compiling and running the program, as the mouse is moved within the window boundary, your program will print out:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    mike@debian:~/projectSDL3-mainbasics$ ./SDL3-eventBasicsSDL3 Library Version 3.2.10
    Loop Time(mS) = 50.8241      Mouse Move at x,y= 661,0  button= 0
    Mouse Move at x,y= 661.257,0.705383  button= 0
    Mouse Move at x,y= 660.257,1.70538  button= 0
    Mouse Move at x,y= 660.257,3.70538  button= 0
    Mouse Move at x,y= 660.257,4.70538  button= 0
</pre>
Note that the button number is useful in this case if you're doing drag control - moving the mouse with a button down.
<br/><br/>
 


### Mouse Wheel

As with button clicks and motion, the library also generates events to reflect rotation of the mouse wheel.  As the user rotates the mouse wheel while in the window, the library will generate events.  Add the following additional else clause to the end of the <code>event.type</code> comparisons to print out the wheel rotation event on the console:
<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                   // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                  // window was closed
                endProgram = true;
            else if (event.type == SDL_EVENT_MOUSE_BUTTON_DOWN) {              // mouse button pressed
                std::cout << "Mouse Click button= " <<
                             (int) event.button.button <<                      // 0=left,1=center,2=right
                             " at x,y= " << event.button.x << "," <<           // x window position
                             event.button.y << std::endl;                      // y window position
            }
            else if (event.type == SDL_EVENT_MOUSE_MOTION) {                   // mouse moving in window
                std::cout << "Mouse Move at x,y= " <<
                             event.motion.x << "," <<                          // x window position
                             event.motion.y << "  button= " <<                 // y window position
                             (int) event.motion.state << std::endl;            // 1=left,2=center,4=right (0=none)
            }
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
            else if (event.type == SDL_EVENT_MOUSE_WHEEL) {                    // mouse wheel
                std::cout << "Mouse wheel= " << event.wheel.y <<               // +-ticks of wheel 
                             "  at x,y= " << event.wheel.mouse_x << "," <<     // x window position
                             event.wheel.mouse_y << std::endl;                 // y window position
            }
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        }                                                                      // end of Event Loop
    }                                                                          // end of Main Loop
</pre> </div><br/>

The structure returned with the <code>SDL_EVENT_MOUSE_WHEEL</code> event type is accessed via <code>event.wheel.*</code>.  In this case, we're only printing out the 'vertical' wheel rotation that most desktop style mouses have (double axis wheels can also send 'horizontal' wheel rotation).  In addition, the position of the mouse when the wheel is rotated can be accessed via the structure - see the <a href="https://wiki.libsdl.org/SDL3/SDL_MouseWheelEvent" style="color: blue">SDL3_MouseWheelEvent</a> in the SDL3 library wiki entry.

Again, after compiling and running the program, as the mouse wheel is rotated, your program will print out which direction the wheel is turning - a -1 is printed if you rotate the wheel toward you while a +1 is output for rotation away:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    Loop Time(mS) = 50.8789      Mouse wheel= -1  at x,y= 647.359,147.43
    Loop Time(mS) = 50.622      Mouse wheel= 1  at x,y= 647.359,147.43
    Loop Time(mS) = 50.8095
</pre><br/>

 

### Keyboard Input


Basic keyboard input is straightforward - an event is generated when a key is pressed, and another event is generated when the key is released.  Beyond that, dealing with keyboard interfaces can rapidly become quite complex due to different key-to-character mapping for different languages and use of the shift, control and alt keys.  In this case, we'll just look at what happens when a key is pressed - add the highlighted else clause to the end of the event loop:
<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                   // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                  // window was closed
                endProgram = true;
            else if (event.type == SDL_EVENT_MOUSE_BUTTON_DOWN) {              // mouse button pressed
                std::cout << "Mouse Click button= " <<
                             (int) event.button.button <<                      // 0=left,1=center,2=right
                             " at x,y= " << event.button.x << "," <<           // x window position
                             event.button.y << std::endl;                      // y window position
            }
            else if (event.type == SDL_EVENT_MOUSE_MOTION) {                   // mouse moving in window
                std::cout << "Mouse Move at x,y= " <<
                             event.motion.x << "," <<                          // x window position
                             event.motion.y << "  button= " <<                 // y window position
                             (int) event.motion.state << std::endl;            // 1=left,2=center,4=right (0=none)
            }
            else if (event.type == SDL_EVENT_MOUSE_WHEEL) {                    // mouse wheel
                std::cout << "Mouse wheel= " << event.wheel.y <<               // +-ticks of wheel 
                             "  at x,y= " << event.wheel.mouse_x << "," <<     // x window position
                             event.wheel.mouse_y << std::endl;                 // y window position
            }
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
            else if (event.type == SDL_EVENT_KEY_DOWN) {                       // keyboard key pressed
                SDL_Scancode code = event.key.scancode;
                SDL_Keycode key = event.key.key;
                SDL_Keymod mod = event.key.mod;
                SDL_Keycode mkey = SDL_GetKeyFromScancode(code, mod, false);
                std::cout << std::hex <<                                       // output to hex format
                          "Keyboard: Scancode=0x" << code <<                   // key scancode (hex)
                          " Key=" << (char)key << " (0x" << key << ")" <<      // key code in char(hex)
                          " Modifier=0x" << mod <<                             // in hex - ie shift, ctrl, alt
                          " ModKey=" << (char)mkey << " (0x" << mkey << ")" << // modified key in char (hex)
                          std::dec << std::endl;          
            }
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        }                                                                      // end of Event Loop
    }                                                                          // end of Main Loop
</pre> </div><br/>


The data returned with the SDL_EVENT_KEY_DOWN event is accessed via <code>event.key.*</code>, and is documented in the SDL3 library wiki:  <a href="https://wiki.libsdl.org/SDL3/SDL_KeyboardEvent" style="color: blue">SDL3_KeyboardEvent</a>.

The code above uses 3 different elements from the structure: <code>event.key.scancode</code>, <code>event.key.key</code>, <code>event.key.mod</code>, and additionally defines another variable <code>mkey</code>, with the value returned from the library function <code>SDL_GetKeyFromScancode()</code>.

After compiling and running the program, if you press only the 'A' key on your keyboard, you'll get the following:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    Loop Time(mS) = 63.6625      Keyboard: Scancode=0x4 Key=a (0x61) Modifier=0x0 ModKey=a (0x61)
</pre>

When a key is pressed on the keyboard, a 'scancode' is generated (<code>event.key.scancode</code>), which tells the library the location of the key on the keyboard.  The library checks to see what type of system it's running on, and uses that to determines what key character the 'scancode' corresponds to - typically the lower-case ASCII representation of the symbol on the keyboard, and that is presented in the <code>event.key.key</code> variable of the event structure - in this case a 'a'.

An event is generated for each key depression on a keyboard - including the shift, cntrl, lock, etc keys - if you press the left 'Shift' key on the keyboard, and then press the right 'Shift' key, the following will be output to the console - each shift key will have a different output:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    Loop Time(mS) = 50.8576      Keyboard: Scancode=0xe1 Key=� (0x400000e1) Modifier=0x1 ModKey=� (0x400000e1)
    Loop Time(mS) = 50.7513      Keyboard: Scancode=0xe5 Key=� (0x400000e5) Modifier=0x2 ModKey=� (0x400000e5)</pre>

The <code>key</code> outputs have a bit set way up in the upper nibbles, and the <code>mod</code> output (from <code>event.key.mod</code>) has it's LS bit set for the left shift key, and bit 1 set for the right shift key.  If you now press the left 'Shift' key again, and keep it held down while pressing the 'a' key, you'll get:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    Loop Time(mS) = 50.9267      Keyboard: Scancode=0xe1 Key=� (0x400000e1) Modifier=0x1 ModKey=� (0x400000e1)
    Loop Time(mS) = 50.8502      Keyboard: Scancode=0x4 Key=a (0x61) Modifier=0x1 ModKey=A (0x41)
</pre>

Now when the scancode and modifier are sent to the SDL_GetKeyFromScancode function, it will interpret them and return the capital 'A', which will be output as the <code>ModKey</code> value.

Note that if you hit the enter key, you'll see some weird behavior - that's because the console will see the 'Carriage Return' character output from your program, and interpret it by moving the cursor back to the start of the line.
<br/><br/>


### A Bit About Focus

When the window is opened by the library, it has input focus, meaning that if a keyboard key is pressed, the program will see it - once any space outside the window is clicked with the mouse, it will loose focus and key depressions will not be routed to your program until the mouse is again clicked on the window.

Mouse focus is different than keyboard input focus, and indicates when the mouse is over our window - the library gates the motion events so that the program will only get them for our window, as we saw above.  This can create some problems when building user interfaces, because the motion events don't tell us when the mouse has left the window - to our program, it's indistinguishable from simply stopping the mouse in the window.

The library takes care of the problem by adding events for when the mouse enters or leaves the window - <code>SDL_EVENT_WINDOW_MOUSE_ENTER</code> and <code>SDL_EVENT_WINDOW_MOUSE_LEAVE</code>.  Typically you'll need to know when the mouse leaves, but it can also be useful to have an event different than the motion events to indicate when the cursor has moved over your window.

Add the following two additional else clauses, save, compile and run:
<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                   // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                  // window was closed
                endProgram = true;
            else if (event.type == SDL_EVENT_MOUSE_BUTTON_DOWN) {              // mouse button pressed
                std::cout << "Mouse Click button= " <<
                             (int) event.button.button <<                      // 0=left,1=center,2=right
                             " at x,y= " << event.button.x << "," <<           // x window position
                             event.button.y << std::endl;                      // y window position
            }
            else if (event.type == SDL_EVENT_MOUSE_MOTION) {                   // mouse moving in window
                std::cout << "Mouse Move at x,y= " <<
                             event.motion.x << "," <<                          // x window position
                             event.motion.y << "  button= " <<                 // y window position
                             (int) event.motion.state << std::endl;            // 1=left,2=center,4=right (0=none)
            }
            else if (event.type == SDL_EVENT_MOUSE_WHEEL) {                    // mouse wheel
                std::cout << "Mouse wheel= " << event.wheel.y <<               // +-ticks of wheel 
                             "  at x,y= " << event.wheel.mouse_x << "," <<     // x window position
                             event.wheel.mouse_y << std::endl;                 // y window position
            }
            else if (event.type == SDL_EVENT_KEY_DOWN) {                       // keyboard key pressed
                SDL_Scancode code = event.key.scancode;
                SDL_Keycode key = event.key.key;
                SDL_Keymod mod = event.key.mod;
                SDL_Keycode mkey = SDL_GetKeyFromScancode(code, mod, false);
                std::cout << std::hex <<                                       // output to hex format
                          "Keyboard: Scancode=0x" << code <<                   // key scancode (hex)
                          " Key=" << (char)key << " (0x" << key << ")" <<      // key code in char(hex)
                          " Modifier=0x" << mod <<                             // in hex - ie shift, ctrl, alt
                          " ModKey=" << (char)mkey << " (0x" << mkey << ")" << // modified key in char (hex)
                          std::dec << std::endl;          
            }
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
            else if (event.type == SDL_EVENT_WINDOW_MOUSE_ENTER) {             // mouse on our window
                std::cout << "Mouse Entry\n";
            }
            else if (event.type == SDL_EVENT_WINDOW_MOUSE_LEAVE) {             // mouse off our window
                std::cout << "Mouse Exit\n";
            }
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        }                                                                      // end of Event Loop
    }                                                                          // end of Main Loop
</pre> </div><br/>


Now if you barely move the mouse into, then out of the window, the output will reflect the entry, movement inside, and then the exit:


<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    Loop Time(mS) = 50.8435      Mouse Entry
    Mouse Move at x,y= 999,145  button= 0
    Mouse Move at x,y= 999.521,145.758  button= 0
    Mouse Move at x,y= 999.195,145.758  button= 0
    Mouse Move at x,y= 998.806,145.758  button= 0
    Loop Time(mS) = 50.8593      Mouse Move at x,y= 999.6,145.758  button= 0
    Mouse Move at x,y= 999.6,144  button= 0
    Mouse Exit
    Loop Time(mS) = 50.7316 
</pre>



Note that the enter/leave events only apply to the drawing area of the window - if you move just onto the frame, no event is generated.





<br/>
### The Finished SDL3 Main Loop Skeleton With Basic Event Handling

Below is a complete version of the code we just worked through - it can act as a good starting point for many applications:

<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;
// compile with:   g++ SDL3-eventBasics.cpp -oSDL3-eventBasics -lSDL3 -Wall

int main(int argc, char *argv[]) {                                             // program entry point
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                    // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;

    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;     // Window and Renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                           // Initialize the library
        !SDL_CreateWindowAndRenderer("SDL3 Event Basics",                      // Create Window with Renderer
                                    1000, 400, 0, &window, &renderer)) {
        std::cout << SDL_GetError() << std::endl;
        return 1;                                                              // program exit
    }

    bool endProgram = false;
    uint64_t time=0, zTime=0, dTime=0, loopCntr=1;

    while (!endProgram) {                                                      // Main Loop
        SDL_Delay(50);                                                         // sleep for 50mS

        if ((dTime=((time=SDL_GetTicksNS())-zTime)) > 1E9) {                   // more than one second
            std::cout << "\rLoop Time(mS) = " << 1E-6*dTime/loopCntr
                                  << "      " << std::flush;
            zTime = time;
            loopCntr = 0;
        }
        ++loopCntr;

        SDL_SetRenderDrawColorFloat(renderer, 0.f, 0.f, 0.f, 1.f);             // black
        SDL_RenderClear(renderer);                                             // clear window

        // window drawing goes here

        SDL_RenderPresent(renderer);                                           // updates the screen

        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                   // Event Loop
            if (event.type == SDL_EVENT_QUIT) {                                // window was closed
                endProgram = true;
            }
            else if (event.type == SDL_EVENT_MOUSE_BUTTON_DOWN) {              // mouse button pressed
                std::cout << "Mouse Click button= " <<
                             (int) event.button.button <<                      // 0=left,1=center,2=right
                             " at x,y= " << event.button.x << "," <<           // x window position
                             event.button.y << std::endl;                      // y window position
            }
            else if (event.type == SDL_EVENT_MOUSE_MOTION) {                   // mouse moving in window
                std::cout << "Mouse Move at x,y= " <<
                             event.motion.x << "," <<                          // x window position
                             event.motion.y << "  button= " <<                 // y window position
                             (int) event.motion.state << std::endl;            // 1=left,2=center,4=right (0=none)
            }
            else if (event.type == SDL_EVENT_MOUSE_WHEEL) {                    // mouse wheel
                std::cout << "Mouse wheel= " << event.wheel.y <<               // +-ticks of wheel 
                             "  at x,y= " << event.wheel.mouse_x << "," <<     // x window position
                             event.wheel.mouse_y << std::endl;                 // y window position
            }
            else if (event.type == SDL_EVENT_KEY_DOWN) {                       // keyboard key pressed
                SDL_Scancode code = event.key.scancode;
                SDL_Keycode key = event.key.key;
                SDL_Keymod mod = event.key.mod;
                SDL_Keycode mkey = SDL_GetKeyFromScancode(code, mod, false);
                std::cout << std::hex <<                                       // output to hex format
                          "Keyboard: Scancode=0x" << code <<                   // key scancode (hex)
                          " Key=" << (char)key << " (0x" << key << ")" <<      // key code in char(hex)
                          " Modifier=0x" << mod <<                             // in hex - ie shift, ctrl, alt
                          " ModKey=" << (char)mkey << " (0x" << mkey << ")" << // modified key in char (hex)
                          std::dec << std::endl;
            }
            else if (event.type == SDL_EVENT_WINDOW_MOUSE_ENTER) {             // mouse on our window
                std::cout << "Mouse Entry\n";
            }
            else if (event.type == SDL_EVENT_WINDOW_MOUSE_LEAVE) {             // mouse off our window
                std::cout << "Mouse Exit\n";
            }
        }                                                                      // end of Event Loop
    }                                                                          // end of Main Loop

    SDL_DestroyRenderer(renderer);          SDL_DestroyWindow(window);         // Shut down the library
    SDL_Quit();
    std::cout << std::endl;                                                    // extra CR/LF

    return 0;                                                                  // program exit
}
</pre> </div><br/>





Other articles in this series on using the SDL3 Library include using the library's drawing primitives in <a href="{% post_url 2026-01-04-Rpi-C++-SDL3-Drawing-Primitives %}" style="color: blue">C++ SDL3 2D Drawing Primitives</a> and getting started with C++ Objects and TrueType text fonts in <a href="{% post_url 2026-01-06-RPi-C++-SDL3-Text-And-Objects %}" style="color: blue">C++ Text and Objects With SDL3</a>.
<br/><br/>
