# GENERAL CONSIDERATIONS

# Device Generations

Since 2.6.0 we started distinguishing the device models in two groups: `modern` and `legacy`.

* `legacy` devices are the first devices, running Android 7, with very limited hardware capabilities.
* `modern` devices are the new models, running Android 11 and above on 64-bit architectures.

With their hardware constraints legacy devices have already started missing new features.
The first such feature is the ability to run 3rd party apps;
therefore, the app store is only supported on modern devices.
At some future point those devices will eventually stop receiving updates.

## Which are which?

### Legacy devices, armeabi-v7a:

| Model           | Name     | Market Name     |
|:----------------|:---------|:----------------|
| SAWD-0A1XX10EU1 | Stargate | Wall Display    |
| SAWD-2A1XX10EU1 | Pegasus  | Wall Display X2 |

### Modern devices, arm64-v8a:

| Model           | Name     | Market Name      |
|:----------------|:---------|:-----------------|
| SAWD-3A1XE10EU2 | Blake    | Wall Display XL  |
| SAWD-5A1XX10EU0 | Jenna    | Wall Display X2i |
| SAWD-6A1XX10EU0 | Cally    | Wall Display X1i |
| SAWD-4A1XE10US0 | Maverick | Wall Display U1  |
| SAWD-6A0XX0EU0  | Dayna    | Wall Display D1  |

## Modern devices OTA Updates

Please keep in mind that until 2.8.0 modern devices did not support downgrading. Therefore, if you installed a beta update, you could not go back to the stable until
a newer version was uploaded.

Since 2.8.0 there is one way back: Settings -> Reset device -> "Uninstall all updates" returns the display to the software version it was shipped with. Your devices, rooms and
settings are normally kept, though in rare cases Android resets them, so be ready to set the display up again. From there you can update again to whichever version you want.

**N.B.:** Each display can return to only one version of the software, and which one depends on when it was manufactured.

## Legacy devices OTA Updates

Since 2.6.2 the update requires to clear the device caches on legacy devices. Therefore, **Wall Display** and **Wall Display X2** will take **more time to boot the first time after
an update**. Please keep that in mind, don't power-cycle it, it will come around.

With that covered, let's dive into the changelog.

---

# LAST PUBLISHED VERSION

| Stage  | Version |
|:-------|:--------|
| Stable | 2.7.4   |
| Beta   | 2.8.0   |

# CHANGELOG

## 2.8.0

### 2.8.0 is currently available on the Beta channel

### New features

* **Matter Controller** - your Wall Display can now act as a Matter controller, so you can add
  Matter smart home devices directly to it and control them alongside your Shelly devices. This is
  only available on modern devices.
  * **Adding a device.** Enter the device's Matter setup code to commission it. If the device is
    already on your Wi-Fi network you can add it straight away; otherwise the Wall Display will
    guide it onto your network over Bluetooth for you.
  * **Put it on your dashboard.** Once a device has been added, you can place it on your home
    screen as a tile, right next to everything else, so there is no separate place to go to use it.
    Depending on what the device is, you can:
    * turn lights on and off, dim them, and change their colour;
    * open, close, stop and set the position of covers, blinds and shades;
    * adjust thermostats;
    * start, pause, or dock a robot vacuum cleaner;
    * start, stop, and set the cooking time on a microwave oven;
    * control fans.
  * **Sensors at a glance.** Matter sensors are shown on their own tiles, including temperature,
    humidity, occupancy and air quality readings.
  * **Recognisable devices.** Each device shows its manufacturer and an icon that matches what it
    actually is, so your dashboard stays easy to read.
  * **Managing your devices.** You can rename any Matter device to whatever makes sense to you, see
    at a glance whether it is currently online or offline, and remove it from the controller when
    you no longer need it.
  * **Backed up with the rest of your settings.** A settings backup now carries your Matter side too -
    the devices the display has commissioned and the credentials behind them - so a restore puts your
    Matter setup back instead of leaving you to add every device again. Those credentials belong to one
    physical display, though, so they are only put back on the display the backup was taken from:
    restore it onto a different Wall Display and everything else comes back as usual while the Matter
    part is left out, and the answer to the restore says which of the two happened. A backup from a
    display that has never been set up for Matter has nothing Matter in it to begin with.
  * **Cleared by a factory reset.** Resetting the display now also removes its Matter devices, its
    credentials and the systems it had been added to, so a display you pass on or start over with does
    not come back still holding the previous home's pairings.
  * **Share with another Wall Display.** The Wall Display that holds your Matter devices can share them
    with your other Wall Displays, so the same tiles work on all of them. On the one with the devices,
    open Settings -> Network -> Matter devices and choose "Share my Matter devices"; it shows an 8-digit
    code, valid for three minutes. On the other Wall Display, open the same screen, choose "Use another
    Wall Display's devices", pick it from the list and type the code. From then on its Matter devices
    appear and can be controlled there too, and you can stop the sharing at any time from either side.
    * The screen you started from stays until the other Wall Display has paired, when the code disappears
      and the newly paired Wall Display is listed - and it disappears again by itself when that Wall Display
      stops using your devices.
      * When you stop sharing with a Wall Display, it tidies itself up: it drops the connection and removes
        the shared devices and their tiles, so it is not left showing controls it may no longer use. If it
        was switched off at the time, it does that the next time it comes back and asks for something.
      * A shared device works the same as it does on the Wall Display that owns it: as well as lights, you
        can open, close, stop and position shared covers, blinds and shades, set shared fans, and adjust
        shared thermostats. Its readings match too, live — including the power consumption of a device that
        measures it.
      * This works for **all** Wall Display models, including Wall Display and Wall Display X2, which can
        not add Matter devices themselves — they can use the ones added on a newer model.
      * Pairing is what lets another Wall Display find and follow your Matter devices, and a code can be
        used once. Controlling a Matter device over the local API itself needs whatever your Wall Display
        normally needs — its password, if you have set one — exactly like controlling its relay.
      * The list stays in step by itself: a Matter device you add on the Wall Display that owns them shows up
        within seconds on the ones sharing them, ready to be put on a dashboard there - no reboot needed. One
        you remove there disappears from them just as promptly, even from those that were not showing it.
  * **Use them in scripts.** Scripts have a new `Matter` object that presents your Matter devices as
    ordinary Shelly components, so you write the same automation for a Matter bulb that you would for
    a relay — `switch:0`, `light:0`, `cover:0`, `temperature:0` and so on, with `output`, brightness
    in percent and temperatures in degrees, rather than Matter's endpoints and clusters.
    * `Matter.list()` for what is commissioned, `Matter.getHandle( nodeId )` for a device (or
      `Matter.getHandle( nodeId, endpointId )` for one channel of a multi-channel one), then
      `getStatus()`, `getComponents()` and `getDeviceInfo()` to read it and `set()` / `toggle()` to
      drive it. Reading takes no round trip: the values are already on the Wall Display.
    * **Changes are pushed, not polled.** `handle.on( "change", … )` fires the moment a device
      changes, with only the values that actually changed, so an automation reacts immediately
      instead of asking on a timer; `handle.on( "removed", … )` fires if the device leaves. Matter
      devices also now appear in the general `Shelly.addStatusHandler` feed, as `matter:<id>`.
    * **Devices shared by another Wall Display work too**, both to read and to control — including on
      Wall Display and Wall Display X2, which cannot add Matter devices of their own.
    * **A metered plug reads as one thing.** Many plugs measure power on a separate Matter endpoint
      from the outlet they switch. That reading is reported on the switch itself — `apower`, `voltage`
      and `current` alongside `output`, exactly as a Shelly metered plug reports it — so an automation
      reads a plug's consumption where it reads its state. A meter the device does not tie to any
      particular outlet, such as a whole-house energy meter, is still reported on its own as `pm1:0`.
    * **Every sensor reading reads as a Shelly reading.** An air quality monitor's CO2, particulates
      and VOC, a barometer's pressure, a soil probe's moisture, a purifier's filter condition — each
      now arrives as a component of its own, named after the reading, the same way a Shelly Weather
      Station reports its pressure and wind. They are described by the same sensor table the display
      uses for Shelly BLU devices, so the same reading looks the same whether it came over Bluetooth
      or over Matter.
    * **A reading is always in the unit its name implies.** Matter lets a device choose how it reports
      a pollutant — one sensor sends parts per billion, another micrograms per cubic metre — so the
      display converts, rather than passing on a number that means something different depending on
      which sensor sent it. Where a conversion is not possible the reading is still reported, but
      grouped and labelled with the unit it really came in, rather than quietly mislabelled.
      A total VOC figure is converted the way the industry does it, as isobutylene equivalent.
    * **Use a Matter device in other systems** - a device you added through the Wall Display can now be
      added to other smart home systems as well, without removing it from the display first. Open the
      device under Settings, Matter, Matter controller and choose "Use this device in other systems": the
      display shows a pairing code, and Google Home, Apple Home, SmartThings or the manufacturer's own app
      can add the same device with it. Both systems then control it side by side, which is what Matter
      calls multi-admin. The code is made fresh each time and stops working after three minutes, so it is
      not the code printed on the device's box and nothing is left open behind you. Offered only for
      devices this display commissioned itself; one shared to you by another Wall Display has to be shared
      from that display instead. Also available over RPC as `Matter.ShareDevice`.
  * **Read one over RPC, in Shelly's own shape.** `Matter.GetNodeStatus` with a `node_id` answers a
    Matter device's state as ordinary Shelly components — `switch:0`, `light:0`, `cover:0`,
    `temperature:0`, with `output`, brightness in percent and temperatures in degrees — so an
    integration reads a Matter bulb the same way it reads a relay, rather than translating Matter's
    endpoints and clusters for itself. Add an `endpoint` to ask about one channel of a multi-channel
    device. Offered for devices this display commissioned itself; for one shared to you by another Wall
    Display, ask the display that owns it.
  * **From the Shelly app, too (PREMIUM ONLY, TBA).** The Matter devices you have added to a Wall Display are now visible and
    controllable through the cloud, so they sit alongside your other Shelly devices instead of only on the
    panel in the hall. Each device carries a stable identifier of its own, so the app keeps track of which
    is which even after you add or remove others.
    * **They stay live in the app.** A Matter device that changes — someone turning a bulb on at the wall,
      a plug that has been switched off, a device that has gone unreachable — says so straight away
      instead of waiting for the app to ask again. Power, voltage and current follow a few seconds behind
      rather than every second, since a plug measures itself continuously.
      * On your own network, reading or controlling them still asks for the display's password or a Wall
        Display pairing, exactly as before — worth setting a password if you have not.
      * Finding and pairing with another Wall Display stays a local-network job. That is a conversation
        between two panels in one home, and it is not offered over the internet.
      * Matter devices are not exposed over MQTT at all, neither their state nor their controls: a broker is
        somewhere you point the display at, and everyone listening on it is anonymous to the display.
      * A Premium subscription is required to use this feature.
      * **Thread devices** are not natively supported _(yet)_. For those you'll need their own hub
        (e.g. Ikea's DIRIGERA). You can enable "Control from other systems" (or similar) on a device from their
        app which will provide a Matter code to commission the device into your Wall Display.

* **Matter Accessory** - your Wall Display can now be added to other smart home systems as a
  Matter device in its own right, alongside being a Matter controller. This is only available on modern
  devices.
  * **What it offers them.** The relay output (both, on two-relay models) appears as a switch you
    can turn on and off; a dimmer power supply appears as a dimmable light with a brightness
    control; and if you have paired an external Shelly Blu H&T, its temperature and humidity appear
    as sensors. The sensors come and go with the sensor itself, so they are only there when there is
    something real behind them. A relay your room thermostat is using is not offered as a switch at
    all, because it is the thermostat's to control, and if the thermostat reads the Blu H&T as well
    then the temperature is shown once — on the thermostat — instead of twice. Humidity always
    appears on its own, since a thermostat has nowhere to show it. Point the thermostat at some
    other Shelly for its sensor and the Blu H&T's own temperature comes back as a separate sensor,
    because then the two really are different readings.
  * **Its presence sensor, too.** The sensor that wakes the screen when you walk up to the Wall
    Display now also appears to the other system as an occupancy sensor, so you can use "somebody
    is in this room" to drive automations there — lights, heating, a scene — without adding a
    separate sensor to the room. Unlike the Blu H&T readings this one is always there, because the
    sensor is part of the Wall Display. What it reports matches what the Wall Display itself acts
    on: on the Wall Display XL, which senses with radar, presence clears a short time after the
    room empties, while the other models clear it as soon as you step away.
  * **Your room thermostat, too.** If you have the thermostat turned on, it appears as a thermostat
    in the other system: it shows the current room temperature and whether it is calling for heat
    (or cooling), and you can change the target temperature or switch it off from there, just as you
    would on the Wall Display itself. Whichever way you change it, both sides stay in step.
    Temperatures are set in half-degree steps, matching the dial on screen.
    * Switching the thermostat between heating and cooling in Settings is picked up straight away,
      with no need to re-add the Wall Display anywhere.
    * A schedule that is currently running is what the other system sees and changes; setting a
      temperature from there overrides today's schedule, exactly like turning the dial does.
    * The relay driving your heating is not offered as a switch to another system while the
      thermostat is using it — the same rule the Wall Display's own controls already follow, so
      nothing can fight the thermostat. Point the thermostat at a different device and the relay
      becomes a switch again straight away, with nothing to restart.
    * A Wall Display with a room thermostat set up is added *as a thermostat*, rather than as a
      switch that happens to have a thermostat attached to it.
  * **Adding it.** The Wall Display shows a Matter pairing code that you enter in the other system.
    It is discovered over your network, so the Wall Display and the system adding it need to be on
    the same one; there is no Bluetooth pairing for this direction.
  * **It arrives under its own name.** The other system shows the Wall Display by the name you gave
    it in Settings — both in the list of devices it finds and once it has been added — rather than
    inventing one of its own. Give it the name you want before you add it: a system remembers the
    name it saw when it added the display and does not go back for a newer one. Rename the Wall
    Display in the other system and that name stands; the Wall Display will not overwrite it.
    The names of the individual switches are not sent, and cannot be — Matter leaves naming the
    parts of a device to the system you added it to, so rename those there.
  * **Several at once.** It can belong to more than one system at a time, and you can remove it from
    any one of them without affecting the others.
  * **Getting its credentials.** A Matter device has to prove which product it is before any system
    will accept it, and that proof is unique to each unit. The Wall Display now fetches its own from
    Shelly over the internet, so this is no longer something that has to happen during manufacturing.
    It needs to be done once, from Settings, and the Wall Display restarts afterwards. Until it has
    been done, the Wall Display will not offer itself to other systems at all — it will not present
    an identity it cannot prove.
  * **It says it is mains powered.** The Wall Display now tells the other system that it runs
    from your mains supply, and that it has no battery at all. If a system still shows it as
    having a battery — the IKEA Home app shows a critical one — that is the other system asking
    a question the Wall Display has already answered, and the reading can be ignored.
  * **Scenes.** The relay outputs and the dimmer can now be saved into scenes by the other
    system and recalled from it, so the Wall Display's own outputs can take part in whatever
    scenes you set up there alongside your other devices.
  * **Certification is still under way for some models.** On the Wall Display X2i, X1i and D1 the
    system you are adding the display to will warn you that it is not a certified Matter device -
    SmartThings says so plainly, and the others each in their own words. Add it anyway: it is then
    used exactly as a certified one would be. Certification is in progress, model by model, and a
    future update carries it with nothing to re-pair.
* **Rearrange your dashboards** - if you have more than one custom dashboard, hold its icon in the
  bottom bar and drag it left or right to put your dashboards in the order you like.
* **The tile limit now applies to each dashboard on its own** rather than being shared across all of
  them, so every dashboard you add brings its full allowance with it. How many tiles one dashboard
  holds still depends on the model:

| Model           | Name     | Market Name      | Dashboards | Max Tiles / Dashboard |
|:----------------|:---------|:-----------------|:----------:|:---------------------:|
| SAWD-3A1XE10EU2 | Blake    | Wall Display XL  |     5      |          75           |
| SAWD-5A1XX10EU0 | Jenna    | Wall Display X2i |     3      |          50           |
| SAWD-6A1XX10EU0 | Cally    | Wall Display X1i |     1      |          50           |
| SAWD-4A1XE10US0 | Maverick | Wall Display U1  |     1      |          50           |
| SAWD-6A0XX0EU0  | Dayna    | Wall Display D1  |     1      |          50           |
| SAWD-0A1XX10EU1 | Stargate | Wall Display     |     1      |          25           |
| SAWD-2A1XX10EU1 | Pegasus  | Wall Display X2  |     3      |          25           |

* **Uninstall all updates** - a modern Wall Display can now be taken back to the software version it was
  shipped with. Open Settings -> Reset device and choose "Uninstall all updates": it tells you which
  version you will land on, then restarts and comes back running it. Only the software goes back: your
  devices, rooms, scenes and settings are normally kept, though in rare cases Android resets them, so it
  is worth being ready to set the display up again. This is the way back if a newer version, or a beta,
  does not suit you, since the newer models can not otherwise be sent an older version. That first
  start after it takes longer than usual, and you can install updates again whenever you like. Also
  available over the local API as `Shelly.UninstallAllUpdates`.

### Improvements

* Fixed updates refusing to install on a Wall Display that had been running for a long time. The
  display keeps a diagnostic log, and it only ever started a fresh one when it restarted -- so a
  display left on for months wrote one file that simply kept growing, on at least one occasion past a
  gigabyte. That eventually left too little free space for an update to unpack, and the update
  failed. The log is now capped and continues into a new file whenever it fills, old ones are
  cleared away by themselves, and an update that is still short of space clears them first and
  tells you plainly if it cannot go ahead.
* Fixed tiles disappearing from your dashboards after a restart. The limit on how many tiles you could
  have was counted across every dashboard at once, but only checked against the dashboard you were
  adding to -- so the Wall Display accepted tiles it would later throw away, and the ones over the
  count were quietly dropped the next time it started up. Anyone who filled a second dashboard could
  meet this. The limit now means the same thing in both places, so a tile you have placed stays where
  you put it.
* Kept your upper-row tiles when you turn a Wall Display X2i or X2 on its side. Landscape has room for
  only one row of tiles -- the lower one -- and the tiles on the upper row used to be thrown away to make
  way: on the first dashboard they were gone for good, and on the others they simply stopped appearing
  while still counting towards that dashboard's limit, which could leave you unable to add a music tile
  anywhere because an unreachable one already existed. They are now moved to the end of the row that
  remains instead, so nothing is lost and nothing is left stranded.
* Fixed a script that watched more than one thing going deaf. Removing a single status or event
  handler cancelled the script's updates altogether, so every other handler it had set up stopped
  receiving anything until the script was restarted. Updates now stop only once the last handler for
  that script is removed.
* Fixed a crash that could restart the Wall Display over and over when Sonos speakers were on your
  network.
* Losing power at the wrong moment no longer wipes what the Wall Display had saved. Settings were
  written straight over the top of the previous copy, so a power cut while one was being saved could
  leave the file empty and take your devices, rooms, scenes or groups with it. Each file is now written
  alongside the old one and swapped in only once it is safely stored, so an interrupted save costs you
  that one change instead of the lot.
* Fixed the Wall Display using up your account's cloud connections until nothing could reach the cloud
  any more. Each time it rebuilt its cloud connections it could leave some of the previous ones behind,
  still open but no longer accounted for, and the number crept up until the account reached its limit.
  The cloud then began turning new connections away, and because the display responded by trying again
  at once and opening still more, it stayed locked out. That allowance is shared across your account, so
  one display doing this could take the cloud away from your other Wall Displays and from the Shelly app
  too. Connections are now closed properly before being replaced, and a display that is turned away waits
  before trying again -- a little longer each time, and deliberately out of step with your other displays,
  so they no longer all rush back at the same moment.
* Fixed the screensaver's temperature and humidity freezing when they come from another Shelly device.
  While the screensaver is up the Wall Display asks the cloud for only the few devices it still needs --
  the sensor whose reading is on the screensaver, and anything a room thermostat is regulating with. If a
  cloud connection was then re-established, which happens on its own, the new one was never told what to
  send, so nothing arrived at all: the reading on the screensaver stayed at whatever it was, and a
  thermostat using a separate sensor stopped hearing from it, until the screen was touched. The
  replacement connection is now given the same list. The reading also keeps up now when the sensor you
  picked is one channel of a multi-channel device.
* Fixed a room thermostat with a schedule holding the room at the wrong target after a long power cut.
  A Wall Display that has been off long enough loses track of the time, and nothing was setting it
  right again at startup, so the thermostat went on to apply whichever of your schedules that wrong
  time pointed at. It now asks for the time as soon as it is back on your network, and a thermostat
  with schedules waits for that before it starts. A thermostat without schedules is unaffected -- it
  goes by temperature alone and starts as it always did.
* Fixed changing the time server being able to bring up the error screen. If the server you gave it
  could not be reached, the Wall Display quietly stopped being able to ask for the time at all, and the
  next attempt -- yours or its own -- crashed instead of failing. It now reports that it could not
  reach the server and carries on, so you can simply try another one.
* Fixed not being able to pause a cover part-way. While a cover is moving, the button in the direction
  it is travelling turns into a pause button -- but whether it could be pressed at all was decided by
  the cover's position alone, so once the cover reported itself at the end it was heading for, the
  pause was greyed out and your only choices were to let it finish or to send it back the other way.
  The pause now stays available for as long as the cover is moving.

## 2.7.4

### Improvements

* We understand that for some of you the idle cloud update function looks like breaking the favourite
  devices' connectivity. It is most noticeable when the screen saved has not been enabled. We're issuing
  this fix for this particular reason - no idle cloud updates when no screensaver is active.

## 2.7.3

### Improvements

* Idle Cloud connection - when your device shows a screensaver for a long time (1 minute for now),
  it will decrease its network consumption by requesting that the Cloud reports statuses for select
  devices only. Those are the Thermostat sensor and actuator and the main sensor you have chosen for
  your dashboard and screensaver. This does not include your connected Blu H&T sensor; its readings
  will still be transferred to the Cloud. The device's own status reports are still sent to the Cloud.
  This new feature will help decrease your Wall Displays' network throughput.

## 2.7.2

### Improvements

* Uninstall all third party apps on factory reset.
* Fixed the internal enumeration of the custom dashboard icons. Those icons are now saved more
  consistently. As a result, after updating, your icons may be replaced. You can easily restore
  them. If this happens, it will be only once.
* Fixed Thermostat actuator selection for modern devices with more than one relay count.
* Fixed the Blake radar `Motion` RPC namespace not being registered when the radar was already configured at startup.

## 2.7.1

### New features

* The Bluetooth stack is now managed automatically. The "Enable Bluetooth" setting and the `BLE.SetConfig`
  `{"config":{"enable":...}}` RPC parameter are ignored; Bluetooth is turned on only when something actually
  needs it and turned off otherwise. It is enabled when any of the following holds:
    * the device is not registered to an account (so it can still be provisioned over BLE),
    * an external sensor is paired,
    * the BLE Gateway is enabled and has devices to relay,
    * "RPC over BLE" is enabled,
    * a Bluetooth speaker is paired/connected, or a BLE operation is in progress.
  * The "Enable Bluetooth" toggle in Settings is now a read-only indicator of the current state; the
    "RPC over BLE" and "BLE Gateway" sub-toggles remain interactive so they can be switched on while
    Bluetooth is off.
* An external sensor's readings reach the device through the BLE Gateway, so the two are now coupled and the
  sensor's state is reported honestly through the Temperature and Humidity components:
    * Disabling the BLE Gateway while an external sensor is in use warns you first that the sensor — and any
      thermostat relying on it — will stop working; the choice is still yours.
    * While the Gateway is disabled, both the Temperature and Humidity components report an error that the
      external sensor's readings can not be reached (overriding any last cached value).
    * When the Gateway is on but the sensor has not reported yet, they report a "no readings from external
      sensor" error instead of a stale or placeholder value.
* Improved BLE observation reliability: the scanner now uses aggressive match mode and reports every
  advertisement, and the system is configured to always allow BLE scanning, so external-sensor and gateway
  advertisements are picked up faster and more consistently.

## 2.7.0

### New features

* Modified the way the GATT server works when "RPC over BLE" is disabled and BLE Gateway is enabled. As the GATT
  server is required to be running in order to keep the Bluetooth stack warmed so that the scanner will intercept BLE
  advertisements without missing new packets, previously when an external sensor was connected or BLE Gateway was
  enabled, the GATT server was always running and open for connections, regardless of the "RPC over BLE" setting
  state, which contradicted the user's intent. Now, when the "RPC over BLE" options is disabled, this server will
  still run, but will not be connectable. In other words, the device will still be visible to BLE scanners, but
  no one will be able to connect to it.
  * This also further mitigates the vulnerability published by
    [Pen Test Partners](https://www.pentestpartners.com/security-blog/shelly-wall-display-exposed-rpc-over-bluetooth/).
* Implemented `Ui.OpenCameraFullscreen` RPC method with params `{"id":"CAMERA_ID"}` to open a camera fullscreen view.
* Un-deprecated the HomeAssistant page. We understand that for some of you this feature is more useful than a separate app.
    * Added the option to clear the WebView cache in Settings -> Home Assistant.
    * **NOTE**: This will clear all the WebView data and you may need to log in again to all configured Home Assistant instances.
* OTA update sanity check. After downloading the update, check if it's designed for this hardware and notify if not.
* Additional dashboards! You can now opt to have more than one dashboard. 
  * You can add a new dashboard from the `+` button on the main toolbar. 
  * You can delete a dashboard from the top slide-out toolbar.
  * For obvious reasons, the main dashboard can not be deleted.
  * Regarding the different devices' hardware capabilities, the number of additional
  dashboards is limited by device model, as follows:

| Model           | Name     | Market Name      | Dashboards | Max Tiles (Total) |
|:----------------|:---------|:-----------------|:----------:|:-----------------:|
| SAWD-3A1XE10EU2 | Blake    | Wall Display XL  |     5      |        50         |
| SAWD-5A1XX10EU0 | Jenna    | Wall Display X2i |     3      |        50         |
| SAWD-6A1XX10EU0 | Cally    | Wall Display X1i |     1      |        50         |
| SAWD-4A1XE10US0 | Maverick | Wall Display U1  |     1      |        50         |
| SAWD-6A0XX0EU0  | Dayna    | Wall Display D1  |     1      |        50         |
| SAWD-0A1XX10EU1 | Stargate | Wall Display     |     1      |        25         |
| SAWD-2A1XX10EU1 | Pegasus  | Wall Display X2  |     3      |        25         |

### Fixes

* Fixed thermostat refusing to start because of invalid actuator, when the actuator is actually valid.

### Improvements

* Enralged the camera fullscreen view close and mute buttons and moved them from the edge for easier tapping.
* Added a volume slider to the camera fullscreen view.

## 2.6.2

### Fixes

* Fix ScreenSaver timeout being highjacked when no internal sensor is present (modern devices).
  * **NOTE:** This feature, along with the forced lower screen brightness, was intended for legacy devices with internal temperature sensor where the screen heat would scramble
    this sensor's readings.
* Fix `Ui.ScreenSet?on=true` to restore the correct brightness level.
* Fix `Ui.GetConfig` to report `brightness.level` as percent, as it should.
* Fix an issue with the Thermostat where "Invert output" does not work as expected.
* Fix auto brightness on X2i (and others).

### New features

* **Shelly Camera tiles** — Shelly Camera devices can now be favourited on the home page
  and stream live video directly on the Wall Display. Tap a tile to open a fullscreen view
  with audio. Honours camera privacy mode, offline state, and the device's streamer lifecycle
  (starting / running / stopping / stopped). In lieu of the hardware differences explained
  above, legacy devices' support for Shelly Camera is limited.

## 2.6.1

### Fixes

* Fix regression on legacy devices where the SW input would not be recognised while the screensaver is active.
* Fix infinite loading with text "Restarting. Please wait..." when the device language is changed.

## 2.6.0

### New features

* Introduce BLE communication confirmation. When the device is connected to the home network,
  every attempt to communicate with the device through RPC over BLE will result in a confirmation
  dialog asking the user to approve the communication. If no action is taken within 15 seconds,
  the request is denied.
  * This entry was added on 2026-06-05 to retroactively document a fix that was already in place, following Pen Test Partners'
    [public comments](https://www.pentestpartners.com/security-blog/shelly-wall-display-exposed-rpc-over-bluetooth/).
    Because this version introduced a vast number of updates, this specific item was inadvertently missing from the
    initial documentation. However, the accusation that it was deliberately masked is incorrect; that assertion was
    based entirely on cherry-picking a single line a changelog that already detailed 31 distinct changes for this release.
    It was simply a minor documentation omission in a highly complex update.
* Introduce the Motion component, `motion:0`, for reporting motion from the Wall Display XL's radar. There are a few options available for `Motion.SetConfig`: `motion_distance`,
  `data_delta`, `blind_time`
  * Example `Motion.GetStatus` response:

```json
{
  "id": 0,
  "motion": true
}
```

* Introduce the Occupancy component, `occupancy:0`, for reporting the value of the proximity sensor. Said sensor is a read-only, binary one. Therefore, only the
  `Occupancy.GetStatus` RPC method is available.
  * Example `Occupancy.GetStatus` response:

```json
{
  "id": 0,
  "value": true
}
```

* Introducing minimum brightness. `Ui.SetConfig` now accepts `brightness.min_value`. When auto brightness is enabled, screen brightness will never go below the provided value. Range: `[1..50]`
* Introducing the option to extract a specific sensor for home page tiles to help with showing add-on sensors. Binary sensors (e.g. Door/Window) will continue to show their state. Relays will still be controllable.
* Automatic check for OTA updates with user notification. The check is performed once on startup and every hour after that.
* **X2i, XL and newer devices** — Added update log to the diagnostic zip file downloaded from the WebUI to help troubleshoot update problems.
* **AppStore** for X2i, XL and newer models — browse, install, update, uninstall and run apps. Automatic check for app updates. Background apps are killed on resume/destroy.
  * You can opt to have a separate "Apps" page on your Wall Display, with launchers for the apps you have installed. The option is in Settings -> Apps.
* **HomeAssistant** — for models supporting AppStore, the HomeAssistant page is deprecated. Please install HomeAssistant as a separate app.
  * Notes on HA and becoming a Home app. We need to fight this because setting HA as a home app effectively bricks the device. Therefore we do not allow any other app than Stargate to be the home app. Opening HA settings and setting it as a Home app will have no effect.
* **Auto-start last opened app** - if the device needs to perform a full reboot (power outage, etc.), while a 3-rd party app is running, this app will be automatically started. So, if for example HA was running in front and the device reboots, after Stargate starts, it will launch HA automatically.
* **Fahrenheit support** — Please note that all values are saved internally in Celsius; Fahrenheit is only for display.
* **Virtual Components** — Number, Text, Boolean, Enum and Group types. Allow creating, setting meta/value, RPC `Set` method, clickable booleans, `NotifyStatus` on value change.
* **Thermostat** — New setting, "Sensor failure protection" — turn off the Thermostat and send a push notification upon sensor failure. Enabled by default.
  * Sensor failures include: Invalid readings, No readings for a long time.
* **Scripting engine** (QuickJS) — `Shelly.call`, timers (`Timer.set`, `Timer.clear`, `Timer.getInfo`), `Shelly.addStatusHandler` / `Shelly.removeStatusHandler`,
  `Shelly.addEventHandler` / `Shelly.removeEventHandler`, BLE.
* **Important notes on BLE**:
  * In order to receive a BLE advertisement, the scanned device must be added to the Wall Display's BLE Observer list;
    * Unconditional scanning is heavy for the device, causes heating and may lead to degradation.
  * The scanner can NOT be started or stopped — it runs perpetually while BLE observer is enabled.
    * `BLE.SetConfig` is not allowed from scripts.
  * Scanner events include fully parsed BTHome data, if present in the advertisement;
    * Advertisements from encrypted devices will not be decrypted. The `advData` will still be there.
    * A method to decrypt BLE advertisements with a user-provided passkey will be introduced in a future release.
  * Scanner events are deduped — only one event is delivered per unique advertisement payload.
  * The advertisement data will not contain binary strings, as opposed to mainstream Shelly devices.
  * Example event payload:

```json
{
  "type": 2,
  "addr": "C0:2C:ED:36:35:9A",
  "addr_type": 1,
  "name": "SBHT-203C",
  "rssi": -48,
  "advData": "0201060F16D2FC40000001612E133A0145FF000A08534248542D3230334310FFA90B0101000B11000A9A3536ED2CC0000000000000000000000000000000",
  "advData_length": 124,
  "advDetails": {
    "1": {
      "length": 1,
      "data": "06"
    },
    "8": {
      "length": 9,
      "data": "534248542D32303343"
    },
    "22": {
      "length": 14,
      "data": "D2FC40000001612E133A0145FF00"
    },
    "-1": {
      "length": 15,
      "data": "A90B0101000B11000A9A3536ED2CC0"
    }
  },
  "sensorReadings": {
    "packetid:0": {
      "pid": 0,
      "id": 0
    },
    "devicepower:0": {
      "battery": {
        "percent": 97
      },
      "id": 0
    },
    "humidity:0": {
      "rh": 19,
      "id": 0
    },
    "v_eve:0": {
      "event": "single_press",
      "id": 0
    },
    "temperature:0": {
      "tC": 25.5,
      "id": 0
    },
    "raw": "00,00,01,61,2E,13,3A,01,45,FF,00"
  },
  "sensorLength": 6,
  "packetId": 0
}
```

* Not (yet) supported in Scripts:
  * MQTT

### Improvements

* Updated home page item limit for XL, X2i and newer devices from 25 to 50;
* Improved symmetricality on home page for portrait orientations; 
* Refactored hardware key input handling — dynamic `/dev/input` device paths via `List<String>`, per-device monitoring threads;
* Capture external key events from the overlay service;
* Add support for the new structure of SN and MAC addresses;
* Removed WiFi MAC validity checks;
* Explicitly show/hide internal relay buttons; hide thermostat relay buttons in sidebar;
* Save tile expanded state and restore it on startup;
* Update WebUI interface;

### Bug fixes

* Fixed restoring paired BLE sensor when restoring settings;
* Fix large relay tiles and unregistered tile layouts;
* Fix restoring of screen brightness on `Ui.Screen.Set`;
* Fix `Ui.SetConfig` for screen `auto_off` settings;
* Fix DeviceID generation from new serial format;
* Fix missing Switch RPC service;
* Fix crash when BT speakers disconnected while playing and alarm interface is visible;
* Fix homepage T/H/C values when enabling them; fix layout for X1, X1i and X2i when disabled;

## 2.5.8

* Fix regression in thermostat schedules not turning the actuator on or off.

## 2.5.7

* Fix layout in cases where editing items' resize buttons are hidden.
* Added a "Close" button to close the alarm UI without stopping MediaPlayer.
* Attempt to fix regression with Home Assistant.
* Fixed thermostat logic for inverted outputs.
* Introduced a popup asking the user to allow RPC communication over BLE.

## 2.5.6

* Removed "Auto Advance Wizards" option. The wizard will now always auto-advance. Thus, the "Next" button was removed;
* Fixed unregistered and large relay tile layouts;
* Make the home screen items symmetrical in size;
* Fixed restore brightness on `Ui.Screen.Set`;
* Pause background running of WebView to avoid crashes on heavy HA dashboards.

## 2.5.5

* Fix radio alarm not stopping when paused via RPC command;
* Fix multi-channel roller commands in groups;
* Fix crash after changing device language list;
* Limit the lowest possible brightness for some devices;
* Add RSSI and WiFi band info to the Network settings page;
* Skip the bluetooth headset pairing confirmation dialog;
* Fix flip switch behaviour;
* Fix popup dialog sizes for different screens;
* Fix weather tile layout when no valid forecast has been downloaded.

## 2.5.4

* Fix weather tile layout when no valid forecast has been downloaded;
* Fix detached SW input actions in Thermostat mode;
* Updated timezone information to 2025b;
* Fixed the layout of X2 in landscape mode;

## 2.5.3

* Added "Toggle relay state" to first items in gesture action selector;
* More robust checking of WebView version for old X1 devices;

## 2.5.2

* Further fix thermostat schedules not firing when needed after being overridden;
* Fix "Turn screen off when idle" not working;

## 2.5.1

* Exposed Android's Accessibility settings - high contrast text, invert colours, colour correction options, etc.;
* Add convenience options for screen dim, screen off, and screensaver timeouts;
* Fixed thermostat schedule rules to be executed again on the next day they're active, after the wheel has been moved;
* Fixed Schedule.Delete to correctly emit a configuration change event;
* Fixed virtual button notifyEvent broadcasts, as well as those when Button.Trigger RPC call is made;
* Fixed erratic behaviour of thermostat stats;
* Fixed mangled text on Weather tile at first start;
* Increased the smallest width of home page tiles for first-gen X1;
* Added full support for Shelly Weather Station;
* Screen gestures can now be assigned to local, cloud, group, scene actions from the screen;
* XL - side buttons can now be used to change the internal thermostat temperature;
* Fix thermostat layouts for different devices;
* Fix SW input actions in different setups;
* X2 now has two scrollers in portrait mode. Consequently, the thermostat is now on a separate page.

## 2.4.5

* Fix MQTT connection problems.

## 2.4.4

* Fix deleting saved WiFi networks when selected form the list, but not available;
* XL portrait mode;
* notifyInputEvent for XL buttons (1..4). Only `single_push` supported.

## 2.4.3

* Fix Zendure tile saved expanded state, added more Zendure readings;
* Fix TURN_ON, TURN_OFF, ROLLER_OPEN, and ROLLER_CLOSE group actions on XL side buttons;
* Replaced illumination lux readings with icons for devices that use thresholds;
* Fix Flood Gen4 warning and danger display;
* Allow disabling horizontal swipe when on HA page to accommodate horizontal HA scrollers;
* More sensor readings on X2 and XL single tiles;
* Fix Zendure tile restore expanded state;
* Fix display values for SBDI-003E;
* Fix slider tapping without movement causing on/off sequence for non-on-off-devices (rollers);
* Fix favourites reload on settings import.

## 2.4.2

* Fix PV tile not updating when devices in your PV configuration exist as tiles on the home screen.

## 2.4.1

* Fix crash caused by the new PV tile in certain conditions.

## 2.4.0

### New features

* **Settings import/export** - This produces a zip file can then be used to restore the settings on any other Wall Display;
    - You can choose not to export your media library - this is particularly recommended for large libraries (>40MB)
* New media types:
    - `RINGTONE` - used for alarms;
    - `ALERT` - used for alarms and for notifications, e.g. `Media.MediaPlayer.PlayAlert`;
* 10 default `RINGTONE`s are now shipped with the update;
* **Alarms**. From the WebUI you can now set an alarm to play a ringtone, alert, favourite radio station, or a media file;
    - If you have an alarm set, a bell icon will appear next to the clock. When tapped it will show info about your next alarm. You will be able to disable the next alarm from
      there;
* **PV Configurations**. If you enabled and configured these in the app, you can add a tile to the home screen to view it real time;

### Improvements

* Save tile expanded state so that when the device is restarted, they will retain their state;
* Added support for the new style of device icons;
* Implemented SpeedTest - in `Settings`/`Network` you can instantiate a speed test from an European or Chinese server;
* Fixed the display of radio station icons when said icons are .svg files;
* WebUI: Media items now have checkboxes for deleting multiple items at once. (Default `RINGTONE`s can not be deleted);
* Fixed Media Library reload when a USB stick is unplugged;
* The "Updating device information" loading screen will now show only once on app startup. Consequent reconnects will not trigger them;
* Improved the speed of firmware update downloads. Interestingly, the majority of the delay was caused by updating the UI. The progress is now only reported on every 25%;
* Implemented timeouts for WiFi connections, with appropriate suggestions when fired;
