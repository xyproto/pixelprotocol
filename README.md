<!--
title: PiPro
favicon: spaceinvaders.ico
-->

# PiPro

PixelProtocol (pipro) is a binary protocol for defining what is being sent between a GUI client and a game engine.

It's for implementing games where old-school looking pixel art can be appreciated.

## Features and limitations

* 256 indexed colors.
* A size of 320x200 pixels is recommended.
* Should be possible to implement both in 16-bit assembly for DOS and in a browser.
* Aiming to be WebSocket-friendly.
* Pixels are not sent over the network, only commands for drawing them.
* It should be possible to create a DosBox server for serving games over this protocol.
* It should be possible to create a mobile client for playing games over this protocol.

## Q&A

**Q**: Wouldn't it be cooler if Vulkan commands was sent instead? Or OpenGL? Or SDL2?<br>
**A**: Protocols for OpenGL over network already exists and I want to keep things really simple.

**Q**: Can't you just use VNC?<br>
**A**: No, I want something specifically for games or demoscene demos that use 320x200 pixels, 256 colors.

# Protocol Definition

## Protocol Header

| Name       | Type              | Description                                         |
|------------|-------------------|-----------------------------------------------------|
| `ver`      |`uint16`           | protocol version                                    |
| `width`    |`uint16`           | width                                               |
| `height`   |`uint16`           | height                                              |
| `commands` | `[]uint16`        | list of commands (`uint8` cmd + `uint8` argument)   |

The commands can be streamed.

## Commands

### Color palette

| Cmd  | Name         | `uint8` argument                        | Description                                                    |
|------|--------------|-----------------------------------------|----------------------------------------------------------------|
| 0x00 | `palsel`     | color index, 0..255                     | choose palette color, prepare for filling the palette          |
| 0x01 | `setred`     | red value, 0..255                       | set red value of chosen palette color                          |
| 0x02 | `setgreen`   | set green value of chosen palette color | set blue value of chosen palette color                         |
| 0x03 | `setblue`    | set blue value of chosen palette color  | set green value of chosen palette color                        |

### Drawing pixels

| Cmd  | Name     | `uint8` argument                        | Description                              |
|------|----------|-----------------------------------------|------------------------------------------|
| 0x04 | `setcol` | pixel color from palette, 0..255        | set the active color                     |
| 0x05 | `setx`   | x position, 0..255                      | set active x position                    |
| 0x06 | `sety`   | y position, 0..255                      | set active y position                    |
| 0x07 | `addx`   | x value, 0..255                         | add value to x position                  |
| 0x08 | `addy`   | y value, 0..255                         | add value to y position                  |
| 0x09 | `plot`   | n pixels                                | plot one or more pixels from (x,y)       |

##### Q&A

**Q**: What is the "add to x position" command for?<br>
**A**: since all arguments are bytes, it's needed to be able to specify X coordinates from 256..320.

**Q**: Isn't that a bit impractical?<br>
**A**: Perhaps, but it makes the protocol very simple and uniform. All commands takes a byte as an argument.

### Filling

| Cmd  | Name    | `uint8` argument                        | Description                                                                       |
|------|---------|-----------------------------------------|-----------------------------------------------------------------------------------|
| 0x0a | `clear` | fill color from palette, 0..255         | clear everything with the selected color                                          |
| 0x0b | `rfill` | fill color from palette, 0..255         | draw linewise until nonzero color or end, for filling the pixel buffer            |
| 0x0c | `lfill` | fill color from palette, 0..255         | draw backwards linewise until nonzero color or end, for filling the pixel buffer  |

### Flipping

| Cmd  | Name         |  `uint8` argument | Description                                    |
|------|--------------|-------------------|------------------------------------------------|
| 0x0d | `flip`       |                   | update all pixels                              |
| 0x0e | `spriteflip` |                   | update pixels where sprites have been drawn    |

### Drawing lines

| Cmd  | Name     | `uint8` argument                 | Description                              |
|------|----------|----------------------------------|------------------------------------------|
| 0x0f | `setlcs` | color for start of line          | prepare to draw a line                   |
| 0x10 | `setlce` | color for end of line            | prepare to draw a line                   |

---

| Cmd  | Name   | `uint8` argument                        | Description                              |
|------|--------|-----------------------------------------|------------------------------------------|
| 0x11 | `lisx` | x coordinate for start of line          | prepare to draw a line                   |
| 0x12 | `lisy` | y coordinate for start of line          | prepare to draw a line                   |
| 0x13 | `liex` | x coordinate for end of line            | prepare to draw a line                   |
| 0x14 | `liey` | y coordinate for end of line            | prepare to draw a line                   |

---

| Cmd  | Name    | `uint8` argument                              | Description                              |
|------|---------|-----------------------------------------------|------------------------------------------|
| 0x15 | `liasx` | add to x coordinate for start of line         | prepare to draw a line                   |
| 0x16 | `liasy` | add to y coordinate for start of line         | prepare to draw a line                   |
| 0x17 | `liaex` | add to x coordinate for end of line           | prepare to draw a line                   |
| 0x18 | `liaey` | add to y coordinate for end of line           | prepare to draw a line                   |

---

| Cmd  | Name    |`uint8` argument      | Description    |
|------|---------|----------------------|----------------|
| 0x19 | `ldraw` |                      | draw the line  |


### Drawing triangles

| Cmd  | Name     | `uint8` argument    | Description                   |
|------|----------|---------------------|-------------------------------|
| 0x1a | `setcp0` | set color for p0    | prepare to draw a triangle    |
| 0x1b | `setcp1` | set color for p1    | prepare to draw a triangle    |
| 0x1c | `setcp2` | set color for p2    | prepare to draw a triangle    |

---

| Cmd  | Name     | `uint8` argument         | Description                    |
|------|----------|--------------------------|--------------------------------|
| 0x1d | `setxp0` | x coordinate for p0      | prepare to draw a triangle     |
| 0x1e | `setyp0` | y coordinate for p0      | prepare to draw a triangle     |
| 0x1f | `setxp1` | x coordinate for p1      | prepare to draw a triangle     |
| 0x20 | `setyp1` | y coordinate for p1      | prepare to draw a triangle     |
| 0x21 | `setxp2` | x coordinate for p2      | prepare to draw a triangle     |
| 0x22 | `setyp2` | y coordinate for p2      | prepare to draw a triangle     |

---

| Cmd  | Name     | `uint8` argument                 | Description                   |
|------|----------|----------------------------------|-------------------------------|
| 0x23 | `addxp0` | add to x coordinate for p0       | prepare to draw a triangle    |
| 0x24 | `addyp0` | add to y coordinate for p0       | prepare to draw a triangle    |
| 0x25 | `addxp1` | add to x coordinate for p1       | prepare to draw a triangle    |
| 0x26 | `addyp1` | add to y coordinate for p1       | prepare to draw a triangle    |
| 0x27 | `addxp2` | add to x coordinate for p2       | prepare to draw a triangle    |
| 0x28 | `addyp2` | add to y coordinate for p2       | prepare to draw a triangle    |

---

| Cmd  | Name    | `uint8` argument                      | Description                              |
|------|---------|---------------------------------------|------------------------------------------|
| 0x29 | `tdraw` | 0 for empty, 1 for filled             | draw a filled or empty triangle          |

### Randomization

| Cmd  | Name   | `uint8` argument  | Description                           |
|------|--------|-------------------|---------------------------------------|
| 0x2a | `ranc` |                   | set a random color for the pixel      |
| 0x2b | `ranl` |                   | set random colors for the line        |
| 0x2c | `rant` |                   | set random colors for the triangle    |

---

| Cmd  | Name    | `uint8` argument | Description                             |
|------|---------|------------------|-----------------------------------------|
| 0x2d | `ranp`  |                  | set random coordinates for the pixel    |
| 0x2e | `ranlp` |                  | set random coordinates for the line     |
| 0x2f | `rantp` |                  | set random coordinates for the triangle |

### Sprites

| Cmd  | Name     | `uint8` argument                              | Description                              |
|------|----------|-----------------------------------------------|------------------------------------------|
| 0x30 | `spid`   | choose sprite ID                              | select a sprite to work with             |
| 0x31 | `spw`    | set sprite width                              | set sprite width                         |
| 0x32 | `sph`    | set sprite height                             | set sprite height                        |
| 0x33 | `spclr`  | clear sprite                                  | clear contents                           |
| 0x34 | `spush`  | push pixel                                    | adds N pixels of the selected color      |
| 0x35 | `spnil`  | push empty                                    | add N transparent pixels                 |
| 0x36 | `spx`    | set x coordinate for drawing the sprite       |                                          |
| 0x37 | `spy`    | set y coordinate for drawing the sprite       |                                          |
| 0x38 | `spax`   | add to x coordinate for drawing the sprite    |                                          |
| 0x39 | `spay`   | add to y coordinate for drawing the sprite    |                                          |
| 0x3a | `spdraw` | draw sprite                                   | draw the current sprite, 2 for dbl. size |
| 0x3b | `spcopy` | copy                                          | copy to another sprite ID                |
| 0x3c | `sprot`  | rotate                                        | rotate the current sprite, 0..255        |

### Convolution Filters

| Cmd  | Name     | `uint8` argument                              | Description                                    |
|------|----------|-----------------------------------------------|------------------------------------------------|
| 0x40 | `con0`   | value from 0..255, used as float              | set convolution filter value 0                 |
| 0x40 | `con1`   | value from 0..255, used as float              | set convolution filter value 1                 |
| 0x40 | `con2`   | value from 0..255, used as float              | set convolution filter value 2                 |
| 0x40 | `con3`   | value from 0..255, used as float              | set convolution filter value 3                 |
| 0x40 | `con4`   | value from 0..255, used as float              | set convolution filter value 4                 |
| 0x40 | `con5`   | value from 0..255, used as float              | set convolution filter value 5                 |
| 0x40 | `con6`   | value from 0..255, used as float              | set convolution filter value 6                 |
| 0x40 | `con7`   | value from 0..255, used as float              | set convolution filter value 7                 |
| 0x48 | `con8`   | value from 0..255, used as float              | set convolution filter value 8                 |
| 0x49 | `condiv` | convolution division, 0..255, used as float   | set convolution division value                 |
| 0x4a | `apply`  |                                               | apply convolution filter to all pixels         |
| 0x4b | `consp`  |                                               | apply convolution filter to the current sprite |

The convolution filter parameters 0..255 are treated as if they were floats between 0 and 1 (inclusive).

Example filters:

* blur is 0,1,0,1,1,1,0,1,0 div 5
* flame is 0,1,0,1,1,1,0,0,0 div 4

### Text

| Cmd  | Name      | `uint8` argument | Description                                 |
|------|-----------|------------------|-----------------------------------------------|
| 0x50 | `radd`    |                  | add a byte to the current UTF-8 rune          |
| 0x51 | `rclear`  |                  | clear the current UTF-8 rune                  |
| 0x52 | `rsprite` |                  | fill the current sprite with the current rune |

At a minimum, these glyphs must exist:

    0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ,._:;"'[]{}()-\/|*#@?!☃

Implementations should supply at least a font that works at 8x8 character size.

The snowman is useful for identifying if the protocol can correctly support at least one non-ASCII character.

Use an `ø` if no glyph is available for an UTF-8 rune.

## Keyboard, Joystick and Mouse

For returning the state of the client.

A channel must be set up for receiving the`uint16`values that are returned by these functions.

Comands that return an `uint16`:

| Cmd  | Name     |  `uint8` argument | Description                                                            |
|------|----------|-------------------|------------------------------------------------------------------------|
| 0x60 | `kesc`   |                   | is Escape being pressed?                                               |
| 0x61 | `kup`    |                   | is W, up or joystick up pressed? args: 0 for both, 1 for P1, 2 for P2. |
| 0x62 | `kleft`  |                   | is A, left or joystick left pressed?                                   |
| 0x63 | `kdown`  |                   | is S, down or joystick down pressed?                                   |
| 0x64 | `kright` |                   | is D, right or joystick right pressed?                                 |
| 0x65 | `ka`     |                   | is Return, Comma (,) or joystick A pressed?                            |
| 0x66 | `kb`     |                   | is Space, Dot (.) or joystick B pressed?                               |

P1 means Player 1, P2 means Player 2.
Player 1 has WASD keys and/or Joystick 1
Player 2 has the arrow keys and Joystick 2

---

| Cmd  | Name     | `uint8` argument                      | Description                        |
|------|----------|---------------------------------------|------------------------------------|
| 0x67 | `kshift` | 0 for left, 1 for right, 2 for any    | returns 1 if Shift is held down    |
| 0x68 | `kalt`   |                                       | returns 1 if Alt is held down      |
| 0x69 | `kctrl`  |                                       | returns 1 if Ctrl is held down     |
| 0x6a | `ksuper` |                                       | returns 1 if Super is held down    |

---

| Cmd  | Name   | `uint8` argument | Description                                                                  |
|------|--------|------------------|------------------------------------------------------------------------------|
| 0x6b | `kget` |                  | returns 0 if keybuffer is empty, keycode of first in keybuffer if not empty  |

---

| Cmd  | Name   | `uint8` argument | Description                                                                       |
|------|--------|------------------|-----------------------------------------------------------------------------------|
| 0x6c | `mx`   |                  | get mouse x coordinate                                                            |
| 0x6d | `my`   |                  | get mouse y coordinate                                                            |
| 0x6e | `mbtn` |                  | get mouse buttons, returns: 0 for none, 1 for left, 2 for right and 3 for middle  |

---

| Cmd  | Name   | `uint8` argument    | Description                                                       |
|------|--------|---------------------|-------------------------------------------------------------------|
| 0x6f | `jbtn` | joystick button ID  | check if joystick button is pressed, returns 1 for pressed        |


### Program Control

| Cmd  | Name     | `uint8` argument| Description            |
|------|----------|-----------------|------------------------|
| 0xfe | `fullt`  |                 | toggle fullscreen mode |
| 0xff | `exit`   |                 | end the program        |

### List of client implementations

* TBA

### List of server implementations

* TBA

### General info

* [GitHub project](https://github.com/xyproto/pixelprotocol)
* Version: 0.2
