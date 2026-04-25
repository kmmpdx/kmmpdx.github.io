---
layout: post
title: "RPi C++ 2D Drawing Primitives With SDL3"
author: mike
excerpt_separator: <!--more-->
---

The first part of this series on using the Simple DirectMedia Layer (SDL3) Library with C++ on the Raspberry Pi covered <a href="{% post_url 2026-01-03-Rpi-C++-SDL3-Getting-Started %}" style="color: blue">getting started</a> by loading the library and building a basic C++ program to display an empty graphics window.  In this part, we'll continue by adding basic graphics to the window using some of the two-dimensional (2D) drawing primitives provided by the library.

<br/>
<img src="/assets/blogImages/2026-01-04/SDL3-2DBasics.png" alt="SDL3graphics window with 2D primitives" width="500" />
<!--more-->
<br/> <br/> <br/>
*The usual disclaimer:  This is not expert or professional advice - it is provided for educational purposes only.  Always back up your files before installing anything.  Always understand why you're doing something.  Always understand what you're doing.  If this breaks something, it's your responsibility.*
<br/><br/>

The <a href="https://wiki.libsdl.org/SDL3/APIByCategory" style="color: blue">The SDL3 Library</a> includes a number of functions for drawing 2D primitives such as points, lines, rectangles and triangles.  We'll start by adding both unfilled and filled rectangles, then lines, points, and finally triangles.




### Open a New C++ Source File

As in the <a href="{% post_url 2026-01-03-Rpi-C++-SDL3-Getting-Started %}" style="color: blue">RPi C++ SDL3 Getting Started</a> article, we'll just use a single text file created in any plain text editor for the source code, and compile the program from the BASH command line - this is typically only useful for very small programs or testing, but is an easy way to get started with C++ programming on the Raspberry Pi.

Assuming you've made a directory to hold your test program from the Getting Started article, change to it and use the command line to create a new file in the geany text editor to use for these 2D Primitives code examples:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    cd ~/ProjectSDL3-mainbasics
    geany SDL3-2Dbasics.cpp
</pre>
<br/>


### Start With an Empty Rectangle

The following code starts with the final Main Loop skeleton program from the <a href="{% post_url 2026-01-03-Rpi-C++-SDL3-Getting-Started %}" style="color: blue">RPi C++ SDL3 Getting Started</a> article, and adds the changes in blue.  The window title is changed in the <code>SDL_CreateWindowAndRederer()</code> function call, along with adding the <code>SDL_WINDOW_RESIZABLE</code> flag in place of the previous defualt value of '0' to make the window resizable.

Above the Main Loop, two new variables of the library defined type <code>SDL_FRect</code> are declared.  The first one <code>rect1</code>, is initialized with the values in the curly brackets.  <code>SDL_FRect</code> is an SDL3 library structure that consists of 4 floating point variables; x and y to define a rectangles upper left corner, followed by the rectangle width and height (w and h) - in this case, it defines a rectangle located 10 pixels to the right of the upper left window corner and 12 pixel down, that is 50 pixels wide and 52 pixels high.

The second variable <code>rect2</code>, is of the same rectangle type, but this time, it's left uninitialized.

In the Main Loop, after clearing the window to black, the drawing color is set to orange'ish using the byte form of <code>SDL_SetRenderDrawColor()</code>, which uses 4 bytes to set each of the RGBA values from 0-255, versus the floating point function used to set the windows background color - both functions are equivalent - use whichever you prefer.

The next function call to <code>SDL_RenderRect()</code> draws the rectangle specified by the variable <code>rect1</code> to the windows off-screen drawing area using the renderer that was assigned to the window when it was created at the start of the program.




<div class="shadowbox"><pre class="monoBackground">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;
// compile with:   g++ SDL3-2Dbasics.cpp -oSDL3-2Dbasics -lSDL3 -Wall

int main(int argc, char *argv[]) {                                              // program entry point
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                     // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;

    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;      // Window and Renderer
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                            // Initialize the library
        !SDL_CreateWindowAndRenderer("SDL3 2D Basics", 1000, 400,               // Create Window with Renderer
                        SDL_WINDOW_RESIZABLE, &window, &renderer)) {
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        std::cout << SDL_GetError() << std::endl;
        return 1;                                                               // program exit
    }

    bool endProgram = false;
    uint64_t time=0, zTime=0, dTime=0, loopCntr=1;
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    SDL_FRect rect1 {10.f,12.f, 50.f,52.f};                                     // SDL_FRect = x,y,w,h
    SDL_FRect rect2;
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
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
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
        SDL_SetRenderDrawColor(renderer, 163, 154, 137, 255);                   // orange'ish Rectangle
        SDL_RenderRect(renderer, &rect1);
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
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









Copy the code into your editor, save the file, and compile from the command line as before, but with the new filename:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    g++ SDL3-2Dbasics.cpp -oSDL3-2Dbasics -lSDL3 -Wall
</pre><br/>


This time, the compiler will give you a warning that the second <code>rect2</code> variable we declared is not being used, but won't affect operation:
<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    mike@debian:~/projectSDL3-mainbasics$ g++ SDL3-mainbasics.cpp -oSDL3-mainbasics -lSDL3 -Wall
    SDL3-mainbasics.cpp: In function ‘int main(int, char**)’:
    SDL3-mainbasics.cpp:11:15: warning: unused variable ‘rect2’ [-Wunused-variable]
       11 |     SDL_FRect rect2;
          |               ^~~~~
</pre>
<br/>

Go ahead and run the program with the new filename:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    ./SDL3-2Dbasics
</pre>

On every pass of the program through it's main loop, the window image is cleared to black, the orange rectangle is drawn to the image, and the image is copied to the screen by <code>SDL_RenderPresent(renderer)</code>.  Note that becasue we added the <code>SDL_WINDOW_RESIZABLE</code> flag to the options parameter in the window creation function, it is now also resizable (but not re-scaling).

<br/>
<img src="/assets/blogImages/2026-01-04/SDL3-2DBasics0.png" alt="SDL3 graphics window with one unfilled rectangle" width="500" />
<br/> <br/>



### More Rectangles

Next, we'll use the other rectangle we defined above (<code>rect2</code>), but this time we'll use it to draw 2 different rectangles in the window.  Add the following in the main loop:
<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
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

        SDL_SetRenderDrawColor(renderer, 163, 154, 137, 255);                   // orange'ish Rectangle
        SDL_RenderRect(renderer, &rect1);
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
        rect2 = {100.f,102.f, 150.f,152.f};                                     // x,y,w,h
        SDL_SetRenderDrawColorFloat(renderer, .04f, .5f, .94f, 1.f);            // blue'ish rectangle
        SDL_RenderFillRect(renderer, &rect2);
        
        SDL_SetRenderDrawColorFloat(renderer, 255, 255, 255, 255);              // white rectangle
        rect2.y += 200.f;
        SDL_RenderFillRect(renderer, &rect2);
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        SDL_RenderPresent(renderer);                                            // updates the screen

        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                    // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                   // window was closed
                endProgram = true;
        }                                                                       // end of Event Loop
    }                                                                           // end of Main Loop
</pre> </div>



 

This time, we directly assign <code>rect2</code> the values in the curly brackets - as before, the values in order, are x, y, w, h, which define a rectangle located at 100.0, 102.0 that is 150.0 pixels wide and 152.0 pixels high, and render it with a new blue'ish color.  The SDL3 library provides two routines for drawing simple rectangles - one for just an outline, and one for filled - this time, we draw a filled rectangle with the libraries <code>SDL_RenderFillRect()</code> function.

Next, a third filled rectangle is rendered by re-using the <code>rect2</code> variable, after changing the color and after modifying the .y member of the structure to 'move' the new one down 200 pixels.

Again, compile and run - this time the compiler warning is gone.  The third white rectangle is too large to fit in the window when first drawn and is clipped at the edge, but note that you can now resize the window so it fits.

<br/>
<img src="/assets/blogImages/2026-01-04/SDL3-2DBasics3.png" alt="SDL3 graphics window with 3 rectangles" width="500" />
<br/> <br/>



### Some Lines

The SDL3 library includes 2 functions to draw lines - one for single lines, and one for multiple connected lines - add the following code below the rectangle drawing code:
<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
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

        SDL_SetRenderDrawColor(renderer, 163, 154, 137, 255);                   // orange'ish Rectangle
        SDL_RenderRect(renderer, &rect1);

        rect2 = {100.f,102.f, 150.f,152.f};                                     // x,y,w,h
        SDL_SetRenderDrawColorFloat(renderer, .04f, .5f, .94f, 1.f);            // blue'ish rectangle
        SDL_RenderFillRect(renderer, &rect2);
        
        SDL_SetRenderDrawColorFloat(renderer, 255, 255, 255, 255);              // white rectangle
        rect2.y += 200.f;
        SDL_RenderFillRect(renderer, &rect2);
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">  
        SDL_SetRenderDrawColor(renderer, 10, 249, 249, 255);                    // teal line
        SDL_RenderLine(renderer, 350.f, 50.f, 950.f, 50.f);                     // line is x1,y1, x2,y2
    
        SDL_FPoint points[6] = {350.f,100.f, 350.f,200.f,                       //SDL_FPoint = x,y
                                500.f,100.f, 500.f,200.f,
                                650.f,100.f, 650.f,200.f};
        SDL_SetRenderDrawColor(renderer, 215, 80, 110, 255);                    // rose lines
        SDL_RenderLines(renderer, points, 6);                                   // connected lines
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        SDL_RenderPresent(renderer);                                            // updates the screen

        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                    // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                   // window was closed
                endProgram = true;
        }                                                                       // end of Event Loop
    }                             
</pre> </div>


A teal single line is drawn with the <code>SDL_RenderLine()</code> function that specifies 2 endpoints - in this case, starting at x1,y1 = 350.0, 50.0 and ending at x2,y2 = 950.0, 50.0.

The second block of code renders multiple connected lines using an array of <code>SDL_FPoints</code> called <code>points</code>, with each pair of floating point values x,y in the array defining a line.  In this case, the array <code>points</code> is both defined and directly initialized with one statement.  The array elements consist of 3 pairs of x,y endpoints (6 points in total) - which, in this case, define 3 vertical lines that are each 100 pixels high, located 350.0, 500.0 and 650.0 pixels from the left edge.  The <code>SDL_RenderLines()</code> function takes the pointer to the <code>points</code> array, along with a line count of 6, and renders the lines connected together.

Compile and run the program as before - you'll notice that each of the vertical lines specified by the <code>points</code> array is drawn, and the endpoint of each line is connected to the succeeding starting point of the following line in the array - in this case, the two diagonal lines.


<br/>
<img src="/assets/blogImages/2026-01-04/SDL3-2DBasics4.png" alt="SDL3 graphics window with rectangles and lines" width="500" />
<br/> <br/>



### Points

The library point rendering routines are very similar to the line routines, with the obvious need for only one x,y pair to define each point.  Add the following code after the line drawing code below.  The first call to <code>SDL_RenderPoint()</code> just draws a single point in yellow - in this case at the start of the first horizontal line we drew above.  The 2nd call to <code>SDL_RenderPoints()</code> uses an array to draw a list of single points, with a point count.  Conveniently, the array of <code>SDL_FPoint</code>'s we created to draw the connected lines above can also be used to draw a list of points - the six x,y pairs that defines the endpoints of the lines will now get a single yellow point drawn at each.

<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
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

        SDL_SetRenderDrawColor(renderer, 163, 154, 137, 255);                   // orange'ish Rectangle
        SDL_RenderRect(renderer, &rect1);

        rect2 = {100.f,102.f, 150.f,152.f};                                     // x,y,w,h
        SDL_SetRenderDrawColorFloat(renderer, .04f, .5f, .94f, 1.f);            // blue'ish rectangle
        SDL_RenderFillRect(renderer, &rect2);
        
        SDL_SetRenderDrawColorFloat(renderer, 255, 255, 255, 255);              // white rectangle
        rect2.y += 200.f;
        SDL_RenderFillRect(renderer, &rect2);

        SDL_SetRenderDrawColor(renderer, 10, 249, 249, 255);                    // teal line
        SDL_RenderLine(renderer, 350.f, 50.f, 950.f, 50.f);                     // line is x1,y1, x2,y2
    
        SDL_FPoint points[6] = {350.f,100.f, 350.f,200.f,                       //SDL_FPoint = x,y
                                500.f,100.f, 500.f,200.f,
                                650.f,100.f, 650.f,200.f};
        SDL_SetRenderDrawColor(renderer, 215, 80, 110, 255);                    // rose lines
        SDL_RenderLines(renderer, points, 6);                                   // connected lines
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
        SDL_SetRenderDrawColor(renderer, 255, 255, 0, 255);                     // yellow points
        SDL_RenderPoint(renderer, 350.f, 50.f);
        SDL_RenderPoints(renderer, points, 6);    
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        SDL_RenderPresent(renderer);                                            // updates the screen

        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                    // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                   // window was closed
                endProgram = true;
        }                                                                       // end of Event Loop
    }                             
</pre> </div>

After saving, compiling and running, the new code draws a single point in yellow at the start of the first single line, and then at each of the points defined for the lines array.  On a high-resolution monitor, the points can be hard to spot, which can limit their use.
<br/><br/>




### Triangles

Rendering triangles in the graphics library is a bit more complicated than the other primitives.  Since defining a triangle requires the definition of 3 points, the library provides an <code>SDL_Vertex</code> structure to use.  Each vertex has an x, y pixel coordinate to define the location, and, in addition, unlike the previous primitives, each point is also given it's own color.  

In the code below, after declaring the vertex array <code>RGBtri[]</code>, the array is cleared to 0.0  using the <code>SDL_zeroa()</code> function.  Since all the array elements are set to 0.0, we can set each of the vertices color to one of Red, Green or Blue primary colors by just changing one of the RGB elements in the array to 1.0, along with the Alpha.

Once the array locations are also set, the triangle is rendered to the working image with the call to the <code>SDL_RenderGeometry()</code> function - there are a couple of other parameters available in the function that relate to texture overlays and indexing that we don't need for now.

Add the new section of code below:

<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
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

        SDL_SetRenderDrawColor(renderer, 163, 154, 137, 255);                   // orange'ish Rectangle
        SDL_RenderRect(renderer, &rect1);

        rect2 = {100.f,102.f, 150.f,152.f};                                     // x,y,w,h
        SDL_SetRenderDrawColorFloat(renderer, .04f, .5f, .94f, 1.f);            // blue'ish rectangle
        SDL_RenderFillRect(renderer, &rect2);
        
        SDL_SetRenderDrawColorFloat(renderer, 255, 255, 255, 255);              // white rectangle
        rect2.y += 200.f;
        SDL_RenderFillRect(renderer, &rect2);

        SDL_SetRenderDrawColor(renderer, 10, 249, 249, 255);                    // teal line
        SDL_RenderLine(renderer, 350.f, 50.f, 950.f, 50.f);                     // line is x1,y1, x2,y2
    
        SDL_FPoint points[6] = {350.f,100.f, 350.f,200.f,                       //SDL_FPoint = x,y
                                500.f,100.f, 500.f,200.f,
                                650.f,100.f, 650.f,200.f};
        SDL_SetRenderDrawColor(renderer, 215, 80, 110, 255);                    // rose lines
        SDL_RenderLines(renderer, points, 6);                                   // connected lines

        SDL_SetRenderDrawColor(renderer, 255, 255, 0, 255);                     // yellow points
        SDL_RenderPoint(renderer, 350.f, 50.f);
        SDL_RenderPoints(renderer, points, 6);    
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
        SDL_Vertex RGBtri[3];                                                   // point,color,point
        SDL_zeroa(RGBtri);                                                      // clear the array
        RGBtri[0].color.r = RGBtri[0].color.a = 1.f;                            // red vertex, full alpha
        RGBtri[0].position.x = 720.f;    RGBtri[0].position.y = 100.f;
        RGBtri[1].color.b = RGBtri[1].color.a = 1.f;                            // blue vertex, full alpha
        RGBtri[1].position.x = 920.f;    RGBtri[1].position.y = 100.f;
        RGBtri[2].color.g = RGBtri[2].color.a = 1.f;                            // green vertex, full alpha
        RGBtri[2].position.x = 920.f;    RGBtri[2].position.y = 300.f;
        SDL_RenderGeometry(renderer, NULL, RGBtri, 3, NULL, 0);                 // render the triangle
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        SDL_RenderPresent(renderer);                                            // updates the screen

        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                    // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                   // window was closed
                endProgram = true;
        }                                                                       // end of Event Loop
    }
</pre> </div>



After saving, compiling and running, the RGB triangle is drawn with interpolated color for each point within the outline.  Note that if you need a single color filled triangle, set all the vertices to that color - and if you need only the outline of a triangle, just use the <code>SDL_RenderLines()</code> function.

<br/>
<img src="/assets/blogImages/2026-01-04/SDL3-2DBasics.png" alt="SDL3 graphics window with 2D primitives" width="500" />
<br/> <br/>




### More About Time

If you've been watching the Main Loop time readout on your console, you'll have noticed that the additional amount of time to render these few primitives has hardly increased over the empty loop time - a few 10's of microseconds, depending upon your processor, and not significant in most applications, although once we start plotting real time data, the time can increase much more significantly - to the point where you might want to bring in another core processor to handle the graphics portion of your program.

A useful tool to examine processor loading is to use the BASH <code>top</code> command - start up your program, bring up a new command window and enter:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    top
</pre>

The <code>top</code> command will continuously display in real-time all the the processes running on your computer, along with the percentage of one CPU core that they're using:


<br/>
<img src="/assets/blogImages/2026-01-04/SDL3-2DBasicsTopRPi.png" alt="BASH top readout with 50mS sleep" width="500" />
<br/> <br/>

You'll notice that your program <code>SDL3-2Dbasics</code> is using a very small percentage of CPU time.  Now, while leaving <code>top</code> running, comment out the SDL_Delay(50) function call in your program with a couple of slashes // and save, recompile and run.  Without the 50mS sleep, the program is now using 30% of a processor, versus a percent of a processor with the Main Loop sleep:

<br/>
<img src="/assets/blogImages/2026-01-04/SDL3-2DBasicsTopRPi0.png" alt="BASH top readout with 50mS sleep" width="500" />
<br/> <br/>


In reality, your program is actually using a bit more CPU time, if you take note of the amount of time in the rendering process (ie XWayland).  As with most command line programs, just hit 'q' when you want to exit <code>top</code>.

As an aside, another useful feature of <code>top</code> is to Kill a run-away process - with <code>top</code> running, hit the 'k' key - at the prompt, enter the PID of the process you want to kill and then enter '9' for the signal to send, and hit Enter.  The <code>top</code> is very useful for characterizing programs - particularly as you start needing multiple threads in real-time applications.
<br/><br/>




### Complete Example Code

Below is a copy of the completed code from the step-by-step description above:

<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;
// compile with:   g++ SDL3-2Dbasics.cpp -oSDL3-2Dbasics -lSDL3 -Wall

int main(int argc, char *argv[]) {                                              // program entry point
    std::cout &lt;&lt; "SDL3 Library Version " &lt;&lt;                                     // Print out the library version
                        SDL_VERSIONNUM_MAJOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MINOR(SDL_GetVersion()) &lt;&lt; "." &lt;&lt;
                        SDL_VERSIONNUM_MICRO(SDL_GetVersion()) &lt;&lt;
                        std::endl;

    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;      // Window and Renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                            // Initialize the library
        !SDL_CreateWindowAndRenderer("SDL3 2D Basics", 1000, 400,               // Create Window with Renderer
                        SDL_WINDOW_RESIZABLE, &window, &renderer)) {
        std::cout << SDL_GetError() << std::endl;
        return 1;                                                               // program exit
    }

    bool endProgram = false;
    uint64_t time=0, zTime=0, dTime=0, loopCntr=1;
    SDL_FRect rect1 {10.f,12.f, 50.f,52.f};                                     // SDL_FRect = x,y,w,h
    SDL_FRect rect2;

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

        SDL_SetRenderDrawColor(renderer, 163, 154, 137, 255);                   // orange'ish Rectangle
        SDL_RenderRect(renderer, &rect1);

        rect2 = {100.f,102.f, 150.f,152.f};                                     // x,y,w,h
        SDL_SetRenderDrawColorFloat(renderer, .04f, .5f, .94f, 1.f);            // blue'ish rectangle
        SDL_RenderFillRect(renderer, &rect2);
        
        SDL_SetRenderDrawColorFloat(renderer, 255, 255, 255, 255);              // white rectangle
        rect2.y += 200.f;
        SDL_RenderFillRect(renderer, &rect2);

        SDL_SetRenderDrawColor(renderer, 10, 249, 249, 255);                    // teal line
        SDL_RenderLine(renderer, 350.f, 50.f, 950.f, 50.f);                     // line is x1,y1, x2,y2
    
        SDL_FPoint points[6] = {350.f,100.f, 350.f,200.f,                       //SDL_FPoint = x,y
                                500.f,100.f, 500.f,200.f,
                                650.f,100.f, 650.f,200.f};
        SDL_SetRenderDrawColor(renderer, 215, 80, 110, 255);                    // rose lines
        SDL_RenderLines(renderer, points, 6);                                   // connected lines

        SDL_SetRenderDrawColor(renderer, 255, 255, 0, 255);                     // yellow points
        SDL_RenderPoint(renderer, 350.f, 50.f);
        SDL_RenderPoints(renderer, points, 6);    

        SDL_Vertex RGBtri[3];                                                   // point,color,point
        SDL_zeroa(RGBtri);                                                      // clear the array
        RGBtri[0].color.r = RGBtri[0].color.a = 1.f;                            // red vertex, full alpha
        RGBtri[0].position.x = 720.f;    RGBtri[0].position.y = 100.f;
        RGBtri[1].color.b = RGBtri[1].color.a = 1.f;                            // blue vertex, full alpha
        RGBtri[1].position.x = 920.f;    RGBtri[1].position.y = 100.f;
        RGBtri[2].color.g = RGBtri[2].color.a = 1.f;                            // green vertex, full alpha
        RGBtri[2].position.x = 920.f;    RGBtri[2].position.y = 300.f;
        SDL_RenderGeometry(renderer, NULL, RGBtri, 3, NULL, 0);                 // render the triangle

        SDL_RenderPresent(renderer);                                            // updates the screen

        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                    // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                   // window was closed
                endProgram = true;
        }                                                                       // end of Event Loop
    }
}
</pre> </div>


<br/><br/>
Other articles in this series on using the SDL3 Library include showing how to handle user interaction with your program in <a href="{% post_url 2026-01-05-Rpi-C++-SDL3-Event-Basics %}" style="color: blue">C++ SDL3 Event Basics</a> and getting started with C++ Objects and TrueType text fonts in <a href="{% post_url 2026-01-06-RPi-C++-SDL3-Text-And-Objects %}" style="color: blue">C++ Text and Objects With SDL3</a>.
<br/><br/>


