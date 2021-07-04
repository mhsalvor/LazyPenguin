Post Installation Configuration
===============================


Default Applications
--------------------

### TUI

* File manager: ranger
* System monitor: htop
* IRC client: weechat
* Web Browser: w3m
* Mail Client: neomutt (?)
* Web Search: googler
* Calendar: calcurse
* Editor: neovim; nano
* Disk cleaning: fslint
* ls replacement: lsd
* diff replacement: colordiff
* Quick commands help: tldr++
* Quotes: fortune-mod

### GUI

* App Launcher/Menu: rofi
* Powermenu: clearine-git
* Lockscreen: betterlockscreen
* Machines syncs: syncthings
* Document reader: zathura (mupdf)
* Categorized menu: morc_menu
* Browser: firefox
* File manager: pcmanfm, thunar
* Image viewer: viewnior
* Video Player: mpv; vlc
* USB Install media creator: etcher
* Image Manipulation: gimp
* Office Suit: libreoffice-fresh
* Remote Desktop (Support): anydesk
* System Monitor: stacer
* Virtual Machines: virtualbox
* Retrogaming: retroarch
* Volume control: pa-applet
* Terminal emulator : Alacritty
* Compositor: picom-jonaburg-git
* Monitor Nightmode: redshift-gtk
* Notifications: dunst (?)
* Scanner Utility: simplescan
* Torrent: deluge
* Audio control panel: pavucontrol
* Wallpaper manager: nitrogen, variety

### Mixed

* Terminal drag-n-drop support: dragon-drag-and-drop

Post Install Configuration
--------------------------

### Hostnames

Make sure your /etc/hosts contains the following:

```
127.0.0.1   localhost.localdomain   localhost   <your hostname>
127.0.1.1   localhost.localdomain   <your hostname>
::1         localhost.localdomain   localhost   <your hostname>
```

### Package Managers

#### Pacman

* Update the system: `sudo pacman -Syu`
* Remove Orphan packages: `sudo pacman -Rns $(pacman -Qtdq)`
* Clean pacman and yay caches: `sudo pacman -Scc && yay -Scc`

#### Pamac

* Update the mirror list:
    * Open "pamac-manager" and go to proprieties (  ) -> preferences -> official-repo -> refresh mirrors;
        Keeping "Worldwide" as a setting is advised.
* (  ) -> preferences -> AUR : activate AUR support and updates.

#### Autoclean packages caches

Using this hook, pacman will clean the cache after every installation.

* `sudo mkdir /etc/pacman.d/hooks`
* `sudo -e /etc/pacman.d/clean_packages_cache.hook`
```shell
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *

[Action]
Description = Cleaning pacman cache...
When = PostTransaction
Exec = /usr/bin/paccache -rk 2
```

### Services

* start cups: `systemctl enable org.cups.cups.d.service`

## Basic Packages

#### Language support

`pacman -S aspell-it libmythes mythes-it languagetool hunspell-it enchant`

#### Fonts

* Install MS fonts:
    * `git clone https://aur.archlinux.org/ttf-ms-win10.git`
    * `cd ttf-ms-win10`
    * Copy the Fonts from a licensed window installation in the same directory as
        PKGBUILD; `C:\Windows\Fonts`
    * Copy the license file in the same directory. `C:\Windows\System32\Licenses\neutral\_Default\Core\license.rtf`
    * `makepkg -rsi`

* install mac fonts:
    * `yay -S ttf-mac-fonts`

* Symbol Fonts and icons
    * `yay -S nerd-fonts-complete` All the Nerd patched fonts;
    * `yay -S ttf-all-the-icons` Icon Fonts (fontawesome,material-design-icons, atom file icons, octicons, weather-icons).
    * pacman -S ttf-font-awesome

#### System Fonts

`pacman -S adobe-source-sans-pro-fonts ttf-anonymous-pro ttf-bitstream-vera ttf-dejavu ttf-droid ttf-liberation ttf-ubuntu-font-family`

#### Multimedia packages

`pacman -S vlc gst-libav gst-plugins-good`


### System Configuration

#### Filesystems Mount Options

* all ext4 entries should have `noatime,data=writeback`
* To avoid boot errors, add the correct propriety to the filesystems: `sudo tune2fs -o journal_data_writeback /dev/sdXY`

#### Sudo

Enable sudo "tickets" to be shared across terminals. It's less secure, but it saves
me a lot of password-typing.

`sudo EDITOR=nvim visudo`

Uncomment or add `Default !tty_tickets`

#### Turn System Beep OFF

Crate the file "/etc/modprobe.d/nobeep.conf" and put the line `blacklist pcspkr` in it:

````shell
sudo echo "blacklist pcspkr" >> /etc/modprobe.d/nobeep.conf
``````

#### SSD configurations

* Check partitions alignments:
    * The easiest way is to use `parted` so you may want to install it if not already present.

```shell
parted /dev/sdX
> align-check opt (X)
```

* Enable trim:
    * For manual trim use `fstrim -v all`
    * For automatic trim, put the above command in a weekly cronjob OR, if you
        use systemd, util-linux provides a timer service: `sudo systemctl enable fstrim.timer`

* Adjust I/O schedulers: `sudo -e /etc/udev/rules.d/60-ioscheduler.rules`

```shell
# set scheduler for NVMe
ACTION=="add|change", KERNEL=="nvme[0-9]n[0-9]*", AT]TR{queue/scheduler}="none"
# set scheduler for SSD and eMMC
ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="mq-deadline"
# set scheduler for rotating disks
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq-sq"
```

#### PulseAudio

* Install PulseAudio: `sudo pacman -Sy manjaro-pulse pa-applet pavucontrol`

* Fix odd behaviour with "phone" apps:
    * comment out `load-module-role-cork` in `/etc/pulse/default.pa`

#### Magic SysRq key

It is a ‘magical’ key combo you can hit which the kernel will respond to
regardless of whatever else it is doing, unless it is completely locked up.

* To enable it: `sudo echo "i" > /proc/sys/kernel/sysrq`
* To use it press `Alt+SysRq`, release SysRq but keep pressing Alt then enter one
    of these commands:

| Keys  |   Action                                                                                   |
|:-----:|--------------------------------------------------------------------------------------------|
| 0 - 9 | Set the console log level                                                                  |
|   b   | Immediate Reboot, without unmounting or fs sync                                            |
|   c   | Perform a system crash with crashdump (if configured)                                      |
|   d   | Disable all currently held Locks                                                           |
|   e   | SIGTERM to all but PID1                                                                    |
|   f   | Call oom_kill                                                                              |
|   g   | When using Kernel Mode Settings, switch to framebuffer console, enter kdb if enabled       |
|   h   | Output an Help document                                                                    |
|   i   | SIGKILL to all bud init                                                                    |
|   j   | Forcibly "Just thaw it" – filesystems frozen by the FIFREEZE ioctl                         |
|   k   | Killall on current visual console                                                          |
|   l   | Shows a stack backtrace for all active CPUs.                                               |
|   m   | Output current memory information to the console                                           |
|   n   | Reset the nice level of all high-priority and real-time tasks                              |
|   o   | Shut off the system                                                                        |
|   p   | Output the current registers and flags to the console                                      |
|   q   | Display all active high-resolution timers and clock sources.                               |
|   r   | Switch the keyboard from raw mode, used by programs such as X11 and SVGALib, to XLATE mode |
|   s   | Sync all mounted filesystems                                                               |
|   t   | Output a list of current tasks and their information to the console                        |
|   u   | Remount all mounted filesystems in read-only mode                                          |
|   v   | Forcefully restores framebuffer console                                                    |
|   w   | Display list of blocked (D state) tasks                                                    |
|   x   | Disables lockdown (Secure Boot restrictions) on some kernels                               |
|   z   | Dump the ftrace buffer                                                                     |

##### Uses

A common use of the magic SysRq key is to perform a safe reboot of a Linux computer
which has otherwise locked up.
* REISUB
    - unRaw take control of the keyboard back from X
    - tErminate send SIGTERM to all processes, allowing them to terminate gracefully
    - kIll send SIGKILL to all processes except init, forcing them to terminate immediately
    - Sync flush data to disk
    - Unmount remount all filesystems read-only
    - reBoot

#### Cron

Enable cronjobs for simple users: `touch /etc/cron.deny`

### Hybrid Graphics

I own 2 laptops with different arrangements of hybrid GPUs. An older one with
an old intel/nVdia setup that requires Bumblebee to work in any sane way, and
a newer, yet stranger, AMD/nVidia (iGPU=AMD, dGPU=nVidia) that usually works just
fine out of the box (with Manjaro proprietary drivers autoinstall) but requires
a few tweaks to avoid flickering/tearing and smooth performances.

#### Bumblebee

1. Install needed packages:
```shell
sudo pacman -S bumblebee mesa xf86-video-intel lib32-virtualgl
yay -S lib32-nvidia{-340xx}-utils nvdock-bumblebee
sudo pacman -S bbswitch primus lib32-primus primus_vk lib32-primus_vk
```
2. Add your user to the bumblebee group: `sudo gpasswd -a <user> bumblebee`
3. Enable SystemD service: `sudo systemctl enable bumblebeed.service`

Now you can use `primusrun <application>` to run something using the dGPU, and
`pvkrun <application>` if you want Vulcan support.

For Steam games add `primusrun %command%` in the game proprieties.

##### Fix Tearing

Create a configuration X11 configuration file: `sudo -e /etc/X11/xorg.conf.d/20-gpu.conf`
Containing:
```
Section "Device"
    Identifier "intel"
    Driver "i916"
    Option "TearFree" "true"
EndSection

Section "Device"
    Identifier "nVidia"
    Driver "nvidia"
    Option "TearFree" "true"
EndSection
```

#### Native

 Manjaro comes with `nvidia-utils`, a nice packages that simplifies this whole
 process greatly.
 It's available for most distributions and you really want to install it.

 Once you have installed that and the proper drivers, you can use the dGPU by
 using the script `prime-run` that comes with `nvidia-utils` as a prefix to whatever
 command you like.

 For Steam games, put `prime-run %command%` in the proprieties of the game.

##### Fix Tearing

Create a configuration X11 configuration file: `sudo -e /etc/X11/xorg.conf.d/20-gpu.conf`
Containing:
```
Section "Device"
    Identifier "amd"
    Driver "amdgpu"
    Option "TearFree" "true"
EndSection

Section "Device"
    Identifier "nVidia"
    Driver "nvidia"
    Option "TearFree" "true"
EndSection
```

### Network

#### Firewall

* Install gufw: `sudo pacman -S gufw`
    * Launch gufw and enable it;
        By default, it will block all incoming connections and allow all exiting ones.
    * Further configurations, such as applications specific rules, may be required

#### Custom DNS

Use `nm-applet` or your favourite network manager to set these.

* Cloudfare DNS
    * IPv4
        - 1.1.1.1
        - 1.0.0.1
    * IPv6
        - 2606:4700:4700::1111
        - 2606:4700:4700::1001

* Cisco OpenDNS
    * IPv4
        - 208.67.222.222
        - 208.67.220.220
    * IPv6
        - 2620:119:35::35
        - 2620:119:53::53

* Google Public DNS
    * IPv4
        - 8.8.8.8
        - 8.8.4.4
    * IPv6
        - 2001:4860:4860::8888
        - 2001:4860:4860::8844


Windows Managers
----------------

### Bspwm

#### Windows Swallowing

The `bspwallow` packages tries to emulate xdwm "swallow" behaviour.
* Installation: `git clone https://github.com/JopStro/bspswallow `
    * Drop the chosen script somewhere in your PATH, and make it executable with `chmod +x`
    * Add `pidof bspswallow || bspswallow & ` in the autostart section of your `bspwmrc`
* Configuration: `noswallow` `swallow` `terminal` files in `.config/bspwmrc/`

NOTE
`noswallow` has 1 blank line, leave it.


Applications
------------

### Betterlockscreen

* Basic configuration:
    * Use the same wallpaper as the desktop: `source ~/.fehbg`
      Change this depending on what wallpaper application you are using.
    * Example keybinding: `alt+shift+x betterlockscreen -l dim`
* SystemD service: lockscreen when suspended:
    * Move the service file to the proper directory (AUR package already does this)
        `sudo cp betterlockscreen@.service /etc/systemd/system`
    * Enable systemd service: `sudo systemctl enable betterlockscreen@USER`

Now you can call systemctl suspend to suspend your system and betterlockscreen
service will be activated so when your system wakes your screen will be locked.

### Doom Emacs

TODO: I'm interested in trying this in the future, this is where I'll be describing the installations
and configuration process. For now, tho, this is just a place-holder.

### Dunst

* Launch in `~/.xinitrc`: `dunst &`
* Sending manual notifications: `otify-send -u {normal,low,critical} "<title>" "<content>"`
* Configuration: `vim .config/dunst/dunstrc`

```
geometry = 0x0-5-5

muse_left_click = do_action
mouse_left_click = close_all
mouse_right_action = close_current
```
### Rofi

* Replace dmenu
    * Symbolic link option:
    ```ihell
cd /usr/bin
sudo mv dmenu dmenu_orig
ln -s /usr/bin/rofi dmenu
```
    * Alias: Add the following line to .aliasrc
`command -v rofi > /dev/null && alias dmenu='rofi -dmenu '`


* If you prefer the look of dmenu, this approximate it:
```shell
rofi -show run -modi run -location 1 -width 100 \
    -lines 2 -line-margin 0 -line-padding 1 \
     -separator-style none -font "mono 10" -columns 9 -bw 0 \
     -disable-history \
     -hide-scrollbar \
     -color-window "#222222, #222222, #b1b4b3" \
    -color-normal "#222222, #b1b4b3, #222222, #005577, #b1b4b3" \
    -color-active "#222222, #b1b4b3, #222222, #007763, #b1b4b3" \
    -color-urgent "#222222, #b1b4b3, #222222, #77003d, #b1b4b3" \
     -kb-row-select "Tab" -kb-row-tab ""
```

* Basic Commands and Settings
```shell
rofi -show combi -switchers combi -combi-modi window,run
rofi -show drun -eh 2 -padding 14 —-better spacing
rofi -combi-modi window,drun -show combi -modi combi
rofi -combi-modi window,drun -show combi -modi combi -lines 20 -font roboto\ 12
rofi -modi run,drun,window,ssh,urxvt -show drun -show-icons -sidebar-mode -location 0 -width 35
rofi -modi run,drun,window -show drun -show-icons -sidebar-mode
```
* Generate keybinding cheat sheet (for sxhkd):
`awk ‘/^[a-z]/ && last {print $0,”\t”,last} {last=””} /^#/{last=$0}’ ~/.config/sxhkd/sxhkdrc | column -t -s $’\t’ | rofi -dmenu -i -no-show-icons -width 1000`

* Run "sudo commands" via  rofi using a script:

```shell
#!/bin/sh

# Take password prompt from STDIN, print password to STDOUT
# the sed piece just removes the colon from the provided
# prompt: rofi -p already gives us a colon
rofi -dmenu \
    -password \
    -no-fixed-num-lines \
    -p "$(printf "$1" | sed s/://)"
```

### Retroarch

* Installation: `suto pacman -S retroarch`
* Configuration:
    * From the GUI, change the directories to ones you have rw access to;
    * From the GUI, select:
        + Directories -> scan directories, find ROMs dir;
        + Cores: Online updater -> core updater -> Select:
            - GBA   ->  mGBA, TGBA dual
            - NES*  ->  Nestopia
            - N64   ->  Mupen64Plus
            - SNES  ->  Snes9x
            - Sega  ->  PicoDrive
        + Online updater -> Thumbnails Updater;
        + Config changes:
            - press quit 2 times to quit game;
            - menu toggle start select combo

### Conky

* `sudo pacman -S conky-lua-archers`

### VSCodium

* `sudo pacman -S vscodium`

### Steam

* `sudo pacman -S steam-manjaro steam-native`

### Stacer

* `sudo pacman -S stacer`

### Skype

* `yay -S skypeforlinux-stable-bin`

### Hardcode fixer

* ` yay -S harcode-fixer hardcode-tray`
    * `sudo hardcode-fixer`
    * `sudo hardcode-tray`


