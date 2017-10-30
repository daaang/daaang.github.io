---
layout: post
title:  "Early thoughts on a PiLO manager"
date:   2017-10-30 10:44:52 -0400
---
I'm thinking about how I can wire a raspberry pi zero into computers I
build so that I'd be able to power-cycle the computer remotely. Ideally,
I'd also be able to capture video and emulate a keyboard to get a
console stream. I'm separating this into four pieces:

1.  Power the rPi zero using the ATX power supply.
2.  Wire the rPi zero between the chassis and the motherboard.
3.  Connected to a motherboard USB header, the rPi zero should pretend
    to be a USB keyboard device.
4.  An entirely separate machine (probably another raspberry pi)
    captures video and provides a good-enough web UI.

This first draft of a circuit schematic deals with the first two only:

![First-draft circuit schematic][1]

In particular, I've placed resistors in a few places where I don't know
(a) whether I need resistors or (b) what impedence I'll need from them.
Also, I have a switch toggling power to the rPi zero, and I'm not sure
how exactly it ought to work.

Ideally, it'd be something like a transistor, except where the addition
of current would *cut* power rather than allow it. Talking to Noah, it
sounds like I'll need a capacitor or a flip-flop on a "reset" model, and
I don't fully understand what those are yet.

I still have a lot to learn, but I've decided it's best to start getting
my thoughts out there. If I succeed, this will be useful for later
collating and presenting as a guide for anyone else who wants to do
this.

[1]: {{ site.url }}/img/first-draft-pilo-circuit.png
