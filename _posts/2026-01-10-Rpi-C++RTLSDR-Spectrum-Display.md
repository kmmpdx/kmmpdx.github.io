---
layout: post
title: "RPi C++ RTLSDR Spectrum Display"
author: mike
excerpt_separator: <!--more-->
---

In this article, we'll go through step-by-step instructions on how to build a simple C++ software radio spectrum display using a low-cost RTL-SDR dongle running on the Raspberry Pi 4/5 or on a Debian desktop.

The project code will use applications already installed on the Raspberry Pi and a couple of libraries that are available in the Raspberry Pi repository.


<br/>
<img src="/assets/blogImages/2026-01-10/RTLSDRspectrum8.png" alt="graphics window with plotting area rectangle" width="500" />
<!--more-->
<br/> <br/> <br/>
*The usual disclaimer:  This is not expert or professional advice - it is provided for educational purposes only.  Always back up your files before installing anything.  Always understand why you're doing something.  Always understand what you're doing.  If this breaks something, it's your responsibility.*
<br/><br/>


Many articles have been written discussing both the usefulness and limitations of RTL-SDR dongles, but their biggest advantage is they are readily available and very low-cost - which makes them useful for providing an easy introduction to software defined radios (SDR's).

The common basic dongle design using R820T tuners are limited to frequencies between about 25MHz and 1.7GHz - they can also be used with direct sampling below 25MHz, but with degraded performance.   Off-the-shelf up-converters for the dongles are available to provide better reception in the HF frequencies - there are also versions of the dongles available with built-in up-converters.  This project was only tested with dongles that have the R820/828 tuners from RTL-SDR and NooElec, but should work with any dongle compatible with the standard driver.

Since Linux distributions include the GNU C/C++ compilers, it makes getting started very easy.  In addition to the GNU C++ compiler, we'll use the RTL-SDR library to access the dongle, the Simple DirectMedia Layer (SDL3) library to build a graphical display and the Fastest Fourier Transform in the West (FFTW3) library to create the spectrum.

A familiarity with C/C++ is useful, but not required to build the application - the project is intended for folks new to C/C++ programming, or new to Linux that want to have a quick example to get started, or for folks experienced with both that just want to see a quick example of how to use the RTL-SDR library.









<br/>
### Update/Install the Library Packages from the Repository

First, install/update the apps and development libraries we'll need from the Raspberry Pi (or Debian) repository - on the command line enter:

<pre class="monoHighlight">
    sudo apt update
    sudo apt install libsdl3-dev
    sudo apt install librtlsdr-dev
    sudo apt install libusb-1.0-0-dev
    sudo apt install libfftw3-dev
    sudo apt install build-essential
    sudo apt install geany
    sudo ldconfig
</pre>


The libsdl3-dev library provides the graphics library for the program, the librtlsdr-dev library provides an interface to RTLSDR Dongles, the libusb-1.0-0-dev is the standard Linux USB interface library and the libfftw3-dev library provides an FFT library.

The build-essential installation updates the GNU compilers, and geany is a good simple editor for editing code.  Finally, the ldconfig command updates the OS library bindings.

Note, if you're using one of the dongles with a built-in upconverter like the RTL-SDR Blog V4, you'll need to install their modified driver - just follow the Linux(Debian) instructions in their <a href="https://www.rtl-sdr.com/V4/" style="color: blue">V4 Users Guide</a>.




<br/>
### Open a New C++ Source File

Since the program is small, we'll just use a single text file created in any plain-text editor for the source code for this project, and we'll compile it from the BASH command line - it's an easy way to get started with C++ programming on the Raspberry Pi.

Open a BASH command line and enter the following to make a new directory <code>ProjectRTLSDR</code> for the program, change to it, and then create a new file <code>RTLSDRspectrum.cpp</code> with the geany text editor to use for the code - the <code>.cpp</code> file extension is default for C++ source files:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    mkdir ~/ProjectRTLSDR
    cd ~/ProjectRTLSDR
    geany RTLSDRspectrum.cpp
</pre>
<br/>


### Start With a Plotting Area

Copy the block of code below into the <code>RTLSDRspectrum.cpp</code> text file in Geany.  It's a modified version of the Main Loop skeleton graphics window program from the <a href="{% post_url 2026-01-03-Rpi-C++-SDL3-Getting-Started %}" style="color: blue">RPi C++ SDL3 Getting Started</a> article, adding the changes in blue.

The window size and title is changed in the <code>SDL_CreateWindowAndRederer()</code> function call, while above the Main Loop, a new variable <code>plotArea</code> of the library defined type <code>SDL_FRect</code> is declared that will be used for the spectrum plot area, initialized with a 1024 pixel wide and 240 pixel high rectangle located 20 pixels in from the left edge and 40 pixels down from the top.

Down in the Main Loop, after clearing the window to black, the drawing color is set to a very light gray and the <code>plotArea</code> rectangle is drawn with the call to <code>SDL_RenderFillRect()</code> to create a slightly lighter area than the black background - the rectangle outline is then drawn over the top to add a border around the area with the call to <code>SDL_RenderRect()</code>, after changing the drawing color to white.




<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;

// compile with:   g++ RTLSDRspectrum.cpp -oRTLSDRspectrum -lSDL3 -Wall
    
int main(int argc, char *argv[]) {                                             // program entry point

    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;     // window and renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                           // Initialize the library
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
                  !SDL_CreateWindowAndRenderer("RTLSDR Complex Spectrum",      // Create a Window with Renderer
                       1024+40, 300+40+10, 0, &window, &renderer)) {
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        std::cout << "SDL3 Error: " << SDL_GetError() << std::endl;
        return 1;                                                              // program exit
    }

    bool endProgram = false;
    uint64_t time=0, zTime=0, dTime=0, loopCntr=1;
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    SDL_FRect plotArea {20.f, 40.f,   1024.f, 300.f};                          // SDL_FRect = x,y,w,h
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
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
        SDL_RenderClear(renderer);                                             // clear window to draw color
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
        SDL_SetRenderDrawColor(renderer, 20,20,20,255);                        // light gray plot area
        SDL_RenderFillRect(renderer, &plotArea);
        SDL_SetRenderDrawColor(renderer, 255,255,255,255);                     // white border
        SDL_RenderRect(renderer, &plotArea);
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        SDL_RenderPresent(renderer);                                           // updates the screen
        
        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                   // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                  // window was closed
                endProgram = true;
        }                                                                      // end of Event Loop
    }                                                                          // end of Main Loop
    
    SDL_DestroyRenderer(renderer);          SDL_DestroyWindow(window);         // Shut down the library
    SDL_Quit();
    std::cout << std::endl;
    return 0;                                                                  // program exit
}
</pre> </div><br/>



Save the file in your editor then open an additional BASH command line and compile the program using:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    g++ RTLSDRspectrum.cpp -oRTLSDRspectrum -lSDL3 -Wall
</pre>


Go ahead and run the program with the new filename:

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    ./RTLSDRspectrum
</pre>

On every pass of the program through it's main loop, the window image is cleared to black, and the new blank plot area is drawn:

<br/>
<img src="/assets/blogImages/2026-01-10/RTLSDRspectrum1.png" alt="graphics window with plotting area rectangle" width="500" />
<br/> <br/>



### Connect to the RTL-SDR

Next, add the <code>rtl-sdr.h</code> include file to the top of the program, along with the FFTW3 library header file <code>fftw3.h</code> and the general math function standard library <code>cmath</code> as shown in blue below.  Also changed is the comment for the command line compile, adding the <code>-lrtlsdr</code> and the <code>-lffftw3</code> for the new libraries to be linked.

The <code>rtl-sdr.h</code> include file has all of the functions available to access and control the dongle - it serves as the API documentation for the RTL-SDR's and is located in <code>/usr/include/rtl-sdr.h</code> when the library is installed from the repository - it's a good idea to open it in your editor and take a look at the functions.  Note, Debian desktops install it in <code>/usr/local/include/rtl-sdr.h</code>.

To connect to the RTL-SDR Dongle, the call to function <code>rtlsdr_get_device_count()</code> first checks to see if any dongles have been plugged into USB ports - the code only requires one to be attatched as indicated by a non-zero return, otherwise it will simply exit the program.  If a dongle is attached, it's name is fetched and printed to the console, followed by the library call to <code>rtlsdr_open()</code> to open the device and fetch the pointer <code>device</code> that will be used throughout the rest of the program to reference it with.

Once the device has been succesfully open for use, the code then initializes the minimal amount of parameters that are required for basic operation - the sample rate, the center frequency and the gain setting.

The dongle sample rate <code>Fs</code> set with the <code>rtlsdr_set_sample_rate()</code> function sets the baseband complex (IQ) output data rate.  Normal sampling gives a Nyquist bandwidth of half the sample rate, but since the dongle provides a real and imaginary complex output, the bandwidth is equal to the sample rate.  The dongle driver allows  rates between 225001Hz to 300000Hz or between 900001Hz and 3200000Hz - in this example, we're arbitrarily setting it to 2MHz, which will give us a bandwidth of 1MHZ above and below the tuned frequency.  Note that due to filtering limitations in the dongle, within about 10% in from the band edges will be subject to aliasing.

The frequency to tune the dongle to <code>F0</code>  is set with function <code>rtlsdr_set_center_frequency()</code> and is entered directly in Hz (cycles-per-second) - in this example, it's being set to 162.475MHz, which is in the middle of a block of frequencies used by NOAA weather transmitters around the US, and makes a convenient strong FM test signal.  If no station is near your location, you can use the commercial FM broadcast band - around 100.1MHz

The tuner gain <code>Gain</code> is set with <code>rtlsdr_set_tuner_gain()</code> - note the parameter passed in to the function is 10x the gain amount in db.  For example a value between 0 and 500 cooresponds to a gain setting of 0dB to 50dB.  The library has a fixed number of gain settings available depending upon the tuner in the dongle, and will use a value equal to or just above the gain setting being requested - for example, the R820T tuner preset gain values are 0, 0.9, 1.4, 2.7, 3.7, 7.7, 8.7, 12.5, 14.4, 15.7, 16.6, 19.7, 20.7, 22.9, 25.4, 28.0, 29.7, 32.8, 33.8, 36.4, 37.2, 38.6, 40.2, 42.1, 43.4, 43.9, 44.5, 48.0, and 49.6dB.  With the value of 15dB being used in the example code, the actual value used by the dongle will be 15.7dB.

Down at the end of the program, a call to the function <code>rtlsdr_close()</code> is added to release the dongle back to the operating system when the program ends.



 
 

<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
#include &lt;rtl-sdr.h&gt;
#include &lt;fftw3.h&gt;
#include &lt;cmath&gt;
// compile with:   g++ RTLSDRspectrum.cpp -oRTLSDRspectrum -lSDL3 -lrtlsdr -lfftw3 -Wall
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
int main(int argc, char *argv[]) {                                             // program entry point
   
    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;     // window and renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                           // Initialize the library
                  !SDL_CreateWindowAndRenderer("RTLSDR Complex Spectrum",      // Create a Window with Renderer
                       1024+40, 300+40+10, 0, &window, &renderer)) {
        std::cout << "SDL3 Error: " << SDL_GetError() << std::endl;
        return 1;                                                              // program exit
    }
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    if (!rtlsdr_get_device_count()) {                                          // returns the number of dongles
        std::cout << "no RTLSDR device attached\n";                            // attached - need at least one
        return 1;                                                              // else program exit
    }

    std::cout << "RTLSDR Name= " << rtlsdr_get_device_name(0);                 // print the device name

    rtlsdr_dev_t* device;                                                      // pointer for opened device
    if (rtlsdr_open(&device, 0) < 0) {                                         // attempt to open device 0
        std::cout << "  failed to open\n";
        return 1;                                                              // program exit
    }
    else std::cout << "  opened\n";

    uint32_t Fs = 2000000;                                                     // receiver sample rate of 2MHz
    uint32_t F0 = 162475000;                                                   // noaa weather station
    int Gain = 15;                                                             // 15db gain
    
    rtlsdr_set_sample_rate(device, Fs);                                        // SDR sample rate
    rtlsdr_set_center_freq(device, F0);                                        // tuning frequency
    rtlsdr_set_tuner_gain(device, Gain*10);                                    // tuner gain (10x)
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
    bool endProgram = false;
    uint64_t time=0, zTime=0, dTime=0, loopCntr=1;
    SDL_FRect plotArea {20.f, 40.f,   1024.f, 300.f};                          // SDL_FRect = x,y,w,h

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
        SDL_RenderClear(renderer);                                             // clear window to draw color

        SDL_SetRenderDrawColor(renderer, 20,20,20,255);                        // light gray plot area
        SDL_RenderFillRect(renderer, &plotArea);
        SDL_SetRenderDrawColor(renderer, 255,255,255,255);                     // white border
        SDL_RenderRect(renderer, &plotArea);

        SDL_RenderPresent(renderer);                                           // updates the screen
        
        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                   // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                  // window was closed
                endProgram = true;
        }                                                                      // end of Event Loop
    }                                                                          // end of Main Loop
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    rtlsdr_close(device);                                                      // release the RTLSDR dongle
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
    SDL_DestroyRenderer(renderer);          SDL_DestroyWindow(window);         // Shut down the library
    SDL_Quit();
    std::cout << std::endl;
    return 0;                                                                  // program exit
}
</pre> </div><br/>






Save the program and compile with the command changed to include the linker files for the new libraries <code>-lrtlsdr</code> and <code>-lfftw3</code> (which isn't yet being used), then run.

<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    g++ RTLSDRspectrum.cpp -oRTLSDRspectrum -lSDL3 -lrtlsdr -lfftw3 -Wall
    ./RTLSDRspectrum
</pre>

If you don't have an RTLSDR dongle plugged in to your computer, you'll get the error message that there's no device attached, but if one is installed, you'll get something like below, which includes several comments printed out by the library itself:


<pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
    Detached kernel driver
    Found Rafael Micro R828D tuner
    RTL-SDR Blog V4 Detected
    RTLSDR Name= Generic RTL2832U OEM  opened
    Exact sample rate is: 2000000.052982 Hz
    Loop Time(mS) = 50.9425 
</pre>
<br/>



### Get Data From the RTLSDR

There are two ways to get data from the dongle - using synchronous function calls which are 'blocking' to request and wait for a block of sampled data, or using an asynchronous interface which can continuously accept data from the dongle as it becomes available.

To do continuous real-time processing of the sampled data with the asynchronous interface requires a more complicated code structure using multiple 'threads' - in this project, to keep things simple, we'll use the synchronous data fetches which works fine for this type of display.


First, add the variables above the start of the Main Loop in blue:
- <code>N</code> is the size of the sample to get from the dongle, which is then used as the FFT size.  The FFT doesn't require any specific length - ie powers of 2 - but the data blocks read from the RTL-SDR dongle need to be multiples of 256 complex samples.

- <code>n_read</code> will hold the number of samples actually returned from the dongle - won't use in this example, but still needed for the function call.

- <code>i2buf</code> is a pointer to an unsigned integer array with <code>N</code> elements, into which the library will put the packed complex data samples from the dongle -  allocated on the heap.

- <code>plotPoints</code> and <code>plotPoints2</code> are pointers to arrays of SDL3 library type <code>SDL_FPoint</code> which are pairs of x,y floating point numbers that will be used to hold the data that is plotted to the graphics window - again, with <code>N</code> elements each, and allocated on the heap.

Down in the Main Loop, the new code in blue first clears the RTL-SDR dongles data buffer with the <code>rtlsdr_reset_buffer()</code> function, and then reads a block of complex samples with the call to <code>rtlsdr-read-synch()</code>.  Since each I/Q sample is two byte variables, a total of N sample pairs equates to 2 * N samples.

Upon return, the code falls into a <code>for loop</code>, where each packed 16 bit word returned in the <code>i2buf</code> array is unpacked into two 8 bit signed complex (I/Q) samples which are converted to the temporary floating point variables <code>dataI</code> and <code>dataQ</code> ranging from -127.4 to +127.6.

To verify basic operation, before converting the data to the frequency domain for the spectral display, and to help visualize what's going on, we'll take the 'real' (I) and 'complex' (Q) data samples and plot them.  Each of the data samples is stored in the <code>.y</code> elements of succeeding entries of the <code>plotPoints[]</code> and <code>plotPoints2[]</code> arrays, offset by the vertical center of the plot rectangle <code>plotArea.y + 0.5*plotArea.h</code>.  The cooresponding <code>.x</code> elements for each point are the pixels going across the screen, starting with the left edge of the plot window <code>plotArea.x</code>.

Once the plot data arrays have been filled, the two calls to <code>SDL_RenderLines()</code> draws a series of connected lines between each of the points in the arrays - in white for the I data, and in yellow for the Q data.




<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
    bool endProgram = false;
    uint64_t time=0, zTime=0, dTime=0, loopCntr=1;
    SDL_FRect plotArea {20.f, 40.f,   1024.f, 300.f};                          // SDL_FRect = x,y,w,h
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    int16_t N = 1024;                                                          // size of data sample
    int n_read;                                                                // temp for return sample count
    uint16_t* i2buf = (uint16_t*) malloc(sizeof(uint16_t) * N);                // array to hold RTLSDR samples
    SDL_FPoint* plotPoints = (SDL_FPoint*) malloc(sizeof(SDL_FPoint) * N);     // arrays to hold plot data points
    SDL_FPoint* plotPoints2 = (SDL_FPoint*) malloc(sizeof(SDL_FPoint) * N);
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
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
        SDL_RenderClear(renderer);                                             // clear window to draw color

        SDL_SetRenderDrawColor(renderer, 20,20,20,255);                        // light gray plot area
        SDL_RenderFillRect(renderer, &plotArea);
        SDL_SetRenderDrawColor(renderer, 255,255,255,255);                     // white border
        SDL_RenderRect(renderer, &plotArea);
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
        rtlsdr_reset_buffer(device);                                           // clear out the sample data buffer
        rtlsdr_read_sync(device, i2buf, 2*N, &n_read);                         // 8 bit I/Q samples packed in 16 bits

        for (int i=0; i<N; i++) {                                              // transfer dongle data
            float dataI = (float)((int16_t)(i2buf[i] & 0xff) - 127.4);         // unpack 8 bit samples
            float dataQ = (float)((int16_t)(i2buf[i] >> 8) - 127.4);

            plotPoints[i].x = plotArea.x + (float)i;                           // x plot points 0 -> (N-1)
            plotPoints[i].y = plotArea.y + (0.5f*plotArea.h) + dataI;          // y plot points - I data
            plotPoints2[i].x = plotPoints[i].x;                                // same x plot points
            plotPoints2[i].y = plotArea.y + (0.5f*plotArea.h) + dataQ;         // y plot points - Q data
        }

        SDL_SetRenderDrawColor(renderer, 255,255,255,255);                     // white
        SDL_RenderLines(renderer, plotPoints, N);                              // plot I sample data
        SDL_SetRenderDrawColor(renderer, 255,255,0,255);                       // yellow
        SDL_RenderLines(renderer, plotPoints2, N);                              // plot Q sample data
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        SDL_RenderPresent(renderer);                                           // updates the screen
        
        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                   // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                  // window was closed
                endProgram = true;
        }                                                                      // end of Event Loop
    }                                                                          // end of Main Loop
</pre> </div><br/>

Save the chages in the editor, then compile and run the program.  Now, on every pass through the main loop, a block of data samples is fetched from the dongle, unpacked and scaled into 2 arrays, with each drawn as a series of connected lines in the plot area.

Since the tuning frequency <code>F0</code> we programmed is in the vicinity of the US NOAA weather stations, if there's one in your area, you'll get something similar to:

<br/>
<img src="/assets/blogImages/2026-01-10/RTLSDRspectrum3.png" alt="graphics window with plotting area rectangle" width="500" />
<br/> <br/>

Because there are 7 different frequencies used for NOAA stations depending upon the region, the frequency and amplitude of the carrier might be different - the dongle tuning frequency is on the center station, and the channel spacing is 25KHz.  The above image shows the result with the station above the tuned frequency by 75KHz, so the FM carrier being shown will be approximately that difference.  Note the white trace is 90 degrees ahead of the yellow trace.  If the station is below the tuned frequency, the 2 sinusoids will reverse, and the yellow trace will be 90 degrees ahead of the white trace.

Becasue there is no scope-like trigger function in the display, it will start at random points on the carrier cycle, making it very hard to look at - you can also see the FM modulation on the carrier causing it to change 'length' like a rubber band.  You can freeze the display by clicking on the command line window and then using Ctrl-Z to halt the program - restart it with <code>fg</code> or <code>bg</code> if you like, or kill it with <code>killall</code> - or clicking on the graphics display window's close button - the OS will notice after a second or two and ask you if you want to end the program.

If no station is nearby or if you don't have an antenna that can recieve the band (or disconnect the antenna), you will get more-or-less random noise:
<br/><br/>
<img src="/assets/blogImages/2026-01-10/RTLSDRspectrum3b.png" alt="graphics window with plotting area rectangle" width="500" />
<br/> <br/>



### The Fastest Fourier Transform in the West

To convert the complex I/Q time domain samples received from the dongle to a frequency display, we'll use the venerable Fastest Fourier Transform in the West <a href="https://fftw.org/" style="color: blue">FFTW3</a> library - it has a huge amount of capability and is very well documented - it can be extremely complicated to use, but in it's simplest form is very easy to use.

In the code below, the variable declarations and setup for the FFT are done above the Main Loop as shown in blue below:
- <code>plan</code> is an object defined by the FFTW3 library to hold all the data it needs to perform the FFT.

- <code>in</code> and <code>out</code> are pointers to complex arrays of the FFTW3 library type <code>fftw_complex</code> which are pairs of x,y floating point numbers that will be used to hold the input sample data from the RTL-SDR and the output FFT frequency data - both allocated on the heap with <code>N</code> elements.

After defining the variables, the function <code>fftw_plan_dft_1d()</code> is called to initialize and configure the library associated with the variable <code>plan</code> - in this case doing a one-dimensional complex Fourier Transform of length <code>N</code>, using the <code>in</code> and <code>out</code> arrays, in the <code>FFTW_FORWARD</code> direction - the  flag <code>FFTW_ESTIMATE</code> tells the library to use it's default 'best' algorithm for the processor and environment it's running in.

Down in the Main Loop in the RTL-SDR transfer <code>for loop</code>, comment out the previous test code that placed the I/Q sample data into the plot arrays, and add the new code in blue to place the output complex sample pairs <code>dataI</code> and <code>dataQ</code> into the FFT <code>in</code> array, scaled by 1/128 to make the input samples approximately +-1.0.  The additional scaling by <code>1/N</code> is used to pre-normalize the levels that will be output from the FFT transform.

Once the transfer is complete, the function call to <code>fftw-execute()</code> does the actual transform that was specified by <code>plan</code> - It takes the complex IQ data samples in the <code>in</code> array and converts them to a complex frequency domain output in the <code>out</code> array.

In the next <code>for loop</code>, the <code>.x</code> elements are done slightly differently than was done above for the I/Q display - each point is scaled by the width of the plot area so that if the size <code>N</code> of the FFT is changed, it will still fit.

For <code>.y</code>, the log magnitude is calculated for each output frequncy 'bin' in the FFT <code>out</code> array, and each bin is then mapped to correctly place them across the display.  The first N/2 points in the <code>out</code> array are the 'positive' frequencies, and start with 'DC' and increase up towards Fs/2 - in this case with the complex down-converted data from the RTL-SDR, they correspond to frequencies from the tuned center frequency up towards Fs/2.

Similarly, the next N/2 points in the FFT <code>out</code> array correspond to the 'negative' frequencies that start from -Fs/2, and increase up towards 'DC' - and in this case correspond to frequncies from -Fs/2 up towards the tuned center frequency.

The data is then plotted the same as was done with the I/Q sample display - we just don't need the second 'trace'.







<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
    bool endProgram = false;
    uint64_t time=0, zTime=0, dTime=0, loopCntr=1;
    SDL_FRect plotArea {20.f, 40.f,   1024.f, 300.f};                          // SDL_FRect = x,y,w,h
    int16_t N = 1024;                                                          // size of data sample
    int n_read;                                                                // temp for return sample count
    uint16_t* i2buf = (uint16_t*)calloc(N, sizeof(uint16_t));                  // array to hold RTLSDR samples
    SDL_FPoint* plotPoints = (SDL_FPoint*) malloc(sizeof(SDL_FPoint) * (N+1)); // array to hold plot data points
    SDL_FPoint* plotPoints2 = (SDL_FPoint*) malloc(sizeof(SDL_FPoint) * N);
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    fftw_plan p;                                                               // fftw3 library control structure
    fftw_complex* in = (fftw_complex*) fftw_malloc(sizeof(fftw_complex) * N);  // allocate complex input array
    fftw_complex* out = (fftw_complex*) fftw_malloc(sizeof(fftw_complex) * N); // allocate complex output array

    if ((p = fftw_plan_dft_1d(N,in,out,FFTW_FORWARD,FFTW_ESTIMATE))==NULL) {   // setup fft library plan
        std::cout << "FFTW Plan Error\n";                                      // couldn't form a plan
        return 1;                                                              // program exit
    }
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
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
        SDL_RenderClear(renderer);                                             // clear window to draw color
                                                                               
        SDL_SetRenderDrawColor(renderer, 20,20,20,255);                        // light gray inside
        SDL_RenderFillRect(renderer, &plotArea);
        SDL_SetRenderDrawColor(renderer, 255,255,255,255);                     // white border
        SDL_RenderRect(renderer, &plotArea);

        rtlsdr_reset_buffer(device);                                           // clear out the sample data buffer
        rtlsdr_read_sync(device, i2buf, 2*N, &n_read);                         // 8 bit I/Q samples packed in 16 bits

        for (int i=0; i<N; i++) {                                              // transfer dongle data
            float dataI = (float)((int16_t)(i2buf[i] & 0xff) - 127.4);         // unpack 8 bit signed
            float dataQ = (float)((int16_t)(i2buf[i] >> 8) - 127.4);           // complex samples
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
//            plotPoints[i].x = plotArea.x + (float)i;                           // x plot points 0 -> (N-1)
//            plotPoints[i].y = plotArea.y + (0.5f*plotArea.h) + dataI;          // y plot points - I data
//            plotPoints2[i].x = plotPoints[i].x;                                // same x plot points
//            plotPoints2[i].y = plotArea.y + (0.5f*plotArea.h) + dataQ;         // y plot points - Q data

            in[i][0] = dataI / (N * 128.f);                                    // normalize complex samples
            in[i][1] = dataQ / (N * 128.f);                                    // to +-1.f (and fft scale)
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        }
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
        fftw_execute(p);                                                       // do the fft for the N sample block

        for (int i=0; i<N; i++) {                                              // transfer fft data
            plotPoints[i].x = plotArea.x + (float)i * (plotArea.w / (float)N); // scale per plot width and fft size
            float mag = 20.l * log10l(sqrtl(out[i][0]*out[i][0]                // complex magnitude
                                           + out[i][1]*out[i][1]));
            if (i < N/2)                                                       // map the complex frequency bins
                plotPoints[i+N/2].y = plotArea.y - 3.f * mag;
            else     // i >= N/2
                plotPoints[i-N/2].y = plotArea.y - 3.f * mag;
        }
        
        SDL_SetRenderDrawColor(renderer, 255,255,255,255);                     // white
        SDL_RenderLines(renderer, plotPoints, N);                              // plot FFT output
//        SDL_SetRenderDrawColor(renderer, 255,255,0,255);                       // yellow
//        SDL_RenderLines(renderer, plotPoints2, N);                              // plot Q sample data
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        SDL_RenderPresent(renderer);                                           // updates the screen
        
        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                   // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                  // window was closed
                endProgram = true;
        }                                                                      // end of Event Loop
    }                                                                          // end of Main Loop
</pre> </div><br/>


Save the chages in the editor, then compile - you'll get a warning for the unused variable <code>plotPoints2</code>, but go ahead and run.

Again, if there's a NOAA weather station in your area, this time you'll see something similar to this, which shows a station at 162.55MHz with the dongle tuned to 162.475MHz - check the NOAA website for stations in your area: <a href="https://www.weather.gov/nwr/station_listing" style="color: blue">NOAA Broadcast Frequencies</a>:

<br/>
<img src="/assets/blogImages/2026-01-10/RTLSDRspectrum4.png" alt="graphics window with plotting area rectangle" width="500" />
<br/> <br/>


Because the plot window is 300 pixels high, and because we're using the log magnitude scaled by a factor of 3x when we plot, the display will show 0 to -100dB referenced to the full-scale of the IQ sampling data converters.

If no station is nearby, or if you don't have an antenna that can recieve the band, you'll get an empty noise floor:

<br/>
<img src="/assets/blogImages/2026-01-10/RTLSDRspectrum5.png" alt="graphics window with plotting area rectangle" width="500" />
<br/> <br/>



### Add a Window


To improve the suppression of sidelobes from strong signals, we'll use a Blackman Window to shape the input data before the FFT - <a href="https://en.wikipedia.org/wiki/Window_function" style="color: blue">Window Functions</a> are almost always used in conjunction when an FFT is used for analysis - the code above the Main Loop in blue below declares a pointer <code>fftwindow</code> to  a floating point array which is allocated on the heap, the entries of which are then loaded with the Blackman function:

<br/>
<img src="/assets/blogImages/2026-01-10/RTLSDRspectrum9.png" alt="blackman window function" width="500" />
<br/> <br/>




Down in the Main Loop, the change in blue scales each entry of the <codedataI</code> and <codedataQ</code> input data samples by each entry of the window array as they are transfered to the FFT <code>in</code> array, yielding the windowed data set that is the input to the FFT:

<br/>
<img src="/assets/blogImages/2026-01-10/RTLSDRspectrum10.png" alt="blackman window function" width="500" />
<br/> <br/>


Since the Blackman window peak is 1.0 at the center and tapers to 0 on either side, the data from the dongle is similarly full-valued at the center and is scaled down to 0 on either edge.



<br/>
<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;"> 
    float* fftwindow = (float*)malloc(N * sizeof(float));                      // allocate array for window
    for (int i=0; i<N; i++)                                                    // blackman window
        fftwindow[i] = 0.42l - 0.5l*cos(2.l*i*M_PIl/(N-1)) +
                                0.08l*cos(4.l*i*M_PIl/(N-1));
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
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
        SDL_RenderClear(renderer);                                             // clear window to draw color
                                                                               
        SDL_SetRenderDrawColor(renderer, 20,20,20,255);                        // light gray inside
        SDL_RenderFillRect(renderer, &plotArea);
        SDL_SetRenderDrawColor(renderer, 255,255,255,255);                     // white border
        SDL_RenderRect(renderer, &plotArea);

        rtlsdr_reset_buffer(device);                                           // clear out the sample data buffer
        rtlsdr_read_sync(device, i2buf, 2*N, &n_read);                         // 8 bit I/Q samples packed in 16 bits

        for (int i=0; i<N; i++) {                                              // transfer dongle data
            float dataI = (float)((int16_t)(i2buf[i] & 0xff) - 127.4);         // unpack 8 bit signed
            float dataQ = (float)((int16_t)(i2buf[i] >> 8) - 127.4);           // complex samples

//            plotPoints[i].x = plotArea.x + (float)i;                           // x plot points 0 -> (N-1)
//            plotPoints[i].y = plotArea.y + (0.5f*plotArea.h) + dataI;          // y plot points - I data
//            plotPoints2[i].x = plotPoints[i].x;                                // same x plot points
//            plotPoints2[i].y = plotArea.y + (0.5f*plotArea.h) + dataQ;         // y plot points - Q data
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
            in[i][0] = fftwindow[i] * dataI / (N * 128.f);                     // scale and window the
            in[i][1] = fftwindow[i] * dataQ / (N * 128.f);                     // complex samples
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        }

        fftw_execute(p);                                                       // do the fft for the N sample block

        for (int i=0; i<N; i++) {                                              // transfer fft data
            plotPoints[i].x = plotArea.x + (float)i * (plotArea.w / (float)N); // scale per plot width and fft size
            float mag = 20.l * log10l(sqrtl(out[i][0]*out[i][0]                // complex magnitude
                                           + out[i][1]*out[i][1]));
</pre> </div><br/>





### Add a Graticule and Frequency Readout

Next, we'll add a center frequency readout to the window and draw a graticule on the display area.  In the code below, first, comment out the <code>SDL_RenderRect()</code> function call that draws the previous plot area outline, then add the <code>for loop</code> which draws 10 vertical lines, starting with the left edge, and moving across the plot area, while also drawing 10 horizontal lines, starting at the top edge, then moving down.

To output the tuning center frequency, we'll use the SDL3 Libraries 'debug text', which is a fixed bit-mapped text that is simple to use.  The frequency readout code first converts the center frequency <code>F0</code> floating point variable to a string in the <code>fstr[]</code> character array using the C++ library function <code>snprintf()</code>, after which it's drawn with the <code>SDL_RenderDebugText()</code> function.

To make the blocky and small debug text look a bit better, it's scaled 2x horizontally and 3x vertically by presetting the scaling of the renderer, which is set back to 1:1 after drawing the text for the additional drawing that follows in the Main Loop.

The position of the text is centered by shifting it left from the horizontal center of the window by half the scaled pixel width of the text string <code>fstrlen</code> that was returned from the conversion to the output character string.




<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
        ++loopCntr;

        SDL_SetRenderDrawColorFloat(renderer, 0.f, 0.f, 0.f, 1.f);             // black
        SDL_RenderClear(renderer);                                             // clear window to draw color

        SDL_SetRenderDrawColor(renderer, 20,20,20,255);                        // light gray inside
        SDL_RenderFillRect(renderer, &plotArea);

        SDL_SetRenderDrawColor(renderer, 255,255,255,255);                     // white
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
//      SDL_RenderRect(renderer, &plotArea);

        for (int i=0;i<=10;i++) {                                              // 10x10 graticule
            float gx = plotArea.x + i * plotArea.w / 10.f;                     // vertical lines
            SDL_RenderLine(renderer, gx, plotArea.y,  gx, plotArea.y+plotArea.h);
            float gy = plotArea.y + i * plotArea.h / 10.f;                     // horizontal lines
            SDL_RenderLine(renderer, plotArea.x, gy,  plotArea.x+plotArea.w, gy);
         }
                                                                               // Frequency readout
        char fstr[15];                                                         // char array to hold frequency
        int fstrlen = snprintf(fstr, 15, "%u", F0);                            // convert F0 to a char array
        float scaleX= 2.f, scaleY=3.f;                                         // debug text 8x8 with 2x3 scale
        SDL_SetRenderScale(renderer, scaleX, scaleY);
        SDL_RenderDebugText(renderer, (plotArea.x + plotArea.w/2.f -           // output text centered
                           fstrlen*scaleX*8.f/2.f)/scaleX, 10.f/scaleY, fstr);
        SDL_SetRenderScale(renderer, 1.f, 1.f);                                // restore scale to 1x
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        rtlsdr_reset_buffer(device);                                           // clear out the sample data buffer
        rtlsdr_read_sync(device, i2buf, 2*N, &n_read);                         // 8 bit I/Q samples packed in 16 bits

</pre> </div><br/>

 

Again, save, compile and run:


<br/>
<img src="/assets/blogImages/2026-01-10/RTLSDRspectrum8.png" alt="graphics window with plotting area rectangle" width="500" />
<br/> <br/>




### The Whole Project With Some Control


Below is the complete project, with a bit more code added in the Event Loop of the Main Loop to provide some control - if a key on the keyboard is pressed, the library event <code>SDL_EVENT_KEY_DOWN</code> occurs, and if the key pressed is an <code>'m'</code> key, the <code>F0</code> tuning variable is increased by 100000, or 100KHz, and then sent to the RTL-SDR using the <code>rtlsdr_set_center_freq()</code> library function.  If an <code>'n'</code> key is seen, <code>F0</code> is decreased by 100000, or 100KHz.  Note that the code isn't limit checking the tuning variable.

Similarly, the 'x' and 'z' keys are used to increase or decrease the gain variable of the receiver by 50 which corresponds to 5 dB - the code does limit-check the <code>Gain</code> variable since the available range is rather limited, after which the value is sent to the dongle with  <code>rtlsdr_set_tuner_gain()</code>.





<div class="shadowbox"> <pre style="font-size: 12px; font-family: monospace;">
#include &lt;iostream&gt;
#include &lt;SDL3/SDL.h&gt;
#include &lt;rtl-sdr.h&gt;
#include &lt;fftw3.h&gt;
#include &lt;cmath&gt;
// compile with:   g++ RTLSDRspectrum.cpp -oRTLSDRspectrum -lSDL3 -lrtlsdr -lfftw3 -Wall

int main(int argc, char *argv[]) {                                             // program entry point
   
    SDL_Window *window = NULL;              SDL_Renderer *renderer = NULL;     // window and renderer
    if (!SDL_Init(SDL_INIT_VIDEO) ||                                           // Initialize the library
                  !SDL_CreateWindowAndRenderer("RTLSDR Complex Spectrum",      // Create a Window with Renderer
                       1024+40, 300+40+10, 0, &window, &renderer)) {
        std::cout << "SDL3 Error: " << SDL_GetError() << std::endl;
        return 1;                                                              // program exit
    }

    if (!rtlsdr_get_device_count()) {                                          // returns the number of dongles
        std::cout << "no RTLSDR device attached\n";                            // attached - need at least one
        return 1;                                                              // else program exit
    }

    std::cout << "RTLSDR Name= " << rtlsdr_get_device_name(0);                 // print the device name

    rtlsdr_dev_t* device;                                                      // pointer for opened device
    if (rtlsdr_open(&device, 0) < 0) {                                         // attempt to open device 0
        std::cout << "  failed to open\n";
        return 1;                                                              // program exit
    }
    else std::cout << "  opened\n";

    uint32_t Fs = 2000000;                                                     // receiver sample rate of 2MHz
    uint32_t F0 = 162475000;                                                   // noaa weather station
    int Gain = 15;                                                             // 15db gain
    
    rtlsdr_set_sample_rate(device, Fs);                                        // SDR sample rate
    rtlsdr_set_center_freq(device, F0);                                        // tuning frequency
    rtlsdr_set_tuner_gain(device, Gain*10);                                    // tuner gain (10x)

    bool endProgram = false;
    uint64_t time=0, zTime=0, dTime=0, loopCntr=1;
    SDL_FRect plotArea {20.f, 40.f,   1024.f, 300.f};                          // SDL_FRect = x,y,w,h

    int16_t N = 1024;                                                          // size of data sample
    int n_read;                                                                // temp for return sample count
    uint16_t* i2buf = (uint16_t*) malloc(sizeof(uint16_t) * N);                // array to hold RTLSDR samples
    SDL_FPoint* plotPoints = (SDL_FPoint*) malloc(sizeof(SDL_FPoint) * N);     // arrays to hold plot data points
//    SDL_FPoint* plotPoints2 = (SDL_FPoint*) malloc(sizeof(SDL_FPoint) * N);

    fftw_plan p;                                                               // fftw3 library control structure
    fftw_complex* in = (fftw_complex*) fftw_malloc(sizeof(fftw_complex) * N);  // allocate complex input array
    fftw_complex* out = (fftw_complex*) fftw_malloc(sizeof(fftw_complex) * N); // allocate complex output array
    if ((p = fftw_plan_dft_1d(N,in,out,FFTW_FORWARD,FFTW_ESTIMATE))==NULL) {   // setup fft library plan
        std::cout << "FFTW Plan Error\n";                                      // couldn't form a plan
        return 1;                                                              // program exit
    }

    float* fftwindow = (float*)malloc(N * sizeof(float));                      // allocate array for window
    for (int i=0; i<N; i++)                                                    // blackman window
        fftwindow[i] = 0.42l - 0.5l*cos(2.l*i*M_PIl/(N-1)) +
                                0.08l*cos(4.l*i*M_PIl/(N-1));

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
        SDL_RenderClear(renderer);                                             // clear window to draw color

        SDL_SetRenderDrawColor(renderer, 20,20,20,255);                        // light gray plot area
        SDL_RenderFillRect(renderer, &plotArea);

        SDL_SetRenderDrawColor(renderer, 255,255,255,255);                     // white graticle
         for (int i=0;i<=10;i++) {                                             // 10x10 graticule
            float gx = plotArea.x + i * plotArea.w / 10.f;                     // vertical lines
            SDL_RenderLine(renderer, gx, plotArea.y,  gx, plotArea.y+plotArea.h);
            float gy = plotArea.y + i * plotArea.h / 10.f;                     // horizontal lines
            SDL_RenderLine(renderer, plotArea.x, gy,  plotArea.x+plotArea.w, gy);
         }
                                                                               // Frequency readout
        char fstr[15];                                                         // char array to hold frequency
        int fstrlen = snprintf(fstr, 15, "%u", F0);                            // convert F0 to a char array
        float scaleX= 2.f, scaleY=3.f;                                         // debug text 8x8 with 2x3 scale
        SDL_SetRenderScale(renderer, scaleX, scaleY);
        SDL_RenderDebugText(renderer, (plotArea.x + plotArea.w/2.f -           // output text centered
                           fstrlen*scaleX*8.f/2.f)/scaleX, 10.f/scaleY, fstr);
        SDL_SetRenderScale(renderer, 1.f, 1.f);                                // restore scale to 1x

        rtlsdr_reset_buffer(device);                                           // clear out the sample data buffer
        rtlsdr_read_sync(device, i2buf, 2*N, &n_read);                         // 8 bit I/Q samples packed in 16 bits

        for (int i=0; i<N; i++) {                                              // transfer dongle data
            float dataI = (float)((int16_t)(i2buf[i] & 0xff) - 127.4);         // unpack 8 bit samples
            float dataQ = (float)((int16_t)(i2buf[i] >> 8) - 127.4);

//            plotPoints[i].x = plotArea.x + (float)i;                           // x plot points 0 -> (N-1)
//            plotPoints[i].y = plotArea.y + (0.5f*plotArea.h) + dataI;          // y plot points - I data
//            plotPoints2[i].x = plotPoints[i].x;                                // same x plot points
//            plotPoints2[i].y = plotArea.y + (0.5f*plotArea.h) + dataQ;         // y plot points - Q data

            in[i][0] = fftwindow[i] * dataI / (N * 128.f);                     // scale and window the
            in[i][1] = fftwindow[i] * dataQ / (N * 128.f);                     // complex samples
        }

        fftw_execute(p);                                                       // do the fft for the N sample block

        for (int i=0; i<N; i++) {                                              // transfer fft data
            plotPoints[i].x = plotArea.x + (float)i * (plotArea.w / (float)N); // scale per plot width and fft size
            float mag = 20.l * log10l(sqrtl(out[i][0]*out[i][0]                // complex magnitude
                                           + out[i][1]*out[i][1]));
            if (i < N/2)                                                       // map the complex frequency bins
                plotPoints[i+N/2].y = plotArea.y - 3.f * mag;
            else     // i >= N/2
                plotPoints[i-N/2].y = plotArea.y - 3.f * mag;
        }
        
        SDL_SetRenderDrawColor(renderer, 255,255,255,255);                     // white
        SDL_RenderLines(renderer, plotPoints, N);                              // plot FFT output

//        SDL_SetRenderDrawColor(renderer, 255,255,255,255);                     // white
//        SDL_RenderLines(renderer, plotPoints, N);                              // plot I sample data
//        SDL_SetRenderDrawColor(renderer, 255,255,0,255);                       // yellow
//        SDL_RenderLines(renderer, plotPoints2, N);                              // plot Q sample data

        SDL_RenderPresent(renderer);                                           // updates the screen
        
        SDL_Event event;
        while (SDL_PollEvent(&event) != 0) {                                   // Event Loop
            if (event.type == SDL_EVENT_QUIT)                                  // window was closed
                endProgram = true;
</pre><pre style="font-size: 12px; font-family: monospace; color: #04b3d6;">
            else if (event.type == SDL_EVENT_KEY_DOWN) {                       // keyboard key pressed
                SDL_Keycode key = event.key.key;
                if (key == 'm') {
                    F0 += 100000;
                    std::cout << "F0= " << F0 << std::endl;
                    rtlsdr_set_center_freq(device, F0);
                }
                else if (key == 'n') {
                    F0 -= 100000;
                    std::cout << "F0= " << F0 << std::endl;
                    rtlsdr_set_center_freq(device, F0);
                }
                else if (key == 'x') {
                    Gain += 5;                                                 // +5dB
                    if (Gain > 50) Gain = 50;                                  // limit to 50dB gain
                    rtlsdr_set_tuner_gain(device, Gain*10);
                    std::cout << "gain= " << Gain << std::endl;
                }
                else if (key == 'z') {
                    Gain -= 5;                                                 // -5dB
                    if (Gain < 0) Gain = 0;                                    // limit to 0dB gain
                    rtlsdr_set_tuner_gain(device, Gain*10);
                    std::cout << "gain= " << Gain << std::endl;
                }
            }
</pre><pre style="font-size: 12px; font-family: monospace; color: #000000;">
        }                                                                      // end of Event Loop
    }                                                                          // end of Main Loop

    rtlsdr_close(device);                                                      // release the RTLSDR dongle
    SDL_DestroyRenderer(renderer);          SDL_DestroyWindow(window);         // Shut down the library
    SDL_Quit();
    std::cout << std::endl;
    return 0;                                                                  // program exit
}
</pre> </div><br/>






### Some Notes

Both the Pi4 and Pi5 can easily handle larger FFT's - adjust or remove the Main Loop sleep time as needed.  For example, a Pi5 can do a 16384 point FFT in about 50mS using about 70% of one one of it's processors - quite respectable.

Be aware that the output from the RTL-SDR will exhibit aliasing within about 10% of the upper and lower band edges - in this project, with the sample rate set to 2MHz, approximately 200KHz at either edge will be subject to aliasing by signals outside the band edges.

Per the RTL-SDR library header file <code>/usr/include/rtl-sdr.h</code>, the available range of sample rates for the dongles is restricted to between 225001Hz to 300000Hz or between 900001Hz and 3200000Hz - and is going to miss samples if the rate is set above 2.4MHz, but that won't be very noticable in this type of usage.

If you don't need to see all of the output spectrum you might select part of the FFT to display - for example, if you want to show 300KHz of bandwidth, it might make sense to sample at 1MHz, then display only 15% of the frequency bins above and below the center frequency to alleviate aliasing problems, or to allow zooming.  Similarly, you don't have to display only around the center tuned frequency - you can chose any slice of bandwidth you might want.

Because the FFTW3 library can handle non-powers-of-two sample sizes, you can adjust the center frequency, sample rate and/or FFT size to 'tune' frequency bins to line up with specific frequencies of interest - just note that the dongle wants to output data in multiples of 256 byte blocks, so just capture more samples than the FFT requires.

The dongles do have Automatic Gain Control (AGC) functions, but they're usually not very useful - do note that lower gains will typically exhibit less distortion artifacts, and usually yield a better signal-to-noise ratios.

Some useful additional features to consider adding:

- add a keyboard key to switch between frequency and time domain displays.
- add a keyboard key to freeze the display.
- add a peak detect or filter on each bin of the FFT output.

