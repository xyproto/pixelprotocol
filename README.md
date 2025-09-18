# pixelprotocol

PixelProtocol (pipro) is a protocol for defining what is being sent between the GUI client and the game engine.

It's for implementing games where old-school looking pixel art can be appreciated.

## Features and limitations

* 256 indexed colors.
* A size of 320x200 pixels is recommended.
* Should be possible to implement both in 16-bit assembly for DOS and in a browser.
* 4-channel audio with multiple waveforms and ADSR envelopes.
* Text rendering with 8 font slots and scaling.
* Advanced sprite operations including rotation, scaling, and collision detection.
* Timing control and frame synchronization.
* Memory management for palettes and screen buffers.

## Q&A

* Q: Wouldn't it be cooler if Vulkan commands was sent instead? Or OpenGL? Or SDL2?
* A: Protocols for OpenGL over network already exists and I want to keep things really simple.

* Q: Can't you just use VNC?
* A: No, I want something specifically for games or demoscene demos that use 320x200 pixels, 256 colors.

# Protocol Definition

* Version: 0.2

## Protocol Header

| name       | type              | description                                     |
|------------|-------------------|-------------------------------------------------|
| ver        | uint16            | protocol version                                |
| width      | uint16            | width                                           |
| height     | uint16            | height                                          |
| commands   | []uint16          | list of commands (uint8 cmd + uint8 argument)   |

The commands can be streamed.

## Commands

### Color palette

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x00 | choose palette color                          | prepare for filling the palette          |
| 0x01 | set red value of chosen palette color         | set the color                            |
| 0x02 | set green value of chosen palette color       | set the color                            |
| 0x03 | set blue value of chosen palette color        | set the color                            |

### Drawing pixels

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x04 | choose pixel color                            | prepare for drawing                      |
| 0x05 | set x position                                |                                          |
| 0x06 | set y position                                |                                          |
| 0x07 | add to x position                             |                                          |
| 0x08 | add to y position                             |                                          |
| 0x09 | plot                                          | draw a pixel                             |

##### Q&A

* Q: What is the "add to x position" command for?
* A: since all arguments are bytes, it's needed to be able to specify X coordinates from 256..320.

* Q: Isn't that a bit impractical?
* A: Perhaps, but it makes the protocol very simple and uniform. All commands takes a byte as an argument.


### Filling

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x0a | clear                                         | clear everything with the selected color |

---

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x0b | draw linewise until nonblack or end           | for filling the pixel buffer             |
| 0x0c | draw backwards linewise until nonblack or end | for filling the pixel buffer             |

### Flipping

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x0d | flip                                          | update all pixels                        |
| 0x0e | sprite flip                                   | update pixels where sprites have been    |

### Drawing lines

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x0f | choose color for start of line                | prepare to draw a line                   |
| 0x10 | choose color for end of line                  | prepare to draw a line                   |

---

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x11 | set x coordinate for start of line            | prepare to draw a line                   |
| 0x12 | set y coordinate for start of line            | prepare to draw a line                   |
| 0x13 | set x coordinate for end of line              | prepare to draw a line                   |
| 0x14 | set y coordinate for end of line              | prepare to draw a line                   |

---

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x15 | add to x coordinate for start of line         | prepare to draw a line                   |
| 0x16 | add to y coordinate for start of line         | prepare to draw a line                   |
| 0x17 | add to x coordinate for end of line           | prepare to draw a line                   |
| 0x18 | add to y coordinate for end of line           | prepare to draw a line                   |

---

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x19 | draw a line                                   | draw the line                            |


### Drawing triangles

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x1a | choose color for p0                           | prepare to draw a filled triangle        |
| 0x1b | choose color for p1                           | prepare to draw a filled triangle        |
| 0x1c | choose color for p2                           | prepare to draw a filled triangle        |

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x1d | set x coordinate for p0                       | prepare to draw a filled triangle        |
| 0x1e | set y coordinate for p0                       | prepare to draw a filled triangle        |
| 0x1f | set x coordinate for p1                       | prepare to draw a filled triangle        |
| 0x20 | set y coordinate for p1                       | prepare to draw a filled triangle        |
| 0x21 | set x coordinate for p2                       | prepare to draw a filled triangle        |
| 0x22 | set y coordinate for p2                       | prepare to draw a filled triangle        |

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x23 | add to x coordinate for p0                    | prepare to draw a filled triangle        |
| 0x24 | add to y coordinate for p0                    | prepare to draw a filled triangle        |
| 0x25 | add to x coordinate for p1                    | prepare to draw a filled triangle        |
| 0x26 | add to y coordinate for p1                    | prepare to draw a filled triangle        |
| 0x27 | add to x coordinate for p2                    | prepare to draw a filled triangle        |
| 0x28 | add to y coordinate for p2                    | prepare to draw a filled triangle        |

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x29 | draw a filled or empty triangle               | 0 for empty, 1 for filled                |

### Randomization

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x2a | choose a random color for the pixel           |                                          |
| 0x2b | choose random colors for the line             |                                          |
| 0x2c | choose random colors for the triangle         |                                          |

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x2d | choose random coordinates for the pixel       |                                          |
| 0x2e | choose random coordinates for the line        |                                          |
| 0x2f | choose random coordinates for the triangle    |                                          |

### Sprites

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x30 | choose sprite ID                              | select a sprite to work with             |
| 0x31 | set sprite width                              | set sprite width                         |
| 0x32 | set sprite height                             | set sprite height                        |
| 0x33 | clear sprite                                  | clear contents                           |
| 0x34 | push pixel                                    | adds N pixels of the selected color      |
| 0x35 | push empty                                    | add N transparent pixels                 |
| 0x36 | set x coordinate for drawing the sprite       |                                          |
| 0x37 | set y coordinate for drawing the sprite       |                                          |
| 0x38 | add to x coordinate for drawing the sprite    |                                          |
| 0x39 | add to y coordinate for drawing the sprite    |                                          |
| 0x3a | draw sprite                                   | draw the current sprite, 2 for dbl. size |

### Convolution Filters

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x40 | set convolution filter 0                      | blur is 0,1,0,1,1,1,0,1,0 div 5          |
| 0x41 | set convolution filter 1                      | flame is 0,1,0,1,1,1,0,0,0 div 4         |
| 0x42 | set convolution filter 2                      |                                          |
| 0x43 | set convolution filter 3                      | the uint8 is treated as a range from     |
| 0x44 | set convolution filter 4                      | 0.0 to 1.0                               |
| 0x45 | set convolution filter 5                      |                                          |
| 0x46 | set convolution filter 6                      |                                          |
| 0x47 | set convolution filter 7                      |                                          |
| 0x48 | set convolution filter 8                      |                                          |
| 0x49 | set convolution division                      |                                          |
| 0x4a | use convolution filter                        |                                          |

## Keyboard, Joystick and Mouse

For returning the state of the client:

Commands that return an uint16:

| cmd  | uint8 argument                                |                                                                       |
|------|-----------------------------------------------|-----------------------------------------------------------------------|
| 0x60 | is Esc                                        | is Escape being pressed?                                              |
| 0x61 | is up                                         | is W, up or joystick up pressed? args: 0 for both, 1 for P1, 2 for P2 |
|      |                                               | Player1 has WASD and Joy1, Player2 has arrows and Joy2                |
| 0x62 | is left                                       | is A, left or joystick left pressed?                                  |
| 0x63 | is down                                       | is S, down or joystick down pressed?                                  |
| 0x64 | is right                                      | is D, right or joystick right pressed?                                |
| 0x65 | is A                                          | is Return, Comma (,) or joystick A pressed?                           |
| 0x66 | is B                                          | is Space, Dot (.) or joystick B pressed?                              |

---

| cmd  | uint8 argument                                |                                                                       |
|------|-----------------------------------------------|-----------------------------------------------------------------------|
| 0x67 | is shift held down                            | uint8 argument: 0 for left, 1 for right, 2 for any, returns 1 for held down |
| 0x68 | is alt held down                              |                                                                       |
| 0x69 | is ctrl held down                             |                                                                       |
| 0x6a | is super held down                            |                                                                       |

---

| cmd  | uint8 argument                                |                                                                       |
|------|-----------------------------------------------|-----------------------------------------------------------------------|
| 0x6b | get keybuffer                                 | returns: 0 if empty, keycode of first in keybuffer if not empty       |

---

| cmd  | uint8 argument                                |                                                                       |
|------|-----------------------------------------------|-----------------------------------------------------------------------|
| 0x6c | get mouse x coordinate                        |                                                                       |
| 0x6d | get mouse y coordinate                        |                                                                       |
| 0x6e | get mouse buttons                             | returns: 0 for none, 1 for left, 2 for right and 3 for middle         |

---

| cmd  | uint8 argument                                |                                                                       |
|------|-----------------------------------------------|-----------------------------------------------------------------------|
| 0x6f | joy button                                    | uint8 argument: button ID, returns: 1 for pressed               |

A channel must be set up for receiving the uint16 values that are returned by these functions.

### Audio

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x70 | set audio channel                             | select channel 0-3 for audio operations |
| 0x71 | set frequency                                 | set frequency (0-255, maps to Hz range) |
| 0x72 | set frequency high byte                       | extend frequency range                   |
| 0x73 | set volume                                    | set volume (0-255)                       |
| 0x74 | set waveform                                  | 0=square, 1=triangle, 2=sawtooth, 3=noise |
| 0x75 | play tone                                     | play tone for N frames                   |
| 0x76 | stop channel                                  | stop audio on selected channel          |
| 0x77 | set envelope attack                           | attack time (0-255)                     |
| 0x78 | set envelope decay                            | decay time (0-255)                      |
| 0x79 | set envelope sustain                          | sustain level (0-255)                   |
| 0x7a | set envelope release                          | release time (0-255)                    |
| 0x7b | play sample                                   | play predefined sample ID               |

### Text Rendering

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x80 | set font                                      | select font 0-7                         |
| 0x81 | set text color                                | color for text foreground               |
| 0x82 | set text background color                     | color for text background (255=transparent) |
| 0x83 | set text x position                           | x position for text                      |
| 0x84 | set text y position                           | y position for text                      |
| 0x85 | add to text x position                        | for positions > 255                     |
| 0x86 | add to text y position                        | for positions > 255                     |
| 0x87 | print character                               | print ASCII character                    |
| 0x88 | print string                                  | print null-terminated string (follows in stream) |
| 0x89 | set text scale                                | scale factor 1-8                        |

### Geometric Primitives

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x90 | set circle center x                           | x coordinate for circle center           |
| 0x91 | set circle center y                           | y coordinate for circle center           |
| 0x92 | add to circle center x                        | for coordinates > 255                    |
| 0x93 | add to circle center y                        | for coordinates > 255                    |
| 0x94 | set circle radius                             | radius in pixels                         |
| 0x95 | draw circle                                   | 0=outline, 1=filled                     |
| 0x96 | set rectangle x                               | top-left x coordinate                    |
| 0x97 | set rectangle y                               | top-left y coordinate                    |
| 0x98 | add to rectangle x                            | for coordinates > 255                    |
| 0x99 | add to rectangle y                            | for coordinates > 255                    |
| 0x9a | set rectangle width                           | width in pixels                          |
| 0x9b | set rectangle height                          | height in pixels                         |
| 0x9c | draw rectangle                                | 0=outline, 1=filled                     |

### Timing and Synchronization

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0xa0 | wait frames                                   | wait N frames (60fps assumed)           |
| 0xa1 | set frame rate                                | set target FPS (0=unlimited)            |
| 0xa2 | get frame counter                             | returns current frame number (uint16)   |
| 0xa3 | reset frame counter                           | reset frame counter to 0                |
| 0xa4 | sync to vblank                                | wait for vertical blank                  |

### Advanced Sprite Operations

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0xb0 | set sprite rotation                           | rotation angle (0-255 = 0-360°)         |
| 0xb1 | set sprite scale x                            | horizontal scale (128=1.0x)             |
| 0xb2 | set sprite scale y                            | vertical scale (128=1.0x)               |
| 0xb3 | set sprite flip                               | 0=none, 1=horizontal, 2=vertical, 3=both |
| 0xb4 | check sprite collision                        | check if two sprites collide (returns uint16) |
| 0xb5 | set sprite layer                              | drawing layer/depth (0-255)             |
| 0xb6 | set sprite alpha                              | transparency (0=transparent, 255=opaque) |

### Memory and Data

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0xc0 | save palette                                  | save current palette to slot N           |
| 0xc1 | load palette                                  | load palette from slot N                |
| 0xc2 | save screen                                   | save current screen to buffer N         |
| 0xc3 | load screen                                   | load screen from buffer N               |
| 0xc4 | copy region                                   | copy rectangular region (params follow) |

### Program Control

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0xfe | toggle fullscreen                             | enable or disable fullscreen mode        |
| 0xff | exit                                          | end the program                          |
