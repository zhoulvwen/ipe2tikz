ipe2tikz
========

TikZ exporter ipelet: a plugin for [ipe](http://ipe.otfried.org/) that exports **readable** [TikZ](https://sourceforge.net/projects/pgf/) pictures for use in LaTeX documents.

TikZ is an amazing piece of software.  To sketch a complicated shape, though, you really want a GUI, like ipe.  This ipelet is meant to allow you to sketch something in ipe, then export it into readable TikZ code that you can tweak and perhaps use in a larger picture.  It is also quite good at exporting large and complex ipe files (subject to the [limitations](#limitations) below) into standalone LaTeX documents.  Thankfully, ipe is well-suited to generating readable TikZ code, since they both rely on flexible symbolic style mechanisms to specify most drawing parameters.

TODO: picture to code

## Installation

1. Copy `tikz.isy` into `~/.ipe/styles` on Linux and Macs.  On Windows, I believe you have to use the directory containing all of the built-in stylesheets.
2. Copy `tikz.lua` into `~/.ipe/ipelets` on Linux and Macs, and into `$USERPROFILE\Ipelets` on Windows.
3. Copy `tikzlibraryipe.code.tex` somewhere  LaTeX can find it, e.g. the same directory as the LaTeX file you're trying to compile.

## Usage

After installing the ipelet, select *TikZ Export* form the *Ipelets* menu, or use the shortcut `Ctrl-Shift-T`.  A dialog appears, with the following options:
 * *Export complete document:* if this is checked, the exporter makes a document that can be compiled standalone.  That is, it generates a preamble, `\begin{document}...\end{document}` tags, etc.  If you want to `\include` the output into another LaTeX file, un-check this option.  Be sure to set `\usetikzlibrary{arrows.meta,patterns,ipe}` somewhere.
 + *Export stylesheet:* if this is checked, the exporter makes a TikZ version of your included ipe stylesheets (see below).
 + *Export colors:* if this is checked, the exporter generates `\definecolor` commands for all colors defined in your ipe stylesheets.
 + *Output file:* this is where the output goes.

The first thing you have to do, however, is to choose whether you want to primarily use TikZ's styles or ipe's styles.

### Using TikZ's styles

Use TikZ's styles if you want to make a sketch in ipe, but do most of the work in TikZ.  In this case, you want ipe's styles to reflect TikZ's styles as closely as possible: for instance, the pen widths should be `very thin`, `thin`, `thick`, etc., arrows should include `Computer Modern Rightarrow` and `Stealth`, and so on.  To do this, use `tikz.isy` in place of the `basic.isy` stylesheet, and un-check *Export stylesheet* in the ipelet dialog.  Be sure to include the TikZ libraries `arrows.meta` and `patterns`, and the provided library `ipe`.  The latter defines a style `ipe import`, which should be executed in any `tikzpicture` (or a `scope`) using exported ipe code.

### Using ipe's styles

Use ipe's styles if (a) you are more comfortable with ipe than TikZ, and (b) you want to do most of the drawing in ipe, then tweak it in TikZ afterwards.  To do this, check *Export stylesheet* in the ipelet dialog.  This has the additional effect of adding an `ipe ...` prefix to many style names: for instance, the `heavier` pen becomes `ipe pen heavier` in TikZ.  This means that, if you have an ipe pen width named `thick`, it will be exported to `ipe pen thick`, and all `thick` paths will reference `ipe pen thick`.  In particular, you can't make a path that uses TikZ's native `thick` style when *Export Stylesheets* is checked.

## What it Does

The best way to get a feel for what the exporter does is to doodle in ipe and see what TikZ code is produced.  The ipelet code also has a lot of comments.  In summary, though:

 + Path objects are exported to usual TikZ paths, created with `\draw` or `\fill` or `\filldraw`.  These may contain path-to `--` commands, `arc` commands, and curve-to `..` commands, as well as `rectangle`, `ellipse`, and `circle` commands.
 + Text objects are exported to TikZ nodes, with the `ipe node` style.
 + Group objects are exported to TikZ scopes.  The clipping path, if any, becomes a `\clip` path in the scope.
 + Reference objects (marks) are exported to TikZ `\pic` commands.
 + If configured to do so, the stylesheet cascade is exported to a TikZ style called `ipe stylesheet`.
 + Tilings are exported to fill patterns (which must be defined by hand in TikZ).

### Coordinate transformations

Objects in ipe may have coordinate transformations applied to them.  In order to produce readable TikZ code, these are handled as follows.

 + If the transformation is just a translation, it is absorbed into any path coordinates.
 + For path objects, if the transformation has a nontrivial linear part, then try to decompose it as a rotation and a scale.  If that works, add `rotate=` and `scale=` options to the path.
 + In this case, however, it's not clear what the origin of the transformation should be: in ipe, this is the origin of the paper at the time the object was created, which doesn't make much sense.  The compromise is to use the first mentioned coordinate as the origin.
 + If the decomposition didn't work, use TikZ's `cm=` option to pass the coordinate matrix directly.
 
### Styles and options

Most options are exported symbolically, as expected.  Some are exported as numbers or RGB colors, if necessary.  Note that, if not using *Export stylesheet*, then all symbols must be known to TikZ as styles.

### The TikZ stylesheet `tikz.isy`

This file makes ipe use TikZ's styles for almost everything it can draw.  Among other things, this file contains:

 + Color definitions for the built-in colors from `xcolor`.
 + The TikZ default cap, join, and fill rules.
 + TikZ's pen widths, dash styles, and opacity styles.
 + TikZ's built-in fill patterns, used for tilings.
 + Definitions of ipe's standard mark/reference shapes.  (TikZ has no analogue to use.)
 + Definitions of some of TikZ's standard arrows, so ipe knows how to draw them.  Since TikZ's arrow facility is much more sophisticated than ipe's, note that exported TikZ arrows will not look quite the same as in ipe.
 + Ipe's usual `textsize` definitions.
 
### The ipe compatibility library `tikzlibraryipe.code.tex`

This file is the TikZ glue code that complements the exporter.  Among other things, this file contains:

 + The definitions `x=1bp` and `y=1bp`.  The basic unit in ipe is the PDF point, which in LaTeX is a "big point" (as opposed to a "Knuth point", which is slightly smaller).  If the exported code is to be readable, it had better use ipe's basic unit.
 + Definition of the `ipe node` style, which sets the default anchor and removes inner and outer separation.
 + Arrow tip definitions for ipe's arrows.  Ipe's arrows look like certain standard TikZ arrows, but they behave rather differently with respect to line width, tip position, etc.  The arrows defined in the `ipe` library behave almost exactly like ipe's arrows.
 + Definitions of ipe's marks (`circle`, `disk`, etc.) as TikZ `pic` commands.
 
## Limitations
