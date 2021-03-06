impress.js extensions by Jacek Kopecky
============

`impress.js` is a presentation framework based on the power of CSS3 transforms
and transitions in modern browsers and inspired by the idea behind prezi.com.

The original impress.js library is at
[impress.js repository](http://github.com/impress/impress.js)

Changes by Jacek Kopecky
------------

<!-- TOC depth:4 withLinks:1 updateOnSave:1 orderedList:0 -->

- [new features](#new-features)
	- [remote control](#remote-control)
	- [presenter console](#presenter-console)
	- [speaker notes](#speaker-notes)
	- [multiscreen support](#multiscreen-support)
	- [radial positioning](#radial-positioning)
	- [relative positioning](#relative-positioning)
	- [step groups useful for styling](#step-groups-useful-for-styling)
	- [sequential step numbering classes and groups for styling](#sequential-step-numbering-classes-and-groups-for-styling)
	- [skipped steps](#skipped-steps)
	- [mainoverview](#mainoverview)
	- [blank steps](#blank-steps)
	- [custom step perspective](#custom-step-perspective)
	- [interactive forms through the RC channel](#interactive-forms-through-the-rc-channel)
	- [out-of-band presentation path definitions](#out-of-band-presentation-path-definitions)
- [impress.js API changes](#impressjs-api-changes)
- [smaller changes](#smaller-changes)
<!-- /TOC -->

# new features

## remote control
(press `p` in the presentation)

For controlling a presentation from another device, the remote control
script opens a WebSocket channel to an impress-server (see
https://github.com/jacekkopecky/impress-server) presumed running on the
machine that served the presentation files.

The remote control shows speaker notes (see below), the IDs of the
current and next step, wall clock and time since start, and of course big
buttons for going to the next or previous step. Pressing `b` will toggle
the big buttons if you're using the RC on a laptop and using the
computer's keyboard to navigate back and forth. Keys `=` and `-` or the
buttons `+` and `-` will make the speaker notes bigger and smaller.
Clicking/tapping the time display will reset it. Clicking/tapping the
next-step ID will open a list of all IDs for quick jumping around in the
presentation.

You can try it at http://jacek.soc.port.ac.uk/presentations/impress.js
(tested on a mac with firefox and chrome; doesn't seem to work in safari and
I don't really know why):

  1. open the presentation in two browser windows
     (possibly one on a presenting machine and another on a mobile device);
     the two windows are (for now) independent of each other
  2. in one window, press `p`, fill in some key (e.g. `abc`) - this window
     is now remote-controlled
  3. in the other window, press `o` to open the remote control view; in the
     new view then fill in the same key `abc` and some password (the server
     uses the first password it sees for any given combination of
     presentation and key)
  4. press space or the » button a few times and watch as the remote control
     view now controls the presentation in the first window
  5. any window with this presentation and this key will be controlled by
     that remote control; the order of opening the conroller and the
     controlled windows doesn't matter; in fact any presentation that
     has the right password will control the others

You can press `p` in a presentation or in the remote control to be able to
set the presentation key (the same presentation can be given in multiple
places independently if the places use different RC keys). Setting also the
password means the presentation will now forward all next()/prev()/goto()
events to all listening presentations.

## presenter console
(press `c` in the presentation)

The presenter console shows speaker notes (see below), current and next
step of the presentation, wall clock and time since start (clicking on the
timer will reset it), and the current screen ID. Keys `=` and `-` will make
the speaker notes bigger and smaller.

Open the presenter console by pressing `c`, then move the new browser
tab/window on your laptop screen while the presentation is on the
projector.

You can think of the presenter console as a remote control running on the
same machine and not needing any server.

In multiscreen presentations (see below), the multiscreen console can be
more useful than this as the presenter console.

## speaker notes

For any step, you can put notes for yourself (or anybody who's presenting
the presentation) in `<div class='stepnotes'>...</div>` preceding that step;
these notes show up in the presenter console or in the remote control.

## multiscreen support

You can have a single presentation spanning multiple screens (either with
coordination over open tabs or with coordination over a websockets
`impress-server`).

To do that, the root has to define the screens, e.g. with `data-screens="0
left:right"` which means that there are two screen setups - one with just
one screen named "0" and another with two screens named "left" and
"right". Then a step can have `data-screen="0 right"` which means it only
shows on the screens "0" and "right", while any other screen stays where
it was in the previous step. To update two screens in one step, the first
one must specify its screen with an asterisk: `data-screen="0 left*"`
would mean that this step shows up on screen 0 as usual, but on screen
left it causes the presentation to show this step while advancing to the
next step (which in this example presumably shows on the "right" screen).

To select the presentation screen of the current window,
either put "screen=id" in the query of the URI, or press `1`-`9` to select
one of the first 9 declared screens.

In a presentation, the body element gets the class based on active screen
name `impress-screen-NAME` (where NAME is the screen's name)... For
example when showing the "right" screen, the body element gets a class of
`impress-screen-right`. This can be used by the CSS for example to hide
some content that shouldn't be getting in the way on some screens.

Also, in a presentation, when a screen shows a step different from the
one that is the final in a set of screens shown together, the body gets
`impress-on-ID` classes from both steps.

Further, sometimes it is useful to specify that a step shows on one screen
while another is blank. This can be done with `data-screen="0 left
right^"` which means that this step shows on screen left and on screen
right this step means to blank the screen. To do that, impress.js adds
`impress-blank` class to the root, the CSS can then blank the screen.

There is also a **multiscreen console** (opened by pressing `s` in a
presentation) which allows you to preview the various screen configurations in a
single browser window. With the URI parameter `pres`, the multiscreen console
becomes more suitable for showing multiscreen presentations on a single screen
(e.g. when the audience wants to see your slides later), or using the console as
a presenter console (see above).

The multiscreen console also supports the URI parameter `ratio`, which is a
number that expresses the aspect ratio of the screen – this is especially useful
when testing that the presentation will work well in a given lecture theatre.
For example, 1.333 means to use the aspect ratio.

If I haven't created a YouTube screencast already, bug me about it.

## radial positioning

By default, impress.js lets you position steps on X,Y,Z coordinates. Now
it can also do positioning by a given distance at a given angle from
a given point.

For example, a step with `data-x="1000" data-y="2000" data-r="3000"
data-angle="45"` would get positioned 3000 pixels north-east (up-right) from a
point at 1000x2000 pixels. This is useful if we want to put steps around
something.

These are all the new attributes:

* `data-r` - the radius
* `data-angle` and `data-angle-z` for angle around Z-axis, clockwise from top
* `data-angle-x` and `data-angle-y` for angle around X-axis or Y-axis

Only one of these angle attributes is used; it's the first one that is
present in this sequence: `data-angle-x`, `data-angle-y`, `data-angle-z`,
`data-angle`.

`radial.html` is a demo of radial positioning.

## relative positioning

Sometimes you want to put one step next to another, or even on top of
another, which means repeating the coordinates. Repeated coordinates are
harder to keep track of, especially if you move steps around as part of
the design process. Hence, relative positioning:

To position a step relatively to a preceding step, use the attribute
`data-rel="last"` or `data-rel="#id"`. The immediately preceding step, or
the step with the given ID, will become the "origin step" for the current
step. The position of the origin step will be the center of coordinates
for the current step; the rotation of the origin step will be added to
the current step's rotation. The perspective and scale of the origin step
will be multiplied with the perspective and scale of the current step.

The scale of the origin step also affects the XYZ coordinates and the
radius of radial positioning: for example, if the current step has
`data-x="1000"` and the origin step has `data-scale="2"`, the current step
will be positioned 2000px to the right of the center of the origin step.
This allows whole groups of steps to be scaled together, keeping their
relative positions intact.

The Z rotation of the origin step also rotates the positioning axes, so
if the origin step is rotated by 90°, `data-x="1000"` means 1000px
downwards; also Z-axis radial positioning is similarly affected. Other
axis (X,Y) rotations do not affect relative (or radial) positioning;
maybe in the future?

The origin step must have an ID and it must precede the current step.

## step groups useful for styling
(e.g. for showing whole groups of steps when one of them is active)

example: in normal impress.js, if the current step has `id="a"`, the body
will have the class `impress-on-a`; with groups, if the current step
also has `data-group="b c"`, the body will have the classes `impress-on-b`
and `impress-on-c` as well

## sequential step numbering classes and groups for styling

Every step gets a class `impress-step-XX` where XX is the sequential
number of the step, so you can have CSS rules for `.impress-step-XX`;
this can be useful to give a bunch of steps a smooth color transition.

Every step also gets a group `step-XX` so you can have CSS rules with
`.impress-on-step-XX` for example to have smoothly changing presentation
backgrounds.

## skipped steps
(steps with the class `skip`)

this is useful to have content positioned by impress.js (with `data-x`,
`data-y` etc.) but not constituting a step – e.g. when there is a big
picture where various steps zoom in on parts of it

## mainoverview
*(tweak)* key `up` goes to step with id "**mainoverview**" (if present, else
to previous step like normal) – this is for good access to presentation
overview, together with clicking it will then allow quick navigation

## blank steps
(added in demo CSS)

because sometimes it's useful in a presentation to hide everything and
just talk, you can add a step with `data-group="blank"` and the CSS
can then blank the screen when the root has the class `.impress-on-blank`.

## custom step perspective

sometimes you may want to affect a step's perspective, use
`data-perspective="number"` for that, the resulting perspective is the
default multiplied by the number

## interactive forms through the RC channel

To make lectures more interactive, it may be useful to ask the audience
questions; the remote control channel can carry those messages.
`forms.html` contains a demo of a presentation with a form and a live
chart of the results.

There is also a forms view for lightweight clients, it can be triggered
by the key `f` or by the URI parameter `formsview`; the resulting page's
URI can be given to clients as a "clicker".

Todo: this needs to be refactored, made reusable, and packaged somehow.

## out-of-band presentation path definitions

Now `impress().init()` can accept a `pathData` option which is a  JavaScript
structure that specifies the desired path through the  steps in the
presentation, and also speaker notes.

The remote control recognizes `pathData` only like this: there should be a
javascript element with the id `impressStepPathScript` which, when evaluated,
will give a value to `window.impressStepPathData`. (FIXME: that should be JSON.)

FIXME: add more documentation and a demo.

# impress.js API changes

 - added `curr()` call in the impress API to return the current step
 - making the API instrumentable (when an API function
   wants to call another (like when next() calls goto()), it will
   call the current one; so you can change the API)
 - added `findNext()` to the impress API
 - added API flag `api.disableInputEvents` to disable input events, e.g. when
   remote control shows password input field; anything that reacts to
   keyboard or mouse input to control the presentation should heed the flag
 - added `options` to `init()`:
   - `hashChanges` (default true)
     make it false to disable URI changes while presenting (so that Firefox on
     Mac in fullscreen with hidden location bar doesn't show the location bar
     on every step)
   - `pathData` see [above](#out-of-band-presentation-path-definitions)
 - added `setScreen(screen)` to set the current screen in a multi-screen setup
 - added `currScreen()` to retrieve the current screen
 - added `getScreenBundles()` to retrieve known screen bundles
 - added `impressStepData` to every step element in the DOM to have access
   to impress's positioning information from the outside

# smaller changes

 - *(tweak)* disabled `tab` key because of interactions with cmd-tab on mac
 - *(tweak)* disabled `PgUp`/`PgDn` keys because of interactions with tab
   switching in my browser
 - *(tweak)* added keys `h` `j` `k` `l` for vi-like navigation
   (left/down/up/right) and `p` `n` for prev/next.
 - *(refactoring)* moved list of recognized keys to extra function
 - *(fix)* scaling the window should not affect the perspective - the position
   of the camera


LICENSE
---------

Original copyright of impress.js 2011-2014 Bartek Szopka

Copyright of the changes 2014–2015 Jacek Kopecky

Released under the MIT and GPL (version 2 or later) Licenses.
