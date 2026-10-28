
> # *Fork Info*
>  
><sub>By Forked Camisadan, Repository by [Gibtnix](https://github.com/Gibtnix/Apex-Macros)</sub>
> 
> Hello. I made this fork as the original repo for some reason binds the left hand keys to random buttons, and also manually binds the media keys for some reason despite the fact media keys aren't handled in the way the ``.hwdb`` expects.
>
> Now, L1-4 are bound to KEY_MACRO11-14 meaning they have their very own keycodes (supported on most Linux kernels, tested on CachyOS Mainline.), ``MX1-10`` Are bound to ``KEY_MACRO1-10``, and ``M1-12`` are bound to ``F13-24``. **Note, this fork does not fix the base Wayland / X11 behaviours of F13-24 firing things like emoji pickers and random DE-specific behaviours.**
>
> ### KDE Users:
>
> Any unwanted ``F13-24`` behaviour can be disabled by doing the following:
>
> - Go to System Settings
> - Go to Keyboard -> Keyboard
> - In the top right, click the Key Bindings button
> - In the Key Bindings menu, tick the Configure Keyboard Options button
> - Search for F13 / go to the Function Keys category
> - Enable 'Use F13-F24 as usual function keys'
>
> Note that the fork makes no major changes, just modifies the .hwdb and makes any ``sudo udevadm control --reload`` also run:
> ```
> sudo udevadm trigger --action=change --subsystem-match=input --sysname-match='event*'
>sudo udevadm settle
> ```
> Meaning the changes apply instantly without a reboot.
> Hooray!


# Apex-Macros

# General

Apex-Macros is a minimalistic tool to enable (or disable) the additional macro keys of the
SteelSeries Apex keyboard family. There is no further configuration option available, e.g. color
configurations or polling frequencies cannot be changed. Actually there exist respective Linux
tools, for instance this one: https://github.com/tuxmark5/ApexCtl
My intention to create Apex-Macros just has been the fact that the macro keys are automatically
disabled as soon as you reboot the system, in contrast to the remaining configurations. Hence,
Apex-Macros is more functional and does not aim at all possible, but actually mainly unneeded
configuration options. Consequently, there is just a minimum number of additional dependencies.


# Installation & Requirements

I tried to keep the external dependencies to a minimum level, however there are some unavoidable
ones. These are:

 * GCC     - the C compiler
 * make    - the main build tool of the Linux world
 * LIBUSB  - to communicate with the keyboard, LIBUSB is required

To install the program, there is an installation script `install.sh` which can be run by opening
the respective folder in a terminal and typing

    ./install.sh

or if there are any problems you can try

    bash install.sh

as well which will most certainly will work on most Linux distributions. This script will do the
following steps, if you do not want to use the installation script for some reason, you can use
the manual commands instead:

 * installation of the dependencies
   The script automatically detects your distribution:
   - **Debian/Ubuntu**: Installs `gcc`, `make`, `libhidapi-dev` using `apt`.
   - **Arch/CachyOS**: Installs `base-devel`, `hidapi`, `evtest` using `pacman`.
   ```bash
   # The script will run the appropriate command for you:
   # sudo apt install -y gcc make libhidapi-dev
   # OR
   # sudo pacman -S --needed --noconfirm base-devel hidapi evtest
   ```

 * compiling of Apex-Macros
   ```
   make
   ```

 * clean up
   ```
   make clean
   ```

 * copy the built program to '/usr/local/bin/apex-macros' and set its permissions
   ```
   sudo mv -fv apex-macros /usr/local/bin/apex-macros
   sudo chmod 755 /usr/local/bin/apex-macros
   ```

Whenever it is used, it should always be run as root because otherwise, the communication with
the keyboard is not possible, hence always use the sudo prefix.


# Supported Devices

Currently the Apex M800, Apex 350 and Apex 300 as well as the respective previous version Apex and
Apex [RAW] are supported (or at least they should be - I could not test them all, so feel free to
give me some feedback if they do or not do).


# Usability

First of all: If the program is not executed as root, it will behave as if there currently is no
compatible keyboard connected to the system! Hence, if it does not work, make sure you to execute
it as root. The main usability is quite simple, to enable the macro keys just run

    sudo apex-macros enable

or to disable them, run:

    sudo apex-macros disable


# Macro Keys Configuration

The macro keys should be detected by the system if `sudo apex-macros enable` has been successfully
executed. To verify this, use a tool to check key events:

*   **Modern Systems / Wayland (Recommended)**: Use `evtest`.
    *   **Arch/CachyOS**: Installed automatically by the script.
    *   **Debian/Ubuntu**: Install manually: `sudo apt install evtest`
    ```bash
    sudo evtest
    ```
*   **Legacy / X11**: You can also use `xev`.
    ```bash
    xev
    ```

The terminal should notify about key press events as soon as the macro keys are pressed. Further,
some keys are bind to predefined multimedia actions, they should already be usable in your media
player (VLC for instance). There might be some other keys that are mapped to the same key code.
Hence, it is only possible to assign an action to all of them, but not a different action for
each one. This can only be changed by modifying the `90-apex.hwdb` file accordingly.

## Configuring Macro Actions

To map custom commands to the macro keys, you need an external tool.

### For Wayland Users (CachyOS / KDE Plasma / GNOME)
`xbindkeys` is designed for X11 and **does not work reliably on Wayland**. Instead, you should use:

1.  **input-remapper** (Recommended): A powerful GUI tool to map buttons to macros, keys, or commands. It works seamlessly on both Wayland and X11.
    ```bash
    # Install from AUR (using yay or paru)
    paru -S input-remapper-git
    # Start the service
    sudo systemctl enable --now input-remapper
    ```

2.  **Desktop Environment Shortcuts**:
    - **KDE Plasma**: Go to *System Settings* -> *Shortcuts* -> *Custom Shortcuts*. You can create new global shortcuts and press the macro key to bind it.

### For X11 Users
If you are still using X11, `xbindkeys` remains a popular option.

To install XBindKeys:

*   **Debian/Ubuntu**:
    ```bash
    sudo apt install -y xbindkeys xbindkeys-config
    ```

*   **Arch Linux**:
    ```bash
    sudo pacman -S xbindkeys
    paru -S xbindkeys-config # Optional GUI from AUR (use paru, yay, etc.)
    ```


# Autostart

A useful additional feature is the optional autostart functionality since the macro keys will be
disabled again as soon as you restart your computer or put it into standby mode. Hence, it is quite
useful to automatically activate the macro keys as soon as a compatible keyboard is connected to
the computer. To do this, there is an additional script called `autostart.sh` that can do this for
you. This script creates a file

    /etc/udev/rules.d/90-apex-macros.rules

that informs the udev service to call Apex-Macros as soon as a compatible keyboard is connected to
the computer. To create this file simply run:

    ./autostart.sh

Finally, the autostart can be disabled by running

    ./autostart.sh --disable

which will disable the autostart by removing the rule file.


# Uninstallation

Apex-Macros also comes with an uninstallation script uninstall.sh which will remove the program
/usr/local/bin/apex-macros as well as running ./autostart --disable, i.e. it disables the
autostart. If you want to use it, simply run:

    ./uninstall.sh
