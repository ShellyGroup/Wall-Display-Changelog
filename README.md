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

Please keep in mind that modern devices do not support downgrading. Therefore, if you opt to install a beta update, you may not go back to the stable until the same version is
uploaded.

## Legacy devices OTA Updates

Since 2.6.2 the update requires to clear the device caches on legacy devices. Therefore, **Wall Display** and **Wall Display X2** will take **more time to boot the first time after
an update**. Please keep that in mind, don't power-cycle it, it will come around.

With that covered, let's dive into the changelog.

---

# CHANGELOG

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

| Model           | Name     | Market Name      | Dashboards | Max Tiles / Dashboard |
|:----------------|:---------|:-----------------|:----------:|:---------------------:|
| SAWD-3A1XE10EU2 | Blake    | Wall Display XL  |     5      |          50           |
| SAWD-5A1XX10EU0 | Jenna    | Wall Display X2i |     3      |          50           |
| SAWD-6A1XX10EU0 | Cally    | Wall Display X1i |     1      |          50           |
| SAWD-4A1XE10US0 | Maverick | Wall Display U1  |     1      |          50           |
| SAWD-6A0XX0EU0  | Dayna    | Wall Display D1  |     1      |          50           |
| SAWD-0A1XX10EU1 | Stargate | Wall Display     |     1      |          25           |
| SAWD-2A1XX10EU1 | Pegasus  | Wall Display X2  |     3      |          25           |

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
