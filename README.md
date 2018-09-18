<!--
title: PiPro
favicon: spaceinvaders.ico
-->

# PiPro

PixelProtocol (pipro) is a binary protocol for defining what is being sent between a GUI client and a game engine.

It's for implementing games where old-school looking pixel art can be appreciated.

## Features and limitations

* 256 indexed colors.
* A resolution of 320x200 pixels is recommended.
* Should be possible to implement both in 16-bit assembly for DOS and in a modern browser.
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

| Cmd    | Name         | **uint8** argument                      | Description                                                    |
|--------|--------------|-----------------------------------------|----------------------------------------------------------------|
| `0x00` | `palsel`     | color index, 0..255                     | choose palette color, prepare for filling the palette          |
| `0x01` | `setred`     | red value, 0..255                       | set red value of chosen palette color                          |
| `0x02` | `setgreen`   | set green value of chosen palette color | set blue value of chosen palette color                         |
| `0x03` | `setblue`    | set blue value of chosen palette color  | set green value of chosen palette color                        |

### Drawing pixels

| Cmd    | Name     | **uint8** argument                      | Description                              |
|--------|----------|-----------------------------------------|------------------------------------------|
| `0x04` | `setcol` | pixel color from palette, 0..255        | set the active color                     |

---

| Cmd    | Name     | **uint16** argument                     | Description                              |
|--------|----------|-----------------------------------------|------------------------------------------|
| `0x05` | `setx`   | x position, 0..65535                    | set active x position                    |
| `0x06` | `sety`   | y position, 0..65535                    | set active y position                    |

---

| Cmd    | Name     | **uint8** argument                      | Description                              |
|--------|----------|-----------------------------------------|------------------------------------------|
| `0x07` | `plot`   | n pixels                                | plot one or more pixels from (x,y)       |

### Filling

| Cmd    | Name    | **uint8** argument                      | Description                                                                       |
|--------|---------|-----------------------------------------|-----------------------------------------------------------------------------------|
| `0x08` | `clear` | fill color from palette, 0..255         | clear everything with the selected color                                          |
| `0x09` | `rfill` | fill color from palette, 0..255         | draw linewise until nonzero color or end, for filling the pixel buffer            |
| `0x0a` | `lfill` | fill color from palette, 0..255         | draw backwards linewise until nonzero color or end, for filling the pixel buffer  |

### Flipping

| Cmd    | Name         |  no argument | Description                                                                            |
|--------|--------------|--------------|----------------------------------------------------------------------------------------|
| `0x0b` | `flip`       |              | update all pixels                                                                      |
| `0x0c` | `spriteflip` |              | update pixels where sprites have been drawn since last time this command was executed  |

### Drawing lines

| Cmd    | Name     | **uint8** argument               | Description                     |
|--------|----------|----------------------------------|---------------------------------|
| `0x0d` | `setlcs` | color for start of line          | prepare to draw a line          |
| `0x0e` | `setlce` | color for end of line            | prepare to draw a line          |

---

| Cmd    | Name   | **uint16** argument                     | Description                    |
|--------|--------|-----------------------------------------|--------------------------------|
| `0x11` | `lisx` | x coordinate for start of line          | prepare to draw a line         |
| `0x12` | `lisy` | y coordinate for start of line          | prepare to draw a line         |
| `0x13` | `liex` | x coordinate for end of line            | prepare to draw a line         |
| `0x14` | `liey` | y coordinate for end of line            | prepare to draw a line         |

---

| Cmd    | Name    | no argument    | Description    |
|--------|---------|----------------|----------------|
| `0x15` | `ldraw` |                | draw the line  |


### Drawing triangles

| Cmd    | Name     | **uint8** argument  | Description                   |
|--------|----------|---------------------|-------------------------------|
| `0x1a` | `setcp0` | set color for p0    | prepare to draw a triangle    |
| `0x1b` | `setcp1` | set color for p1    | prepare to draw a triangle    |
| `0x1c` | `setcp2` | set color for p2    | prepare to draw a triangle    |

---

| Cmd    | Name     | **uint16** argument       | Description                    |
|--------|----------|--------------------------|--------------------------------|
| `0x1d` | `setxp0` | x coordinate for p0      | prepare to draw a triangle     |
| `0x1e` | `setyp0` | y coordinate for p0      | prepare to draw a triangle     |
| `0x1f` | `setxp1` | x coordinate for p1      | prepare to draw a triangle     |
| `0x20` | `setyp1` | y coordinate for p1      | prepare to draw a triangle     |
| `0x21` | `setxp2` | x coordinate for p2      | prepare to draw a triangle     |
| `0x22` | `setyp2` | y coordinate for p2      | prepare to draw a triangle     |

---

| Cmd    | Name    | **uint8** argument                    | Description                              |
|--------|---------|---------------------------------------|------------------------------------------|
| `0x23` | `tdraw` | 0 for empty, 1 for filled             | draw a filled or empty triangle          |

### Sprites

| Cmd    | Name       | **uint8** argument                                | Description                                                           |
|--------|------------|---------------------------------------------------|-----------------------------------------------------------------------|
| `0x30` | `spid`     | sprite ID                                         | select a sprite ID to work with                                       |
| `0x31` | `spw`      | sprite width                                      | set sprite width                                                      |
| `0x32` | `sph`      | sprite height                                     | set sprite height                                                     |
| `0x33` | `spclr`    | color                                             | clear contents with the given color                                   |
| `0x34` | `spush`    | amount of pixels                                  | add N pixels of the active color                                      |
| `0x35` | `spt`      | amount of pixels                                  | add N transparent pixels                                              |
| `0x36` | `sprot`    | value from 0..255, used as float from `0..2*PI`   | rotate the current sprite                                             |
| `0x37` | `spscale`  | value from 0..255, used as float from -20..20     | scale the current sprite                                              |
| `0x38` | `spcopy`   | sprite ID                                         | copy to another sprite ID                                             |

---

| Cmd    | Name       | **uint16** argument    | Description                                       |
|--------|------------|------------------------|---------------------------------------------------|
| `0x39` | `spx`      | x coordinate           | set x coordinate for where to draw the sprite     |
| `0x3a` | `spy`      | y coordinate           | set y coordinate for where to draw the sprite     |

---

| Cmd    | Name       | **uint8** argument           | Description                                                           |
|--------|------------|------------------------------|-----------------------------------------------------------------------|
| `0x3b` | `blit`     | number of sprites to draw    | draw n instances of this sprite, following the pixel buffer direction |
| `0x3c` | `blitinc`  | number of sprites to draw    | like `blit`, but increases the sprite ID at every step                |

* The "pixel buffer direction" is from left to right, then starting on the next y coordinate (+ sprite height) when reaching the end of the line.
* Several sprites can be placed in a row with the `blit` command. They are placed next to each other, without overlapping.
* For the `blitinc` command, increasing the value from 255 wraps around and sets to current sprite ID to 0.
* Sprites can have a resolution up to `128*128` (inclusive).

### Convolution Filters

| Cmd    | Name     | **uint8** argument                                        | Description                                    |
|--------|----------|-----------------------------------------------------------|------------------------------------------------|
| `0x40` | `con0`   | value from 0..255, used as float from -20..20             | set convolution filter value 0                 |
| `0x41` | `con1`   | value from 0..255, used as float from -20..20             | set convolution filter value 1                 |
| `0x42` | `con2`   | value from 0..255, used as float from -20..20             | set convolution filter value 2                 |
| `0x43` | `con3`   | value from 0..255, used as float from -20..20             | set convolution filter value 3                 |
| `0x44` | `con4`   | value from 0..255, used as float from -20..20             | set convolution filter value 4                 |
| `0x45` | `con5`   | value from 0..255, used as float from -20..20             | set convolution filter value 5                 |
| `0x46` | `con6`   | value from 0..255, used as float from -20..20             | set convolution filter value 6                 |
| `0x47` | `con7`   | value from 0..255, used as float from -20..20             | set convolution filter value 7                 |
| `0x48` | `con8`   | value from 0..255, used as float from -20..20             | set convolution filter value 8                 |
| `0x49` | `condiv` | convolution division, 0..255, used as float from -20..20  | set convolution division value                 |
| `0x4a` | `apply`  |                                                           | apply convolution filter to all pixels         |
| `0x4b` | `consp`  |                                                           | apply convolution filter to the current sprite |

The convolution filter parameters 0..255 are treated as if they were floats between -20 and 20 (inclusive).

Example filters:

* blur is 0,1,0,1,1,1,0,1,0 div 5
* flame is 0,1,0,1,1,1,0,0,0 div 4

### Text

| Cmd    | Name      | **uint8** argument | Description                                 |
|--------|-----------|--------------------|-----------------------------------------------|
| `0x50` | `radd`    | utf-8 byte         | add a byte to the current UTF-8 rune          |

| Cmd    | Name      | no argument | Description                                 |
|--------|-----------|-------------|-----------------------------------------------|
| `0x51` | `rclear`  |             | clear the current UTF-8 rune                  |
| `0x52` | `rsprite` |             | fill the current sprite with the current rune |

At a minimum, these glyphs must exist:

    0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ,._:;"'[]{}()-\/|*#@?!☃

Implementations should supply at least a font that works at 8x8 character size.

The snowman is useful for identifying if the protocol can correctly support at least one non-ASCII character.

Use an `ø` if no glyph is available for an UTF-8 rune.

## Keyboard, Joystick and Mouse

For returning the state of the client.

A channel must be set up for receiving the`uint16`values that are returned by these functions.

Comands that return an `uint16`:

| Cmd    | Name     |  no argument | Description                                                             |
|--------|----------|--------------|-------------------------------------------------------------------------|
| `0x60` | `kesc`   |              | is Escape being pressed?                                                |
| `0x61` | `kup`    |              | is W, up or joystick up pressed? args: 0 for any, 1..4 for Player 1..4  |
| `0x62` | `kleft`  |              | is A, left or joystick left pressed? args: 0..5                         |
| `0x63` | `kdown`  |              | is S, down or joystick down pressed? args: 0..5                         |
| `0x64` | `kright` |              | is D, right or joystick right pressed? args: 0..5                       |
| `0x65` | `ka`     |              | is Return, comma (,) or joystick A button pressed? args: 0..5           |
| `0x66` | `kb`     |              | is Space, dot (.) or joystick B button pressed? args: 0..5              |

* P1 means Player 1, P2 means Player 2.
* Player 1 has WASD keys, lshift, lctrl and/or Joystick 1.
* Player 2 has the arrow keys, comma (,), dot (.) and/or Joystick 2.
* Player 3 has the numpad arrows and/or Joystick 3.
* Player 4 has Joystick 4.

---

| Cmd    | Name     | **uint8** argument                    | Description                        |
|--------|----------|---------------------------------------|------------------------------------|
| `0x67` | `kshift` | 0 for left, 1 for right, 2 for any    | returns 1 if Shift is held down    |
| `0x68` | `kalt`   | 0 for left, 1 for right, 2 for any    | returns 1 if Alt is held down      |
| `0x69` | `kctrl`  | 0 for left, 1 for right, 2 for any    | returns 1 if Ctrl is held down     |
| `0x6a` | `ksuper` | 0 for left, 1 for right, 2 for any    | returns 1 if Super is held down    |

---

| Cmd    | Name   | no argument | Description                                                                  |
|--------|--------|-------------|------------------------------------------------------------------------------|
| `0x6b` | `kget` |             | returns 0 if keybuffer is empty, keycode of first in keybuffer if not empty  |

---

| Cmd    | Name   | no argument | Description                                                                       |
|--------|--------|-------------|-----------------------------------------------------------------------------------|
| `0x6c` | `mx`   |             | get mouse x coordinate                                                            |
| `0x6d` | `my`   |             | get mouse y coordinate                                                            |
| `0x6e` | `mbtn` |             | get mouse buttons, returns: 0 for none, 1 for left, 2 for right and 3 for middle  |

---

| Cmd    | Name   | **uint8** argument | Description                                                       |
|--------|--------|--------------------|-------------------------------------------------------------------|
| `0x6f` | `jbtn` | joystick button ID | check if joystick button is pressed, returns 1 for pressed        |

### Program Control

| Cmd    | Name     | no argument | Description                 |
|--------|----------|-------------|-----------------------------|
| `0xff` | `exit`   |             | end the program, disconnect |

### Client Side State

| Description                             | Type              |
|-----------------------------------------|-------------------|
| r, g, b, a * 256 palette info           | 4 * 256 * uint8   |
| x position for pixel                    | uint16            |
| y position for pixel                    | uint16            |
| color index for pixel                   | uint16            |
| x1 position for line                    | uint16            |
| y1 position for line                    | uint16            |
| x2 position for line                    | uint16            |
| y2 position for line                    | uint16            |
| c1 color for line start                 | uint8             |
| c2 color for line end                   | uint8             |
| x1 position for triangle                | uint16            |
| y1 position for triangle                | uint16            |
| x2 position for triangle                | uint16            |
| y2 position for triangle                | uint16            |
| x3 position for triangle                | uint16            |
| y3 position for triangle                | uint16            |
| c1 color for triangle point 1           | uint8             |
| c2 color for triangle point 2           | uint8             |
| c3 color for triangle point 3           | uint8             |
| convolution filter + division, 10 bytes | 10 * uint8        |
| sprites                                 | 128 * 128 * uint8 |
| current sprite ID                       | uint8             |

### List of client implementations

* TBA

### List of server implementations

* TBA

### General info

* Version: 2.0.0
* Author: Alexander F. Rødseth
* [GitHub Project](https://github.com/xyproto/pixelprotocol)
