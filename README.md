# ydotool
Generic Linux command-line automation tool (no X!)

[![Build Status](https://github.com/ReimuNotMoe/ydotool/workflows/Build/badge.svg)](https://github.com/ReimuNotMoe/ydotool/actions/workflows/push_pr_build_cmake.yml) [![Release Status](https://github.com/ReimuNotMoe/ydotool/workflows/Release/badge.svg)](https://github.com/ReimuNotMoe/ydotool/actions/workflows/release_cmake.yml)

## Important Notes
### Current situation
This project is now being maintained **thanks to all the people that are supporting this project!**

All backers and sponsors are listed [here](https://github.com/TheNeuronProject/BACKERS/blob/main/README.md).

### How to support us
- Donate on [Patreon](https://www.patreon.com/classicoldsong)
- Buy our products on our [Tindie Store](https://www.tindie.com/stores/sudomaker/), if they are meaningful to you (please leave a message: `ydotool user`, so we can know that this purchase is for supporting ydotool)

### More talks
[READ THIS BEFORE COMPLAINING ANYTHING ABOUT THE DONATION & ITS PRICING](https://christine.website/blog/open-source-broken-2021-12-11)

Independent software developers in China, like us, have 10 times more life pressure than Marak, the author of faker.js. Since ydotool has the opportunity to benefit large IT companies who won't pay a penny to us, we've changed the license to AGPLv3. These large IT companies are the main cause of life pressure here, such as the "996" working hours.

**Marak's fate will repeat on all open source developers eventually (of course we aren't talking about those who were born in billionare families) if we just keep fighting with each other and do nothing to improve the situation. If you make open source software as well, don't hesitate to ask for donations if you actually want them.**

Also make sure you understand all the terms of AGPLv3 before using this software.

## Usage
Currently implemented command(s):
- `type` - Type a string
- `sleep` - Sleep some time
- `key` - Press keys
- `mousemove` - Move mouse pointer to absolute position
- `click` - Click on mouse buttons
- `recorder` - Record/replay input events

Now it's possible to chain multiple commands together, separated by a comma between two spaces.

## Examples
Switch to tty1, wait 2 seconds, and type some words:

    ydotool key ctrl+alt+f1 , sleep 2000 , type 'Hey guys. This is Austin.'

Close a window in graphical environment:

    ydotool key Alt+F4

Relatively move mouse pointer to -100,100:

    ydotool mousemove -100 100

    Warning: implicit mousemove call does not work with negative values, so until https://github.com/ReimuNotMoe/ydotool/issues/119 is fixed, use explicit method:
    ydotool mousemove -x -100 -y 100

Move mouse pointer to 100,100:

    ydotool mousemove --absolute 100 100

Mouse right click:

    ydotool click right

Mouse click queue:

    ydotool click left right left

Mouse repeating left click:

    ydotool click --repeat 5 --next-delay 25 left

## Notes
#### Runtime
This program requires access to `/dev/uinput`. **This usually requires root permissions.**

You can use it on anything as long as it accepts keyboard/mouse/whatever input. For example, wayland, text console, etc.

#### Available key names
See `/usr/include/linux/input-event-codes.h`

#### Why a background service is needed
ydotool works differently from xdotool. xdotool sends X events directly to X server, while ydotool uses the uinput framework of Linux kernel to emulate an input device.

When ydotool runs and creates a virtual input device, it will take some time for your graphical environment (X11/Wayland) to recognize and enable the virtual input device. (Usually done by udev)

So, if the delay was too short, the virtual input device may not got recognized & enabled by your graphical environment in time.

In order to solve this problem, I made a persistent background service, ydotoold, to hold a persistent virtual device, and accept input from ydotool. When ydotoold is unavailable, ydotool will try to work without it.

## Build
**CMake 3.14+ is required.**

All dependencies will be configured by CPM automatically to save you from building & installing them manually. **So an Internet connection is required.**

The libraries, `libevdevPlus` and `libuInputPlus`, are too small and too troublesome to be complied as shared libraries. So now they are statically linked. As a result, the compiled binaries will only depend on `libc` and `libstdc++`.

CMake will no longer build the packages or install compiled files to system directories. Because it's complicated to build packages for every distro correctly using CPack, and every distribution has its own customs of filesystem hierarchy. I suggest you to copy compiled files to desired destination manually.


### Compile

    mkdir build
    cd build
    cmake ..
    make -j `nproc`


## Troubleshooting
### Custom keyboard layouts
Currently, ydotool does not recognize if the user is using a custom keyboard layout. In order to comfortably use ydotool alongside a custom keyboard layout, the user could use one of the following fixes/workarounds:

#### Sway
In [sway](https://github.com/swaywm/sway), the process is [fairly easy](https://github.com/swaywm/sway/wiki#keyboard-layout). Following the instructions there, you would end up with something like:
```
input "16700:8197:DELL_DELL_USB_Keyboard" {
	xkb_layout "us,us"
	xkb_variant "dvorak,"
	xkb_options "grp:shifts_toggle, caps:swapescape"
}
```
The identifier for your keyboard can be obtained from the output of `swaymsg -t get_inputs`.

#### Use a hardware-configurable keyboard
[As mentioned here](https://github.com/ReimuNotMoe/ydotool/issues/43#issuecomment-605921288), consider using a hardware-based configuration that supports using a custom layout without configuring it in software.
