## Understanding basics of /dev/fb0

### Debug info

After loading the driver, you could use `fbset` command to get information
abt the fb device.
`fbset -i -fb /dev/fb1`

### How to setup ST7735R (black tab) with rpi2

Refer pinout diagram: http://www.jameco.com/Jameco/workshop/circuitnotes/raspberry_pi_circuit_note_fig2a.jpg

#### Connections
rpi2                st7735  

1  3.3v              Vcc  
39 GND               GND   
19 MOSI GPIO10       SDA  
CS                   0 no need  
18 GPIO24               RS/DC  
22 GPIO25               RST  

### running x 
`sudo modprobe fbtft_device name=sainsmart18`  

`sudo FRAMEBUFFER=/dev/fb1 startx`
Note: You might need to move an xorg config out of the way if X doesn't start:  
`sudo mv /usr/share/X11/xorg.conf.d/99-fbturbo.conf ~`  
To make permanent settings:

Add to file /etc/modules-load.d/fbtft.conf  

`spi-bcm2835`  
`fbtft_device`  

(the auto loading of spi-bcm2835 is too late for fbtft_device to find the spi bus, so it has to be manually loaded)  

   Add to file /etc/modprobe.d/fbtft.conf  

   `options fbtft_device name=sainsmart18`  

### Speed

A monitor draws an image on the screen by using an electron beam (3 electron
                                                                  beams for color models, 1 electron beam for monochrome monitors). The front of
the screen is covered by a pattern of colored phosphors (pixels). If a phosphor
is hit by an electron, it emits a photon and thus becomes visible.

The electron beam draws horizontal lines (scanlines) from left to right, and
from the top to the bottom of the screen. By modifying the intensity of the
electron beam, pixels with various colors and intensities can be shown.

After each scanline the electron beam has to move back to the left side of the
screen and to the next line: this is called the horizontal retrace. After the
whole screen (frame) was painted, the beam moves back to the upper left corner:
this is called the vertical retrace. During both the horizontal and vertical
retrace, the electron beam is turned off (blanked).

The speed at which the electron beam paints the pixels is determined by the
dotclock in the graphics board. For a dotclock of e.g. 28.37516 MHz (millions
                                                                     of cycles per second), each pixel is 35242 ps (picoseconds) long:
Say the driver board speed is 28.37516 Mhz, each pixel is 35242 picoseconds.  
1/(28.37516 E6) =  3.524209202697007e-08

If the screen resolution is 640 x 480, it will take
640 * 3.524209202697007e-08 = 22.555E-6 s  
to paint 640 xres pixels on one scanline. but the horizontal retrace
also takes time (e.g. 272 `pixels`) so a full scanline takes  
(640 + 272) * 35.242E-9 s = 32.141E-6 s  
We will say the horizontal scanrate is abt 31 Khz.
1/32.141E-6 s) = 31.113E3 Hz  


A full screen counts 480 (yres) lines, but we have to consider the vertical
retrace too (e.g. 49 `lines'). So a full screen will take  

(480+49)*32.141E-6 s = 17.002E-3 s  

The vertical scanrate is about 59 Hz:  

1/(17.002E-3 s) = 58.815 Hz  



This means the screen data is refreshed about 59 times per second. To have a
stable picture without visible flicker, VESA recommends a vertical scanrate of
at least 72 Hz. But the perceived flicker is very human dependent: some people
can use 50 Hz without any trouble, while I'll notice if it's less than 80 Hz.


