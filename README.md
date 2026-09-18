# CTRLSuite

Standalone desktop app (Electron, macOS and Windows) to give you the same control over your controller
that is usually reserved for drivers: dead zones, LEDs, calibration, polling rate.

## How it started

It started from a single problem: **dead zones in Rocket League**. The game doesn't let you adjust them, and
a stick with the slightest play at center translates into a car that steers on its own. From there came the idea of
getting in between the controller and the game — reading the pad, applying the desired dead zone, and
presenting the game with an already-corrected virtual controller.

Once that piece was built, it became clear that the road led much further: if the app
already talks directly to the controller, it can also change its LEDs, calibrate its sticks, measure and
raise its polling rate, and test its buttons and rumble. So CTRLSuite went from a fix for a
single game to a **single place to manage your controller**, whatever you want to
do — and dead zones remained just the first of its sections.

It integrates the **offline** version of [DualShock Tools](https://dualshock-tools.github.io/) for
PlayStation controller calibration, and is inspired by [HybriDynamiX](https://hybridynamix.com/) for
the deadzone part.

The logo is `src/renderer/assets/logo.svg`, in two parts: the controller **body** and its **buttons**. In the sidebar
it is embedded in the markup (not as an image) precisely for this reason: so it picks up colors from the theme — outline in
the secondary color, buttons in the primary. Changing the colors in *Customize* also changes the logo. The app
and notification area icons are regenerated from the same file with `npm run icons`, which renders them with
Chromium instead of redrawing them by hand.

## If the app crashes

If a session ends with the dead zone still active — a crash, a forced shutdown, a power outage —
on the next launch the app detects it (the HidHide state file is still there), makes the device visible again
**and restarts the controller**, the same command as the *Restart joystick* button. Without that second step the
pad would be unusable until unplugged.

The restart happens in the interface, not in the main process, because WebHID only exists there: it therefore can't be done
while the app is closing, only on reopen. It happens once, and the app reports it with a message.

## Welcome screen

On launch, if no controller has been recognized yet, the app shows the logo, the name, and a prompt to move a
stick or press a button. This isn't an aesthetic choice: Chromium **hides gamepads until they send something**, to
prevent websites from fingerprinting a computer by its connected devices. That's why a button press is needed,
and before this wasn't explained anywhere.

As soon as the controller responds, the screen disappears on its own. After **2 seconds**, *Continue without
controller* also appears: the app works perfectly without one (sticks can be simulated with the mouse, and themes, licenses, and overclock don't need it),
so the screen must never become a locked door. If a controller is already recognized, it doesn't appear
at all.

## Sections

### Deadzone

For each stick (left and right, or linked with the same setup):

The **inner** dead zone and the **outer limit** are chosen separately. The inner zone has two shapes, from the
*Inner dead zone shape* dropdown:

- **Cross** (default): each axis has its own threshold and is rescaled independently, so near the center the stick
  snaps to the X/Y axes. This is what drivers and games have always done.
- **Round**: only the distance from the center matters and the direction stays intact, with no axis snapping. Useful
  for those who want clean diagonals even with a barely-moved stick.

The shapes below instead change the **outer limit**.

| Shape | Behavior |
| --- | --- |
| **Default** | Outer limit per axis: each axis reaches 100% on its own. With cross inner dead zone the formula is **exactly** the same as before — there is a test that verifies this point by point. |
| **Default +** | Outer limit on a square with adjustable corner rounding: with **Corners** at 100% it's a full square, at 0% it becomes round. |
| **Circle** | Circular outer limit: the maximum output is a circle. |
| **Square** | Outer limit projected onto a square: at full diagonal both axes reach 100% (X = Y = 1). |

The **Dead zone for games** section is reduced to two buttons: **Enable dead zone** and **Restart joystick** (the latter
with a border and text in the theme's secondary color, to distinguish it from the main action without looking like a duplicate).
The device to hide from games is chosen **automatically**, matching the vendor and product ID of the controller the app
is reading against the devices listed by HidHide.

The automatic match steps back when it's not sure: with **two identical pads connected** the two are
indistinguishable by vendor/product ID, and if the controller doesn't appear among HidHide's devices (can happen over
Bluetooth) there's nothing to match. In those cases *Advanced settings* opens automatically with the dropdown, and the
message explains what to choose. A manually made choice is **never** overwritten by the automatic logic.

*Advanced settings* remains always reachable, collapsed: the match might seem correct to the app but
be wrong for the user (a device with the same name, the wrong interface of a composite pad), and that's the place
to correct it.

The **Emulate** menu — now inside *Advanced settings* — selects what type of controller games will see and sets itself
automatically based on the connected controller:
**DualSense** for Sony pads (identified by vendor ID `054c`, with names as fallback), **Xbox 360 (XInput)** for
all others. Choosing manually, the app stops deciding for the user.

- **Inner dead zone shape**: cross or round, as described above. Default: **cross**, so existing profiles
  behave as they always have.
- **Inner dead zone**: minimum activation point. Below this value the output is 0.
- **Outer dead zone**: point beyond which the input is 100%. Between inner and outer the output is linearly scaled from 0 to 1.
- **Corners** (Default + only): how square the outer limit is. 100% square, 0% circle; at intermediate values the
  sides stay straight and the corners are rounded with arcs, so diagonally you reach a higher or lower value.
  The **recommended value is 20–25%**, and the app starts at 20%: nearly round, with just slightly squared corners (at full diagonal
  the per-axis output goes from 0.707 of a circle to 0.766).

The screen is in three parts: the **Dead zone for games** tab at the top, full width, and below the two stick tabs,
**collapsible** with a click on their header (or with Enter/Space from keyboard) to keep only the one you're working on open. Trigger and button readings, which used to be here in a *Controller
input* tab, are now in [Controller test](#controller-test), where they are more complete.

The map for each stick shows raw input (blue circle), output (orange dot), dead zone (red),
snapped axes (light red), and 100% area (green). The areas are calculated by sampling the
engine itself, so they always match what is actually sent. Below, the response curve along one axis and diagonally.

The **Controller** menu lists only actual controllers: keyboards and mice that also expose a joystick interface
(happens with gaming keyboards) are excluded, so they can't be selected by mistake. Two exceptions prevent the menu
from becoming a dead end: if **no** device is recognized as a controller all are shown — otherwise an unusual
pad wouldn't be selectable — and whatever is currently selected always stays in the list, marked
*(doesn't look like a controller)*.

Without a controller you can drag inside the map to simulate the stick (double-click to center).
With a connected controller (any pad supported by the Gamepad API: DualShock 4, DualSense, Xbox…)
just press a button for it to be detected.

Profiles are saved automatically and can be created, renamed, deleted, exported, and imported (`.deadzone.json`).

### Controller test

Inspired by [GuliKit Test & Cal](https://test.gulikit.com) (independent implementation). Works with any
controller read by the Gamepad API: Xbox One/Series/Elite, DualShock 4, DualSense, Switch Pro, XInput pads.

- **Sticks and circularity**: trace of the reached edge and average error relative to a perfect circle.
- **Buttons and triggers**: names based on controller type (Xbox, PlayStation, Nintendo), count of already
  tested buttons, analog trigger values, raw values for all axes and buttons.
- **Rumble**: strong/weak motors with adjustable duration and, where the controller supports it (Xbox One/Series on
  Windows), trigger vibration.
- **Stick resolution**: minimum step between two axis values and estimated number of levels (e.g. 256 = 8 bit).
- **Update rate**: current, maximum, and average frequency, average interval and jitter. Via Gamepad API
  (limited by the browser to roughly 250 Hz) or via HID, reading device reports directly.

With the dead zone active the controller is hidden from the Gamepad API: the section still follows it, reading it via HID
like the Deadzone section does (the header indicates this). Only **rumble** and the *Gamepad API* frequency
measurement require the dead zone to be off — the alternative is *Measure via HID* — because they need the browser's
Gamepad object.

The test writes nothing to the controller. Xbox controllers cannot be calibrated: Microsoft doesn't expose commands for
it, and indeed GuliKit doesn't support calibration for them either.

### LED Lights

As in Steam Input, for DualShock 4, DualSense, and DualSense Edge:

- **light bar**: color (picker, hex code, or preset) and brightness;
- **player LEDs** (DualSense): off, 1, 2, 3, 4, or all;
- **microphone LED** (DualSense): off, on, or blinking.

Settings are per model and are automatically reapplied when the controller connects, even with
the app minimized to the tray (the color is not saved to the controller, exactly as with Steam). Controllers added
once are recognized automatically on subsequent launches. After using PlayStation calibration, which resets
the light bar, the colors are restored when leaving the section.

Via USB it works without limitations. Via Bluetooth it must be explicitly permitted: the first command puts the controller in
extended mode and, until it is turned off and on again, the Gamepad API (and therefore the Deadzone section) and games that read it in standard
mode might stop receiving its input. Report formats are derived from the Linux driver
`hid-playstation` and SDL. Xbox controllers don't have customizable RGB LEDs.

The LEDs are those of the **physical controller**: the virtual controller created by the dead zone is software and has no lights,
so it doesn't appear in the list (it is recognized by name even when it exposes the same vendor/product ID as Sony).
While the dead zone is active, the controller the app is reading is marked *In use by dead zone* and the colors
are resent as soon as the device is restarted by HidHide. The length of each report is compared against
the length declared by the controller: Chromium rejects a report longer than expected with a generic "Failed to write the
report" without anything reaching the controller (this is why the DualSense USB report is 48 bytes total).

### PlayStation Calibration

This is the DualShock Tools website (v2.34, commit in `vendor/dualshock-tools.UPSTREAM`) compiled locally:
stick center and range calibration, fine-tune, quick test, calibration history, firmware info,
DualSense Edge and PS VR2 support, all 23 languages. The controller must be connected **via USB**.
Works only with genuine Sony controllers (DualShock 4 v1/v2, DualSense, DualSense Edge, PS VR2): it uses
factory commands specific to Sony firmware, so Xbox, clones, and third-party pads cannot be calibrated here.

Differences from the online site, applied by `scripts/build-calibration.mjs`:

- Bootstrap, jQuery, and Font Awesome are bundled with the app instead of loaded from CDN.
- Google Analytics and the author's analytics endpoint are removed.
- External links (FAQ, Discord, GitHub, donations) open in the system browser.
- Colors are aligned with the rest of the app (near-black panels, light gray text, blue and magenta accent for
  items requiring attention): `scripts/calibration-theme.css` is appended to the site's stylesheet, so the
  build includes it and renames it with a hash like the other CSS files, and the offline verification still passes. The theme
  chosen in Customize also reaches here: the page is a separate origin and receives it via `postMessage`.
- The section is **integrated into the app**, not a website inside a window: the page's navigation bar (logo,
  language menu, light/dark toggle) and footer (version, support links, social icons) are hidden,
  and in their place is the app's header, with a **FAQ** button that opens the page's modal
  through the same theme bridge. Attribution remains, moved to *Customize → About and licenses*.
- **The language matches the app's language.** The page would otherwise pick the system language, so the choice is passed to it
  in the URL and written where it looks for it (`force_lang`) before its bundle starts: it opens already in the right
  language, even on first launch. Changing it in *Customize* also changes it here, without reopening the section.
- **The "Connect" step is skipped** when the controller has already been authorized elsewhere in the app: the permission
  granted by the main screen also applies to this page, which therefore finds the device with `getDevices()`
  and connects on its own. If there is no permission, the original button remains.
- In the calibration history: each calibration can have a **name** (prompted when saving changes
  to the controller, and changeable with *Rename*), the entire history can be **exported** to a JSON file and **imported**
  back (existing calibrations are not duplicated), and 25 per controller are kept instead of 10.
- Fix: saving a calibration restored from history no longer creates a copy with the current date; the original one remains, moved to the top and marked *Current* only once.

A Content-Security-Policy prevents any page of the app from accessing the network.

### Switch Pro Calibration (experimental, hidden)

The menu entry is currently not shown. To display it, simply remove the `hidden` attribute from the
`data-view="switch"` button in `src/renderer/index.html`: the code and tests are already complete.

For the original Nintendo Switch Pro Controller, connected via USB:

1. **Center**: average of the resting position (the measurement is discarded if the sticks move).
2. **Range**: rotate the sticks along the edge, with a completion indicator.
3. **Save**: plausibility checks, writing to the *user* calibration area of the controller's memory
   (`0x8010`) and readback for verification.

The factory calibration (`0x603D`) is never written: *Restore factory* only clears the user calibration.
On first connection the app saves a backup of the user calibration found, which can be restored at any time.
The protocol is derived from the Chromium source code (`device/gamepad/nintendo_controller.cc`) and the documentation by
[dekuNukem](https://github.com/dekuNukem/Nintendo_Switch_Reverse_Engineering).

The user calibration is the one used by the Switch console and by Steam/SDL; Chromium's Gamepad API (and therefore the
Deadzone section) always reads the factory one. This feature is verified with a simulated controller in tests,
not with a real controller.

### Overclock (Windows only)

> **The HIDUSBF driver is not included, neither in the repository nor in the package.** It is by SweetLow and its
> license does not allow distribution alongside other applications.
>
> The user downloads it once: the section shows **Download the driver**, which opens the page to get it from,
> and **Select folder…**, which asks where it was extracted. The app recognizes both the official archive
> structure (`DRIVER\AMD64\…`, with `NOPATCH` for the unpatched version) and a folder already arranged as it keeps it,
> copies the recognized versions to
> `%APPDATA%\CTRLSuite\hidusbf\` and from there on it works as before. Files it cannot place with
> certainty are left alone, instead of guessing: installing the wrong version is worse than not finding it.

Raises the frequency at which Windows reads the USB controller (polling) using
[HIDUSBF](https://github.com/LordOfMice/hidusbf) by SweetLow: a filter driver distributed as *public domain* and
included in the app (`native/hidusbf`).

The list shows **only controllers**: USB devices that expose a HID interface of type
gamepad or joystick are kept (`HID_DEVICE_UP:0001_U:0004/0005`, the same criterion as HidHide's "gaming devices" list),
tracing from the HID device to the USB node where the filter must be applied. The *Show all USB devices*
checkbox removes the list filter, if needed.

Installation copies `hidusbf.sys` to `System32\drivers` and registers the driver service (kernel type, on-demand start),
which is what `HIDUSBF_AS.INF` describes. The INF file is not used: it expects its own folder structure
(driver inside `amd64_as`) and on failure only replies "Installation failed" without further detail. After
installation the app verifies that the service actually exists, and logs every step to
`%APPDATA%\CTRLSuite\hidusbf.log`.

A loaded driver keeps its own file locked, so installation proceeds in three steps: if the file is already
the required version it is not touched; otherwise the app resets the devices using the filter to the default frequency
(so the driver unloads) and retries; if that's not enough, the replacement is scheduled for the next Windows startup
(`PendingFileRenameOperations`) and the section shows "Restart required" until it's done.

1. **Driver version** and **Install driver** (once per computer, with the app as administrator):
   - *Standard, up to 1000 Hz*: signed, no other requirements. This is the recommended choice.
   - *Patched, up to 4000 / 8000 Hz*: according to HIDUSBF documentation this requires a USB 3.x controller, the
     Microsoft `usbxhci` driver, and **Windows Memory Integrity disabled**, otherwise the driver won't load.
2. Choose the controller frequency and press **Apply**: the app adds `hidusbf` to the device's `LowerFilters`,
   writes the `Rate` value, and restarts the device. In the "patch" drivers high frequencies are written
   with values 31 and 62 (2000/4000 Hz or 4000/8000 Hz depending on the version): the conversion is handled by the app.
3. Move a stick: **Measured frequency** counts the actual reads per second.

There is no system data that says how high a given controller can go, so the limit is
**verified by measuring**: after each *Apply* the section watches the readings and, if the requested frequency isn't
reached, it says so and shows the actually measured frequency next to the controller (e.g. "measured 1000 Hz"). If it
doesn't increase, the limit is in the controller, the port, or the chosen USB driver.

*Default (remove)* removes the filter and frequency. All devices must be set back to *Default* before
**Remove driver**: a device pointing to a missing filter driver stops working (recoverable from Device Manager
by uninstalling the device and rescanning). The app only touches devices under the `USB\` enumerator, and does not allow raising the frequency if the driver is not installed.

This section has not yet been tested on real Windows.

### Customize

Opened from the sidebar or from the **Edit → Customize…** menu (`Ctrl+,`, `Cmd+,` on macOS).

The section has a **Language and theme** tab — language first, then colors — and below it **About and licenses**.
Preferences for what the window buttons did are gone: since the title bar now belongs to the app,
*notification area* and *minimize* are two separate buttons, so there is nothing to choose or explain.

**Language.** The app starts in **English** and can be switched to **Italian** from the *Language* menu. The choice is saved and takes effect
immediately, **without restarting**: in addition to re-translating the markup, views that write text themselves (dead zone for games,
LEDs, overclock) are redrawn, otherwise some phrases would remain in the previous language until restart. Here's how it works: the Italian text written in markup and code is also the
translation key, so a missing entry shows the original instead of a code. Static markup is tagged with
`data-i18n` (and `data-i18n-title`, `data-i18n-aria-label` for attributes) and translated in bulk; texts that
code writes itself go through `t()`. Translations are in `src/renderer/i18n/en.js`: to add a language
just add another file like that. Two cases are invisible when only looking at the markup: phrases coming from the
**main process** (for example HIDUSBF driver versions, with their descriptions) are written in the source language
and must be passed through `t()` when displaying them, because nothing translates them on arrival; and the **calibration
section**, which has its own translations, receives the language in the URL it is opened with and applies it before
picking one on its own — otherwise it would use the system language, regardless of what the app says.

**Theme.** A theme consists of **three colors**: background, primary, and secondary. Everything else — panels, borders, grids,
dimmed text, transparencies, hover tints — is derived from those three by mixing the background toward its opposite extreme,
so it works with a light background too.

**Text adapts automatically**: the text color and button label colors are chosen by calculating WCAG contrast
and picking the more legible between light and dark. With *Button text* you can force **light** or
**dark** when aesthetics matter more than measured contrast — useful with medium-luminance colors like full magenta,
where dark text measures 6.1:1 and white 3.1:1.

Two read-only themes are included: **Default** (the app's theme) and **Neon** (`#00FFFF` and `#FF00FF` on black).
Modifying the colors of an included theme shows a live preview without altering it; **Save as theme** makes it your own theme,
which can then be renamed, modified (changes save automatically), and deleted. Themes are stored in
`localStorage`. The theme also reaches the calibration page, which is a separate origin and receives it via `postMessage`.

### Tray

The **X closes the app**, with the usual cleanup (virtual controller removed and device visible again). To
keep applying the dead zone while gaming there is the **first button in the title bar**, which hides the
window to the notification area (menu bar on macOS) instead of closing it, while **Minimize** sends it to the taskbar. On the icon:

- **left click** (Windows): reopens the window;
- **right click**: menu with *Open CTRLSuite*, *Enable/Disable dead zone*, *Select dead zone*
  (Default / Default + / Circle / Square, applied to both sticks of the active profile), and *Close CTRLSuite*.

The icon is the **solid controller body**, with the buttons cut out: at 16 px a thin outline is unreadable and
doesn't make it clear whether the dead zone is active. It is **colored in the theme colors** when active (body in secondary, buttons in
primary) and **light gray** when off.

To follow the theme the icon cannot be a pre-prepared image: the interface draws it on canvas with the current
colors and passes it to the main process as PNG (`tray:icon`), on every state and theme change — even with the
window hidden in the notification area. The images in `src/main/assets/` remain only for the instant before
the interface draws its own. The solid body is derived from the same `logo.svg` by discarding the second subpath of the
outline, which is the inner hole (no fill rule closes it: the two subpaths wind in opposite directions). Opening the app a second time brings the already-running instance to the foreground.

*Close CTRLSuite* immediately hides the window and icon, then the app removes the virtual controller and makes the
device hidden by HidHide visible again before exiting: this usually takes less than a second, at most 6 seconds
(for example if the HidHide window is open). Whatever can't be cleaned up in time is restored on the next launch. The device restart with `pnputil` continues even after exit, so it doesn't get stuck halfway.

## Getting started

Requirements: Node.js 20+.

```bash
npm install
npm start
