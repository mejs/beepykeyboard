Beepy keyboard sound
=====================================

This project provides beeping sound when typing, intended for use with an IBM Model M keyboard with an integrated PC Speaker. 

Forked from [Bucklespring project](https://github.com/zevv/bucklespring) by [Ico Doornekamp](https://github.com/zevv/).

Below are notes on the changes I've made, but the main functionality works pretty much the same way as in the original project. The project is adapted for specific use with my Model M on a laptop running Linux.

![Model M](/img/model-m.jpg)

My IBM Model M keyboard came with an integrated PC speaker, wired to pins F and A on the SDL connector used by the Model M:

![SDL diagram](/img/sdl.png)

Diagram from [ps-2.kev009.com](http://ps-2.kev009.com/ohlandl/keyboard/Keyboard.html#RS6000_Style_Pinout_for_Keyboard)

![IBM Model M speaker](/img/speaker.png)

The speaker is an 8 ohm, 0.2W, IBM part number 1392326

Per [ps-2.kev009.com](http://ps-2.kev009.com/ohlandl/keyboard/Keyboard.html#RS6000_Style_Pinout_for_Keyboard), these speakers were added to Model M's meant for use with [IBM RS/6000 workstations](https://en.wikipedia.org/wiki/IBM_RISC_System/6000). I couldn't find a recording of how the speaker was used when connected to an RS/6000, but per a [few](https://deskthority.net/viewtopic.php?t=12153) [forum](https://geekhack.org/index.php?topic=16219.0) [posts](), these were meant to emulate terminal beeping sounds. Unfortunately none of the forum posters reported whether they were able to enable the speaker.

I wanted to enable the speaker, so I attempted to wire the spare pins 5 and 6 on the PS/2 side of the keyboard cable. Unfortunately, the pins 5 and 6 on the PS/2 side and pins F and A on the SDL side were not terminated on my cable, making this impossible. While I found some SDL connectors for sale, I don't have a crimping tool capable of terminating them, so I gave up on this idea.

Instead, I decided to wire the speaker directly, using a 3.5mm audio cable:

![IBM Model M speaker](/img/speaker2.png)

![Audio cable](/img/audiocable.png)

The speaker is connected to a line out on my Dell D6000 docking station. 

Once I completed the hardware portion of this project, I started looking for a good typing sound emulation solution. The bucklespring program provided a lot more complexity, since [zevv](https://github.com/zevv) included precise buckling sounds for each key, but it was otherwise exactly what I was looking for, including additional functionality such as muting and unmuting.

To make it produce beeping sounds instead of buckle springs, I recorded a 1khz beep using the speaker-test program:

```
$ ( speaker-test -t sine -f 1000 )& pid=$! ; sleep 0.1s ; kill -9 $pid
```

Since the original code replicate both pressing and releasing a key (just like a real bucklespring), I removed all the wav files ending in -0 (for release), and replaced all the wav files ending in -1 (for pressing the key). This should be done more ellegantly by changing the code, but for now it works with intended effect.

Additional changes I've made:

- defined the model M scroll lock key as 0x77 to use as mute key
- added LED control for engaging the mute key to light up the scroll lock LED (by issuing "xset led 3" to the system)

I run the program with the mute function (-M) and then double press the scroll lock to enable beeping (turning scroll lock LED on). When done with beeping, I double click the scroll lock (turning scroll lock LED off).

Here's a [sample recording](https://vimeo.com/441630096), including unmuting and muting the program.




Installation
------------


### Linux, building from source

To compile on debian-based linux distributions, first make sure the require
libraries and header files are installed, then simply run `make`:

#### Dependencies on Debian
```
$ sudo apt-get install libopenal-dev libalure-dev libxtst-dev
```

#### Dependencies on Arch Linux
```
$ sudo pacman -S openal alure libxtst
```

#### Dependencies on Fedora Linux
```
$ sudo dnf install gcc openal-soft-devel alure-devel libX11-devel libXtst-devel
```

#### Building
```
$ make
$ ./buckle
```

The default Linux build requires X11 for grabbing events. If you want to use
Bucklespring on the linux console or Wayland display server, you can configure
buckle to read events from the raw input devices in /dev/input. This will
require special permissions for buckle to open the devices, though. Build with

```
$ make libinput=1
```


Usage
-----

````
usage: ./buckle [options]

options:

  -d DEVICE use OpenAL audio device DEVICE
  -f        use a fallback sound for unknown keys
  -g GAIN   set playback gain [0..100]
  -m CODE   use CODE as mute key (default 0x46 for scroll lock)
  -M        start the program muted
  -h        show help
  -l        list available openAL audio devices
  -p PATH   load .wav files from directory PATH
  -s WIDTH  set stereo width [0..100]
  -v        increase verbosity / debugging
````
