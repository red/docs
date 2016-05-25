# Draw dialect

* [Abstract](#abstract)
* [Commands](#commands)
 * [Line](#line)
 * [Triangle](#triangle)
 * [Box](#box)
 * [Polygon](#polygon)
 * [Circle](#circle)
 * [Ellipse](#ellipse)
 * [Arc](#arc)
 * [Curve](#curve)
 * [Spline](#spline)
 * [Image](#image)
 * [Text](#text)
 * [Font](#font)
 * [Pen](#pen)
 * [Fill-pen](#fill-pen)
 * [Line-width](#line-width)
 * [Line-join](#line-join)
 * [Line-cap](#line-cap)
 * [Anti-alias](#anti-alias)
* [Default values](#default-values)
* [Sub blocks](#sub-blocks)
* [Source position](#source-position)
* [Draw function](#draw-function)
* [Graphics source code](#graphics-source-code)

# Abstract

Draw is a dialect (DSL) of Red language that provides a simple declarative way to specify 2D drawing operations. Such operations are expressed as lists of ordered commands (using blocks of values), which can be freely constructed and changed at run-time.

Draw blocks can be rendered directly on top of an image using the `draw` function, or inside a View face using the `draw` facet (see [View documentation](View.md)).

# Commands

Commands can be drawing instructions or settings for the drawing instructions. When a mode is set, it will affect all subsequent operations in the current Draw session (or until changed).

Most drawing commands require coordinates to be specified. The 2D coordinate system used is:
* x axis: increasing from left to right of the display.
* y axis: increasing from top to bottom of the display.

Some drawing commands require lengths to be specified. The length required is number of pixels.

![](images/coord-system.png)

List of Draw commands:

## Line

**Syntax**

    line <point> <point> ...
    
    <point> : coordinates of a point (pair!).
    
**Description**

Draws a line between two points. If more points are specified, additional lines are drawn, connecting each point in the provided order.

## Triangle

**Syntax**

    triangle <point> <point> <point>
    
    <point> : coordinates of a vertex of the triangle (pair!).

_(Note: A vertex is the point at which two lines meet. They are points where edges join.)_ 
    
**Description**

Draws a triangle with edges between the supplied vertices.

## Box

**Syntax**

    box <top-left> <bottom-right>
    box <top-left> <bottom-right> <corner>
    
    <top-left>     : coordinates of the top-left of the box (pair!).
    <bottom-right> : coordinates of the bottom-right of the box (pair!).
    <corner>       : (optional) radius of the arc used to draw a round corner (integer!).
    
**Description**

Draws a box using the top-left (first argument) and bottom-right (second argument) vertices. An optional radius can be provided for making round corners.

## Polygon

**Syntax**

    polygon <point> <point> ...
    
    <point> : coordinates of a vertex (pair!).
    
**Description**

Draws a polygon using the provided vertices. The last point does not need to be the starting point, an extra line will be drawn anyway to close the polygon. Minimal number of points to be provided is 3.

## Circle

**Syntax**

    circle <center> <radius>
    circle <center> <radius-x> <radius-y>
    
    <center>   : coordinates of the circle's center (pair!).
    <radius>   : radius of the circle (integer!).
    <radius-x> : (ellipse mode) radius of the circle along X axis (integer!).
    <radius-y> : (ellipse mode) radius of the circle along Y axis (integer!).
    
**Description**

Draws a circle from the provided center and radius values. The circle can be distorted to form an ellipse by adding an optional integer indicating the radius along Y axis (the other radius argument then becomes the radius along X).

## Ellipse

**Syntax**

    ellipse <top-left> <size>
    
    <top-left> : coordinates of the ellipse's bounding box top-left point (pair!).
    <size>     : size of the bounding box (pair!).
    
**Description**

Draws an ellipse from the specified bounding box. The `size` argument represents the X and Y diameters of the ellipse.

_Note_: `ellipse` provide a more compact and box-oriented way to specify a circle/ellipse compared to `circle` command.

## Arc

**Syntax**

    arc <center> <radius> <begin> <end>
    arc <center> <radius> <begin> <end> closed
    
    <center> : coordinates of the circle's center (pair!).
    <radius> : radius of the circle (pair!).
    <begin>  : starting angle in degrees (integer!).
    <end>  : ending angle in degrees (integer!).
    
**Description**

Draws the arc of a circle from the provided center and radius values. The arc is defined by two angles values. An optional `closed` keyword can be used to draw a closed arc using two lines coming from the center point.

## Curve

**Syntax**

    curve <end-A> <control-A> <end-B>
    curve <end-A> <control-A> <control-B> <end-B>
    
    <end-A>     : end point A (pair!).
    <control-A> : control point A (pair!).
    <control-B> : control point B (pair!).
    <end-B>     : end point B (pair!).

**Description**

Draws a Bezier curve from 3 or 4 points:
* 3 points: 2 end points, 1 control point.
* 4 points: 2 end points, 2 control points.

Four points allow more complex curves to be created.

## Spline

**Syntax**

    spline <point> <point> ...
    spline <point> <point> ... closed
    
    <point> : a control point (pair!).

**Description**

Draws a B-Spline curve from a sequence of points. At least 3 points are required to produce a spline. The optional `closed` keyword will draw an extra segment from the end point to the start point, in order to close the spline.

_Note_: 2 points are accepted, but they will produce only a straight line.

## Image

**Syntax**

    image <image>
    image <image> <top-left>
    image <image> <top-left> <bottom-right>
    image <image> <top-left> <top-right> <bottom-left> <bottom-right>
    image <image> <top-left> <top-right> <bottom-left> <bottom-right> <color>
    image <image> <top-left> <top-right> <bottom-left> <bottom-right> <color> border
    
    <image>        : image to display (image! word!).
    <top-left>     : (optional) coordinate of top left edge of the image (pair!).
    <top-right>    : (optional) coordinate of top right edge of the image (pair!).
    <bottom-left>  : (optional) coordinate of bottom left edge of the image (pair!).
    <bottom-right> : (optional) coordinate of bottom right edge of the image (pair!).
    <color>        : (optional) key color to be made transparent (tuple! word!).
    
**Description**

Paints an image using the provided information for position and width. If the image has positioning information provided, then the image is painted at 0x0 coordinates. A color value can be optionally provided, it will be used for transparency. 

_Note_: 
* Four points mode is not yet implemented. It will allow to stretch the image using 4 arbitrary-positioned edges.
* `border` optional mode is not yet implemented.

## Text

**Syntax**

    text <position> <string>
    
    <position> : coordinates where the string is printed (pair!).
    <string>   : text to print (string!).

**Description**

Prints a text string at the provided coordinates using the current font. 

_Note_: If no font is selected or if the font color is set to `none`, then the pen color is used instead.

## Font

**Syntax**

    font <font>
    
    <font> : new font object to use (object! word!).

**Description**

Selects the font to be used for text printing. The font object is a clone of `font!`.

## Pen

**Syntax**

    pen <color>
    
    <color> : new color to use for drawing (tuple! word!).

**Description**

Selects the color to be used for line drawing operations.

## Fill-pen

**Syntax**

    fill-pen <color>
    fill-pen off
    
    <color> : new color to use for filling (tuple! word!).

**Description**

Selects the color to be used for filling operations. All closed shapes will get filled by the selected color until the fill pen is set to `off`.

## Line-width

**Syntax**

    line-width <value>
    
    <value> : new line width in pixels (integer!).

**Description**

Sets the new width for line operations.

## Line-join

**Syntax**

    line-join <mode>
    
    <mode> : new line joining mode (word!).

**Description**

Sets the new line joining mode for line operations. Following values are accepted:
* `miter` (default)
* `round`
* `bevel`
* `miter-bevel`

![](images/line-join.png)

_Note_: `miter-bevel` mode selects automatically one or the other joining mode depending on the miter length (See [this](https://msdn.microsoft.com/en-us/library/windows/desktop/ms534148%28v=vs.85%29.aspx) for detailed explanation).

## Line-cap

**Syntax**

    line-cap <mode>
    
    <mode> : new line cap mode (word!).

**Description**

Sets the new line's ending cap mode for line operations. Following values are accepted:
* `flat` (default)
* `square`
* `round`

![](images/line-cap.png)

## Anti-alias

**Syntax**

    anti-alias <mode>
    
    <mode> : `on` to enable it or `off` to disable it.
    
**Description**

Turns on/off the anti-aliasing mode for following Draw commands.

_Note_: Anti-aliasing gives nicer visual rendering, but degrades performance.

# Default values

When a new Draw session starts, the following default values are used:

Property | Value
----- | --------
background	| `white`
pen color	| `black`
filling		| `off`
anti-alias	| `on`
font		| `none`
line width	| `1`
line join	| `miter`
line cap	| `flat`

# Sub blocks

Inside Draw code, commands can be arbitrarily grouped using blocks. Semantics remain unchanged, this is currently just a syntactic sugar allowing easier group manipulations of commands (notably group extraction/insertion/removal). Empty blocks are accepted.

# Source position

Set-words can be used in the Draw code **in-between** commands to record the current position in Draw block and be able to easily access it later.

_Note_: if the Draw block length preceeding a set-word is changed, the recorded position is not updated.

# Draw function

It is possible to render a Draw block directly to an image using the `draw` function.

**Syntax**

    draw <size> <spec>
    draw <image> <spec>
    
    <size>  : size of a new image (pair!).
    <image> : image to use as canvas (image!).
    <spec>  : block of Draw commands (block!).

**Description**

Renders the provided Draw commands to an existing or a new image. The image value is returned by the function.

# Graphics source code

The graphics in this documentation are generated using Red and Draw dialect, here is the source code (you can copy/paste it in a Red console to try/play/improve it):

	Red [
		Title:	"Graphics generator for Draw documentation"
		Author: "Nenad Rakocevic"
		File:   %draw-graphics.red
		Needs:	View
	]

	Arial: make font! [name: "Consolas" style: 'bold]
	small: make font! [size: 9 name: "Consolas" style: 'bold]

	save %line-cap.png draw 240x240 [
		font Arial
		text 20x220  "Flat"
		text 90x220  "Square"
		text 180x220 "Round"

		line-width 20 pen gray
		line-cap flat	line 40x40  40x200
		line-cap square line 120x40 120x200
		line-cap round	line 200x40 200x200

		line-width 1 pen black
		line 20x40  220x40
		line 20x200 220x200
	]

	save %line-join.png draw 500x100 [
		font Arial
		text 10x20  "Miter"
		text 170x20 "Round"
		text 330x20 "Bevel"

		line-width 20 pen gray
		line-join miter line 140x20 40x80  140x80
		line-join round line 300x20 200x80 300x80
		line-join bevel line 460x20 360x80 460x80

		line-join miter
		line-width 1 pen black
		line 140x20 40x80  140x80
		line 300x20 200x80 300x80
		line 460x20 360x80 460x80
	]

	save %coord-system.png draw 240x240 [
		font small
		text 5x5 "0x0"
		line-width 2
		line 20x20 200x20 195x16
		line 200x20 195x24

		line 20x20 20x200 16x195
		line 20x200 24x195

		font Arial
		text 205x12 "X"
		text 12x205 "Y"
	]
