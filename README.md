# Welcome to the JTinnovations Project Page

## What is this?

This page is to document the development of a project using the NuttX POSIX realtime operating system.

Since this is a commercial design, the full details of the end product will not be shared as yet. This is a record of the steps I used to get the project underway, in the hope it might help others doing similar things and, in the future, as a public record of the full project to assist future developers working with the design.

<details open>
<summary>About Me</summary>

I'm Tim, the "T" of JTinnovations, and the <s>chief cook and bottle-washer</s> sole engineer. I'm an all-rounder, undertaking mechanical (SolidEdge 3D CAD), hardware (OrCAD) and software engineering (C and a bit of C#).

My career has spanned >40 years. I cut my teeth as a teenager in the 1970's getting Commodore PETs to do silly things using BASIC in Dixons' showrooms, and I was a keen electronics hobbyist (my pride and joy was an "International 4600 Synthesiser" mostly built with reclaimed components plus Maplin parts). Pre-university saw me working at Marconi where I got my first taste of UNIX on a DEC VAX: ASCII-based dungeons and dragons at lunchtime is what I remember most, but I also learnt Fortran to create a waveguide RF filter design tool for RADAR systems.

Once I graduated with a degree in Electronic Engineering, I initially had a job as a hardware engineer designing test signal generators for the TV industry, but soon jumped departments to write assembler on a Z80 (NSC800 to be pedantic) for what should have been the world's first microprocessor controlled colour TV camera (Sony stole that thunder in the end), followed by many years writing OCCAM for transputers, for an audio product, before I moved on to C with a little bit of C# along the way. In parallel with this I was always very involved with mechanical and overall product design.

Fast forward through 20 years of so-called promotion into product management, and redundancy pushed me back into product design and engineering: in reality my forte and passion. My wife and I started JTinnovations in 2011 (Jane is the "J" of JTinnovations), combining my engineering experience with our joint love of fast cars, to develop products for like-minded enthusiasts.

</details>

# The Project

A custom board with the following hardware:

- SAMA5D27-D1G
  - Arm Cortex A5 running up to 500 MHz, with 1Gbit DDR2 memory
  - Many peripherals, almost all of which are used with this design
    - CAN, LIN, LCD, resistive touchscreen (TSD), main and flexcom U(S)ART, SPI, I2C, PWM, RTTC, ADC, USB device+host, Class D Audio, efuse (SFC)
  - More information [here](https://www.microchip.com/en-us/product/ATSAMA5D27C-D1G).
- TI LP5030/5036 RGB LED driver chips
- SkyTraq dead-reckoning GPS receiver with 6-axis motion sensors
- USB C as front-end to the processor's hi-speed USB device and host peripherals using an OnSemi FUSB302 as the interface controller
  - Act as MSD device for user configuration data exchange as well as access to internal logging memory
  - Support USB memory sticks
  - Support USB keyboard/mouse
- analogue and digital I/O

## Why Nuttx?

The project had progressed quite well as a bare-metal design - something I'm totally comfortable with - until I realised that the plan for the product to not only behave as a USB device (well catered for by Atmel/Microchip) but also as a USB host,  to allow the connection of memory sticks for example, meant the bare-metal approach would likely prove to be an uphill struggle.

Looking at the available NuttX features and functions opened my eyes to the many many things this product could do, with (I hoped), ease: so I made the decision to rework the design.

It has not been easy. Understatement of the century.

## Before you ever start with NuttX, maybe read this

<details open>
<summary>What you need to know</summary>

<h3>Development Environment </h3>

I'm experienced, I thought. I've tinkered with Linux, I used to use Unix, 40 years experience. NuttX can be built under Windows (with which I have the most experience) or Mac (got two of those, but not an expert). It'll be easy!

<h4>MAC</h4>


I followed the NuttX guides but got lots of error messages, it wouldn't compile, and I didn't have enough MacOS experience to work out what was wrong. So I gave up.


#### Windows with Cygwin + GNU

I'd worked on some applications this way in the past but it always felt slow and cumbersome, so I didn't try it. Especially as Win10/11 has the Linux Subsystem (WSL).


#### Ubuntu using WSL

I followed the guides in the NuttX documentation - and it worked!

So I started development that way, using a SAMA5D2 eval board to get familiar with it all.

Unfortunately, it all went wrong when I tried to get a debugging environment working to use my Segger J-link.

I tried Eclipse under WSL using X-Windows, but it wouldn't run; OpenOCD; Visual GDB. All had issues - they probably could be sorted, just not by me in the time available.

So I sat back and realised that since NuttX is POSIX, and more similar to Linux than any other OS (apart from MacOS maybe - but, as said, I have the least experience of that), and with so many Linux tools out there, it was time to bite the bullet and go "full-Linux".

#### Linux

Bottom line: I should not have bothered with any of the above and I strongly strongly commend anyone wanting to start working with NuttX to NOT mess around: use a Linux development environment!

A £450 Intel NUC with a quad-core i5 processor was bought, the dual-boot SSD wiped, and a fresh install of Ubuntu 20.04 applied, followed by:

- [arm-none-eabi tools](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads)
  - I followed the information [here](https://askubuntu.com/questions/1243252/how-to-install-arm-none-eabi-gdb-on-ubuntu-20-04-lts-focal-fossa) but also found I had to add links for:
    - arm-none-eabi-ar
    - arm-none-eabi-ld
    - arm-none-eabi--nm
    - arm-none-eabi-objcopy
- Visual Studio Code with extensions
  - Microsoft C/C++
  - [Cortex debug](https://marketplace.visualstudio.com/items?itemName=marus25.cortex-debug&ssr=false#overview)


## Things to read

### Books

I was not that familiar with POSIX, and I was advised to read some books. I read 2 of them, started work, got into grief, then read the third, and wished I'd read that before starting. Mine were all bought second hand for not much money.

1. POSIX programmers guide (POSIX.1)  
2. POSIX.4 Programmers Guide: Programming for the Real World Programming for the Real World
3. Programming with POSIX Threads (Addison-Wesley Professional Computing Series) Addison-Wesley Professional Computing Series
4. "Pthreads Programming A Posix Standard for Better Multiprocessing A Nutshell handbook" - a good addition to your reading

Why?

The NuttX community suggests the best approach is to look at similar boards/drivers and learn from that. I did that but REALLY struggled with the extensive use of structures - it was consistent but I didn't "get it".

Reading "Programming with POSIX threads" _really_ helped.

### NuttX Documentation

With the benefit of hindsight, I would suggest familiarisation with:

- [The NuttX coding standard](https://nuttx.apache.org/docs/latest/contributing/coding_style.html?highlight=coding%20standard). If you don't follow this then any code you do write will not be accepted should you want to submit your code back to the community. It's a unique, unusual, maybe even quirky standard, but it's not overly onerous.
- As much of the [basic NuttX documentation](https://nuttx.apache.org/docs/latest/index.html) as you can cope with. And revisit it often!

## It may be in the repo, but...

Once I was underway I very soon discovered that the SAMA5D2 processor, although in the repo with support for various eval boards, was full of bugs and incomplete support. Not saying this is true of all processors, but I would advice MUCH caution - do not expect your custom board to work "just like that". Take a careful look and see if there are drivers there for what you need.

At the start of this journey I was very nervous of modifying code and submitting changes to the main NuttX repo, so I set about making changes on my own personal fork. It went well, but after an enforced break due to family issues, I found that changes to the underlying NuttX "master" made much of my work outdated and incompatible.

Did I mention? I had no experience of GitHub, having used a locally hosted SVN repo for all my other work so far. It was a watershed momennt.

I decided to bite on the bullet, and learned how to merge the latest "upstream/master" to my fork, re-work my changes as needed and - feeling brave - start submitting PR's to get my changes incorporated.

With the wonderful benefit of hindsight I would commend anyone to always fork NuttX and keep up to date with the latest upstream master.

### NuttX Coding Standard

I put my hand up. Over the years my coding "style" got sloppy. I so often worked with code originally written by others, then modified by me, that I never really adopted a specific style. Probably my default was related to Kernighan and Ritchie (anyone remember that!?) but heavily influenced by GNU.
  
Originally I was going to write this section to mildly criticise the NuttX style...but having used it for over a year now it is second nature and I am *really* pleased that my code is consistent. I am even changing non-NuttX code of mine to match it as I love it all being the same.
  
So: it may not be "your" style but just follow it. It's great that everything in the repo is pretty much the same.

### GitHub

Most reading this are probably well-versed with GitHub. For me, it was an uphill struggle. I think I might have made it harder for myself by using GitHub Desktop - it works OK on Windows and MAC but the Linux version is some kind of port so doesn't support many of the ease-of-use features such as cherry-picking by drag/drop: Linux really needs command-line skills.
  
Here are the few useful things I picked up along the way.

- Make plenty of use of branches <details to be added>
- rebase to "upstream master" when necessary <details to be added>
- Commits **must** be squashed, with a single commit message. The command line tricks for this are <details to be added>
- Sometimes a force-push is needed. Do it like this <details to be added>
 
</details>

