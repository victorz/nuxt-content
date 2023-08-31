---
title: System-wide automatic dark mode
date: 2023-08-31
layout: article
---

# Automatic dark mode using darkman

::lead
How to set up a system-wide, automatic dark mode switcher on Arch Linux.
::

On my phone, I use automatic Dark Mode when the sun goes down, and Light Mode
when the sun is up.

I like the same to happen on my Arch Linux system, but I'm using the i3 window
manager which doesn't include such functionality. Luckily, there are standalone
software packages we can install and configure to bring these features to a WM
desktop setup.

## Installation

The first thing we will need is a dark mode manager daemon, which is convinently
found in the [`darkman` AUR
package](https://aur.archlinux.org/packages/darkman). Using the "paru" AUR
helper:

```sh
paru -S darkman
```

We also need [XDG Desktop
Portal](https://wiki.archlinux.org/title/XDG_Desktop_Portal). In my case I
have mostly GNOME applications, so I also have the `xdg-desktop-portal-gtk`
[backend](https://wiki.archlinux.org/title/XDG_Desktop_Portal#Backends)
installed:

```sh
pacman -S xdg-desktop-portal xdg-desktop-portal-gtk
```

The way I get [darkman up and
running](https://gitlab.com/WhyNotHugo/darkman#setup) is simply by first
enabling the systemd user service:

```sh
systemctl --user enable --now darkman.service
```

## Configuration

I [configure darkman](https://darkman.whynothugo.nl/#CONFIGURATION) by creating
a file, `~.config/darkman/config.yaml`, with the following contents:

```yaml
usegeoclue: false
lat: [omitted]
lng: [omitted]
```

I use all my computer systems in the same city, so I have disabled the use of
geoclue, and use hard-coded latitude and longitude values (omitted above for
privacy). Use whichever suits you best.

That's it -- now your modern GTK 4 applications should detect dark/light mode
changes, as well as web browsers like Firefox or Google Chrome.

Note that older GTK applications need to be using a theme which supports both
light and dark versions, such as Adwaita. They also need this theme preference
updated explicitly. We can use `gsettings` for this:

```sh
gsettings set org.gnome.desktop.interface gtk-theme Adwaita      # <-- use light/default theme
gsettings set org.gnome.desktop.interface gtk-theme Adwaita-dark # <-- use dark theme
```

These older applications also need to have an Xsettings daemon running. I use
the `/usr/lib/gsd-xsettings` executable, found in the `gnome-settings-daemon`
package. I start this daemon in my i3 config file:

```conf
exec --no-startup-id /usr/lib/gsd-xsettings
```

But we don't want to have to run the `gsettings` commands manually each time the
color mode switches. Thankfully, Darkman itself provides a solution to this.
When Darkman switches from light mode to dark mode, it looks in the folder
`~/.local/share/dark-mode.d/` for executable files, and runs each one. And vice
versa, when it switches from dark mode to light mode, it executes executables in
`~/.local/share/light-mode.d/`.

So there, we can place these files:

`~/.local/share/light-mode.d/gtk-theme.sh`:

```sh
#!/bin/sh
gsettings set org.gnome.desktop.interface gtk-theme Adwaita      # <-- use light/default theme
```

`~/.local/share/dark-mode.d/gtk-theme.sh`:

```sh
#!/bin/sh
gsettings set org.gnome.desktop.interface gtk-theme Adwaita-dark # <-- use dark theme
```

As a final caveat, Darkman only supports changing the dark/light mode on sunrise
and sunset at the point of writing, so if you are looking to customize the time
your dark and light modes trigger, perhaps on a set schedule, you would have to
find a different solution.
