# PiPro

PixelProtocol (pipro) is a binary protocol for defining what is being sent between the GUI client and the game engine.

It's for implementing games where old-school looking pixel art can be appreciated.

## Features and limitations

* 256 indexed colors.
* A size of 320x200 pixels is recommended.
* Should be possible to implement both in 16-bit assembly for DOS and in a browser.

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

| cmd  | uint8 argument                                |                                                |
|------|-----------------------------------------------|------------------------------------------------|
| 0x0d | flip                                          | update all pixels                              |
| 0x0e | sprite flip                                   | update pixels where sprites have been drawn    |

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

---

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x1d | set x coordinate for p0                       | prepare to draw a filled triangle        |
| 0x1e | set y coordinate for p0                       | prepare to draw a filled triangle        |
| 0x1f | set x coordinate for p1                       | prepare to draw a filled triangle        |
| 0x20 | set y coordinate for p1                       | prepare to draw a filled triangle        |
| 0x21 | set x coordinate for p2                       | prepare to draw a filled triangle        |
| 0x22 | set y coordinate for p2                       | prepare to draw a filled triangle        |

---

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x23 | add to x coordinate for p0                    | prepare to draw a filled triangle        |
| 0x24 | add to y coordinate for p0                    | prepare to draw a filled triangle        |
| 0x25 | add to x coordinate for p1                    | prepare to draw a filled triangle        |
| 0x26 | add to y coordinate for p1                    | prepare to draw a filled triangle        |
| 0x27 | add to x coordinate for p2                    | prepare to draw a filled triangle        |
| 0x28 | add to y coordinate for p2                    | prepare to draw a filled triangle        |

---

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x29 | draw a filled or empty triangle               | 0 for empty, 1 for filled                |

### Randomization

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x2a | choose a random color for the pixel           |                                          |
| 0x2b | choose random colors for the line             |                                          |
| 0x2c | choose random colors for the triangle         |                                          |

---

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
| 0x3b | copy                                          | copy to another sprite ID                |
| 0x3c | rotate                                        | rotate the current sprite, 0..255        |

### Convolution Filters

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x40 | set convolution filter 0                      ||
| 0x41 | set convolution filter 1                      ||
| 0x42 | set convolution filter 2                      |                                          |
| 0x43 | set convolution filter 3                      ||
| 0x44 | set convolution filter 4                      ||
| 0x45 | set convolution filter 5                      |                                          |
| 0x46 | set convolution filter 6                      |                                          |
| 0x47 | set convolution filter 7                      |                                          |
| 0x48 | set convolution filter 8                      |                                          |
| 0x49 | set convolution division                      |                                          |
| 0x4a | apply convolution filter                      | apply to all pixels                      |
| 0x4b | apply convolution filter to sprite            | apply to current sprite                  |

* blur is 0,1,0,1,1,1,0,1,0 div 5
* flame is 0,1,0,1,1,1,0,0,0 div 4         
* the convolution filter parameters 0..255 are treated as if they were floats from 0 to 1

### Text

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0x50 | add a byte to the current UTF8 rune           |                                          |
| 0x51 | clear the current UTF8 rune                   |                                          |
| 0x52 | fill the current sprite with the current rune | use a "ø" when no glyph is available     |

At a minimum, these glyphs must exist:

    0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ,._:;"'[]{}()-\/|*#@?!☃

Implementations should supply at least a font that works at 8x8 character size.

The snowman is useful for identifying if the protocol can correctly support at least one non-ASCII character.

## Keyboard, Joystick and Mouse

For returning the state of the client:

Comands that return an uint16:

| cmd  | uint8 argument                                |                                                                       |
|------|-----------------------------------------------|-----------------------------------------------------------------------|
| 0x60 |                                       | is Escape being pressed?                                              |
| 0x61 |                                       | is W, up or joystick up pressed? args: 0 for both, 1 for P1, 2 for P2. Player1 has WASD and Joy1, Player2 has arrows and Joy2                |
| 0x62 |                                       | is A, left or joystick left pressed?                                  |
| 0x63 |                                       | is S, down or joystick down pressed?                                  |
| 0x64 |                                       | is D, right or joystick right pressed?                                |
| 0x65 |                                       | is Return, Comma (,) or joystick A pressed?                           |
| 0x66 |                                       | is Space, Dot (.) or joystick B pressed?                              |

---

| cmd  | uint8 argument                                |                                                                       |
|------|-----------------------------------------------|-----------------------------------------------------------------------|
| 0x67 |  0 for left, 1 for right, 2 for any | returns 1 if Shift is held down |
| 0x68 |                                        | returns 1 if Alt is held down
| 0x69 |                                        | returns 1 if Ctrl is held down
| 0x6a |                                        | returns | if Super is held down

---

| cmd  | uint8 argument                                |                                                                       |
|------|-----------------------------------------------|-----------------------------------------------------------------------|
| 0x6b |                                | returns 0 if keybuffer is empty, keycode of first in keybuffer if not empty       |

---

| cmd  | uint8 argument                                |                                                                       |
|------|-----------------------------------------------|-----------------------------------------------------------------------|
| 0x6c |  | get mouse x coordinate                        |                                                                       |
| 0x6d | | get mouse y coordinate                        |                                                                       |
| 0x6e |  | get mouse buttons, returns: 0 for none, 1 for left, 2 for right and 3 for middle         |

---

| cmd  | uint8 argument                                |                                                                       |
|------|-----------------------------------------------|-----------------------------------------------------------------------|
| 0x6f | joystick button ID                                    | check if joystick button is pressed, returns: 1 for pressed               |

A channel must be set up for receiving the uint16 values that are returned by these functions.

### Program Control

| cmd  | uint8 argument                                |                                          |
|------|-----------------------------------------------|------------------------------------------|
| 0xfe | toggle fullscreen                             | enable or disable fullscreen mode        |
| 0xff | exit                                          | end the program                          |
