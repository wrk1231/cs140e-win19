---
layout: page
title: Interrupts.
show_on_index: true
---

### Overview


Big picture for today's lab:


   0. We'll show how to setup interrupts / exceptions on the r/pi.
   These are useful for responding quickly to devices, do pre-emptive
   threads, handle protection faults.

   1. We strip interrupts down to a fairly small amount of code.  You will
   go over each line and get a feel for what is going on.  

   2. You will use timer interrupts to implement a simple but useful
   statistical profiler.

The good thing about the lab is that interrupts are often made very
complicated or just discussed so abstractly it's hard to understand them.
Hopefully by the end of today you'll have a reasonable grasp of at least
one concrete implementation.  If you start kicking its tires and replacing
different pieces with equivalant methods you should get a pretty firm
grasp.

#### Why interrupts

As you'll see over the rest of the quarter, managing multiple devices
can be difficult.  Either you check constantly (poll), which means most
checks are fruitless.    Or you do not check constantly, which means
most actions have significant delay.  Interrupts will allow you do to
your normal actions, yet get interrupted as soon as an interesting
event happens on a device you care about.

You can use interrupts to mitigate the problem of device input by telling
the pi to jump to an interrupt handler (a piece of code with some special
rules) when something interesting happens.  An interrupt will interrupt
your normal code, run, finish, and jump back.  If both your regular code
and the interrupt handler read / write the same variables (which for us
will be the common case) you can have race conditions where they both
overwrite each other's values (partial solution: have one reader and one
writer) or miss updates because the compiler doesn't know about interrupt
handling and so eliminates variable reads b/c it doesn't see anyone
changing them (partial solution: mark shared variables as "volatile").

Interrupts also allow you to not trust code to complete promptly, by giving
you the ability to run it for a given amount, and then interrupt (pre-empt)
it and switch to another thread of control.  We will use this ability
to make a pre-emptive threads package and, later, user-level processes.
The timer interrupt we do for today's lab will give you the basic 
framework to do this.

Traditionally interrupts are used to refer to transfers of control
initiated by "external" events (such as a device or timer-expiration).
These are sometimes called asynchrounous events.  Exceptions are
more-or-less the same, except they are synchronous events triggered by the
currently running thread (often by the previous or current instruction,
such as a access fault, divide by zero, illegal instruction, etc.
The framework we use will handle these as well; and, in fact, on the
arm at a mechanical level there is little difference.

### Supplemental documents

There's plenty to read, all put in the `docs` directory in this lab:
 
   1. If you get confused, the overview at `valvers` was useful: (http://www.valvers.com/open-software/raspberry-pi/step04-bare-metal-programming-in-c-pt4)

   2. We annotated the Broadcom discussion of general interrupts and
   timer interrupts on the pi in `docs/BCM2835-ARM-timer-int.annot.pdf`.
   It's actually not too bad.

    3. We annotated the ARM discussion of registers and interrupts in
   `docs/armv6-interrupts.annot.pdf`.

    4. There are two useful lectures on the ARM instruction set.
    Kind of dry, but easier to get through than the arm documents:
    `docs/Arm_EE382N_4.pdf` and `docs/Lecture8.pdf`.

If you find other documents that are more helpful, let us know!

### Deliverables:

Turn-in:

  1.  Look through the code in `timer-int`, compile it, run it.  Make sure
  you can answer the questions in the comments.  We'll walk through it
  in lab.

  2. Implement `gprof` (in the `gprof` subdirectory).   You should run it
  and show that the results are reasonable.


### lab extensions:

There's lots of other things to do:

  1. Mess with the timer period to get it as short as possible.

  2. To access the banked registers can use load/store multiple (ldm, stm)
    with the caret after the register list: `stmia sp, {sp, lr}^`.

  3. Using a suffix for the `CPSR` and `SPSR` (e.g., `SPSR_cxsf`) to 
	specify the fields you are [modifying]
     (https://www.raspberrypi.org/forums/viewtopic.php?t=139856).

  4. New instructions `cps`, `srs`, `rfe` to manipulate privileged state
   (e.g., Section A4-113 in `docs/armv6.pdf`).   So you can do things like

		cpsie if @ Enable irq and fiq interrupts
		cpsid if @ ...and disable them