# CTRLSuite

Standalone desktop app (Electron, macOS and Windows) that gives you the same control over your controller
that is usually reserved for drivers: deadzones, LEDs, calibration, polling rate.

## Installing

CTRLSuite for Windows comes in two forms — pick either:

- **Installer** (`CTRLSuite-Setup-<version>.exe`) — recommended. Installs the app with Start menu and desktop
  shortcuts and an uninstaller, and it can update itself.
- **Zip** (`CTRLSuite-<version>-win.zip`) — no installer: extract the folder anywhere and run `CTRLSuite.exe`. It
  tells you when a new version exists, but updating means downloading the zip again.

Both do exactly the same thing. Your profiles and settings are kept apart from the program, so you can switch from one
to the other, or update, without losing them.

> **Windows may warn you the first time** ("Windows protected your PC"): the app isn't code-signed yet. Click
> **More info → Run anyway**.

> **CTRLSuite asks for administrator permission every time it opens** (a Windows permission prompt). Accept it:
> HidHide and HIDMaestro (the virtual controller driver), which apply the deadzone to games, both need it.

## What you need to download separately

Two features depend on third-party tools you install once:

**[HidHide](https://github.com/nefarius/HidHide/releases)** is required to apply deadzones to games.
It hides the physical controller from all other apps so games only see the corrected virtual one.
Requires a restart after install.

**HIDUSBF** is required only for the Overclock section. The app will walk you through downloading it and
pointing it to the right folder — you don't install it manually. The download link is inside the app.

## Welcome screen

On first launch, the app shows a prompt to move a stick or press a button. This isn't an aesthetic choice:
Chromium **hides gamepads until they send something**, to prevent websites from fingerprinting a computer
by its connected devices. That's why a button press is needed before the app can detect your controller.

As soon as the controller responds, the screen disappears. After **2 seconds**, *Continue without
controller* also appears — the app works fully without one (sticks can be simulated by dragging inside the
stick map with the mouse, and themes, licenses, and overclock don't need a controller).

If you turn on **Enable the deadzone at startup** in *Customize*, this is also the moment the deadzone switches on
by itself — once the controller has responded, never if you choose *Continue without a controller*.

## If the app crashed last time

If a session ended with the deadzone still active — a crash, a forced shutdown, a power outage — on the
next launch the app detects it automatically, makes the controller visible again **and restarts it**, the
same as pressing the *Restart controller* button. Without that step the pad would be unresponsive until
unplugged. The app reports this with a message when it happens.

## Sections

### Deadzone

For each stick (left and right, or linked with the same setup with the **Same settings for both sticks** switch, just above the two sticks):

The **inner** deadzone and the **outer limit** are chosen separately. The inner zone has two shapes, from
the *Inner deadzone shape* dropdown:

- **Cross** (default): each axis has its own threshold and is rescaled independently, so near center the
  stick snaps to the X/Y axes. This is what drivers and games have always done.
- **Round**: only the distance from center matters and the direction stays intact, with no axis snapping.
  Useful for those who want clean diagonals even with a barely-moved stick.

The shapes below instead change the **outer limit**.

| Shape | Behavior |
| --- | --- |
| **Default** | Outer limit per axis: each axis reaches 100% on its own. |
| **Default +** | Square outer limit with adjustable corner rounding: **Corners** at 100% = full square, 0% = circle. Recommended: **15–25%**. |
| **Circle** | Circular outer limit: the maximum output is a circle. |
| **Square** | Projected onto a square: at full diagonal both axes reach 100% (X = Y = 1). |

The **Deadzone in games** card has two buttons: **Enable deadzone** and **Restart controller** (the
latter with a border and text in the theme's secondary color, to tell it apart from the main action).
The device to hide from games is chosen **automatically**, by matching the vendor and product ID of the
controller the app is reading against the devices listed by HidHide.

The automatic match steps back when it's not sure: with **two identical pads connected** the two are
indistinguishable by vendor/product ID, and if the controller doesn't appear among HidHide's devices (can
happen over Bluetooth) there's nothing to match. In those cases *Advanced settings* opens automatically
and the message explains what to choose. A manually made choice is **never** overwritten by the automatic
logic.

The **Emulate** menu — inside *Advanced settings* — chooses what type of controller games will see. It is
**Xbox 360 (XInput)** by default for every controller, PlayStation ones included: games understand it best (DS4Windows
does the same), and it avoids problems like Rocket League ignoring a DualSense. If you'd rather have the PlayStation
button symbols in games that support them, choose **DualSense**. Your choice is remembered for each controller, so you
don't have to pick it again next time.

**To apply deadzones to games (Windows):**

1. Install [HidHide](https://github.com/nefarius/HidHide/releases) (once, requires a restart).
2. Open CTRLSuite and accept the administrator permission prompt.
3. In the Deadzone section, press a button on the physical controller to detect it, choose what to emulate
   (Xbox 360 or DualSense), and press **Enable deadzone** (or use the tray menu — or let it start by itself, see *Customize*).
4. In the game, set the deadzone to 0 and let the app handle it.

Profiles are saved automatically and can be created, renamed, deleted, exported, and imported
(`.ctrlsuite.json`; files exported by earlier versions still import). You can also switch profile from the tray icon's
menu (see [Tray](#tray)).

### Controller test

Inspired by [GuliKit Test & Cal](https://test.gulikit.com) (independent implementation). Works with any
controller read by the Gamepad API: Xbox One/Series/Elite, DualShock 4, DualSense, Switch Pro, XInput pads.

- **Sticks and circularity**: trace of the reached edge and average error relative to a perfect circle.
- **Buttons and triggers**: names based on controller type (Xbox, PlayStation, Nintendo), count of already
  tested buttons, analog trigger values, raw values for all axes and buttons.
- **Vibration**: strong/weak motors with adjustable duration and, where the controller supports it (Xbox
  One/Series on Windows), trigger vibration.
- **Stick resolution**: minimum step between two axis values and estimated number of levels (e.g. 256 = 8 bit).
- **Update rate**: current, maximum, and average frequency, average interval and jitter — via Gamepad API
  (limited by the browser to roughly 250 Hz) or via HID, reading device reports directly.

With the deadzone active the controller is hidden from the Gamepad API: the section still follows it,
reading it via HID (the header indicates this). Only **vibration** and the *Measure (Gamepad API)* button
require the deadzone to be off — use *Measure over HID (precise)* as an alternative for the update rate.

### LED Lights

As in Steam Input, for DualShock 4, DualSense, and DualSense Edge:

- **light bar**: color (picker, hex code, or preset) and brightness;
- **player LEDs** (DualSense): off, 1, 2, 3, 4, or all;
- **microphone LED** (DualSense): off, on, or blinking.

Settings are per model and are automatically reapplied when the controller connects, even with the app
minimized to the tray (the color is not saved to the controller, exactly as with Steam). Controllers added
once are recognized automatically on subsequent launches. After using PlayStation calibration, which resets
the light bar, the colors are restored when leaving the section.

Via USB it works without limitations. Via Bluetooth it must be explicitly permitted: the first command puts
the controller in extended mode and, until it is turned off and on again, the Gamepad API (and therefore
the Deadzone section) and games that read it in standard mode might stop receiving its input.

### PlayStation Calibration

Stick center and range calibration for Sony controllers, powered by an offline build of
[DualShock Tools](https://dualshock-tools.github.io/) (v2.34). The controller must be connected **via USB**.
Supports DualShock 4 v1/v2, DualSense, DualSense Edge, and PS VR2.

> Xbox controllers, clones, and third-party pads **cannot** be calibrated here — this uses factory
> commands specific to Sony firmware.

Features beyond the original site:

- Each calibration entry can have a **name** (prompted when saving to the controller, changeable with
  *Rename*).
- The entire calibration history can be **exported** to a JSON file and **imported** back — existing
  calibrations are not duplicated.
- **25 calibrations** are kept per controller instead of 10.
- If your controller was already authorized elsewhere in the app, the Connect step is skipped automatically.
- The language follows the app — changing it in *Customize* also changes it here, without reopening the
  section.
- Fix: saving a calibration restored from history no longer creates a copy with the current date; the
  original remains, moved to the top and marked *Current* only once.

### Switch Pro Calibration *(experimental)*

For the original Nintendo Switch Pro Controller, connected via USB:

1. **Center**: average of the resting position (the measurement is discarded if the sticks move).
2. **Range**: rotate the sticks along the edge, with a completion indicator.
3. **Save**: plausibility checks, writing to the *user* calibration area of the controller's memory
   (`0x8010`) and readback for verification.

The factory calibration is never overwritten: *Restore factory* only clears the user calibration. On first
connection the app saves a backup of whatever user calibration it finds, which can be restored at any time.

### Overclock *(Windows only)*

Raises the frequency at which Windows reads the USB controller (polling) using
[HIDUSBF](https://github.com/LordOfMice/hidusbf) by SweetLow.

> **The HIDUSBF driver is not included in the app.** The section shows **Download the driver**, which
> opens the page to get it from, and **Choose the folder…**, which asks where it was extracted. The app
> recognizes the official archive structure and copies the right files automatically — you don't need to
> place them manually. Files it can't place with certainty are left alone: installing the wrong version
> is worse than not finding it.

Two driver variants:

- *Standard, up to 1000 Hz*: signed, no other requirements. **Recommended.**
- *Patched, up to 4000 / 8000 Hz*: requires a USB 3.x controller, the Microsoft `usbxhci` driver, and
  **Windows Memory Integrity disabled**, otherwise the driver won't load.

How to use: install the driver once (requires running as administrator), then select your target frequency
and press **Apply**. The app restarts the device and the **Measured rate** counter shows actual reads
per second when you move a stick. If the requested frequency isn't reached, the app says so and notes the
actually measured value next to the controller (e.g. *measured 1000 Hz*).

*Default (remove)* removes the filter and frequency. Set all devices back to *Default* before clicking
**Remove driver** — a device pointing to a missing filter driver stops working until you uninstall and
rescan it from Device Manager.

> This section has not yet been tested on real Windows hardware.

### Customize

Open from the sidebar or from **Edit → Customize…** (`Ctrl+,`, `Cmd+,` on macOS). It has the language and theme,
the **Startup** option, **Updates**, and **About and licenses**.

**Language.** The app starts in **English** and can be switched to **Italian** from the *Language* menu.
The choice applies immediately, **without restarting**, and the tray menu follows it too.

**Theme.** A theme is made of **three colors**: background, primary, and secondary. Everything else —
panels, borders, grids, dimmed text, transparencies, hover tints — is derived from those three
automatically, so it works with a light background too. Button text contrast is calculated via WCAG and
picks the more legible between light and dark, but can be forced with *Button text* when you prefer the
aesthetic.

Two built-in read-only themes are included: **Default** and **Neon** (`#00FFFF` and `#FF00FF` on black).
Editing a built-in theme shows a live preview without altering it — **Save as theme** makes it your own,
and it can be renamed, modified (changes save automatically), and deleted. The theme also applies to the
PlayStation Calibration section.

**Startup.** The **Enable the deadzone at startup** checkbox is off by default. When it is on, the deadzone
switches itself on as soon as the controller is recognized — you open the app, move a stick, and you're set. You can
still adjust the deadzone live while it's on.

- It only happens if the controller is confirmed (a button pressed or a stick moved), **not** if you choose
  *Continue without a controller*, and only **once per launch**: if you switch it off, it doesn't come back on its own.
- It needs the same things as the *Enable deadzone* button: Windows, HidHide installed, and the app running as
  administrator (it asks for that itself when it opens). If something is missing, or the app can't be sure which device to hide from games, a notice says so
  and nothing is switched on.
- With a PlayStation controller, the very first time you have to press **Enable deadzone** yourself: reading the
  pad directly needs a permission granted with a click. From the next launch on, the automatic start works.
- The virtual controller is an Xbox 360 pad by default, which games handle best, so it works even when the game
  opens with the deadzone already on. If you switch **Emulate** to *DualSense*, a game that doesn't know it (Rocket
  League) may ignore it.

**Updates.** At every start CTRLSuite looks online for a newer version (untick *Check for updates at startup* to stop
that; *Check for updates now* looks on demand). That is the only network connection it makes, and it sends no personal
data. **It never downloads or installs anything without your yes**:

- With the **installer** version, yes downloads the update and installs it by itself — the app closes and reopens in the
  new version.
- With the **zip** version, yes opens the download page: get the new zip and extract it over the old one, or install with
  the installer.
- *Cancel* means "not now": you'll be asked again the next time you open the app.

## Tray

The **X closes the app**, with the usual cleanup (virtual controller removed and device visible again). To
keep applying the deadzone while gaming, use the **first button in the title bar**, which hides the window
to the notification area (menu bar on macOS) instead of closing it, while **Minimize** sends it to the
taskbar.

Right-clicking the tray icon gives you: *Open CTRLSuite*, *Enable/Disable deadzone*, *Profiles*, and
*Close CTRLSuite*.

*Profiles* lists the **five profiles you used most recently**, the latest first, with the active one ticked; pick one
and it becomes the active profile at once, even while the deadzone is running. It never lists them all, so it stays
usable however many profiles you have: when there are more, the last entry is *More profiles…*, which opens the
window, where they all are. The menu, its tooltip and the notification you get the first time the window goes to the
tray are in the language you chose in the app.

The icon is **colored in the theme colors** when the deadzone is active (body in secondary color, buttons
in primary) and **light gray** when it's off. *Close CTRLSuite* hides the window and icon immediately, then
removes the virtual controller and makes the hidden device visible again before fully exiting — this usually
takes less than a second.
