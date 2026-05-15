# 2.6.1

## Fixes

* Fix regression on legacy devices where the SW input would not be recognised while the screensaver is active.
* Fix infinite loading with text "Restarting. Please wait..." when the device language is changed.

# 2.6.0

## New features

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
* **HomeAssistant** — for models supporting AppStore, the HomeAssistant page is deprecated. Please install HomeAssistant as a separate app.
  * Notes on HA and becoming a Home app. We need to fight this because setting HA as a home app effectively bricks the device. Therefore we do not allow any other app than Stargate to be the home app. Opening HA settings and setting it as a Home app will have no effect.
* **Auto-start last opened app** - if the device needs to perform a full reboot (power outage, etc.), while a 3-rd party app is running, this app will be automatically started. So, if for example HA was running in front and the device reboots, after Stargate starts, it will launch HA automatically.
* **Fahrenheit support** — Please note that all values are saved internally in Celsius; Fahrenheit is only for display.
* **Virtual Components** — Number, Text, Boolean, Enum and Group types. Allow creating, setting meta/value, RPC `Set` method, clickable booleans, `NotifyStatus` on value change.
* **Thermostat** — New setting, "Sensor failure protection" — turn off the Thermostat and send a push notification upon sensor failure. Enabled by default.
  * Sensor failures include: Invalid readings, No readings for a long time.
* **Scripting engine** (QuickJS) — `Shelly.call`, timers (`Timer.set`, `Timer.clear`, `Timer.getInfo`), `Shelly.addStatusHandler` / `Shelly.removeStatusHandler`,
  `Shelly.addEventHandler` / `Shelly.removeEventHandler`, BLE.
* Notes on BLE:
  * In order to receive a BLE advertisement, the scanned device must be added to the Wall Display's BLE Observer list;
    * Unconditional scanning is heavy for the device, causes heating and may lead to degradation.
  * The scanner can NOT be started or stopped — it runs perpetually while BLE observer is enabled.
    * `BLE.SetConfig` is not allowed from scripts.
  * Scanner events include fully parsed BTHome data, if present in the advertisement;
    * Advertisements from encrypted devices will not be decrypted. The `advData` will still be there.
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

## Improvements

* Updated home page item limit for XL, X2i and newer devices from 25 to 50;
* Improved symmetricality on home page for portrait orientations; 
* Refactored hardware key input handling — dynamic `/dev/input` device paths via `List<String>`, per-device monitoring threads;
* Capture external key events from the overlay service;
* Add support for the new structure of SN and MAC addresses;
* Removed WiFi MAC validity checks;
* Explicitly show/hide internal relay buttons; hide thermostat relay buttons in sidebar;
* Save tile expanded state and restore it on startup;
* Update WebUI interface;

## Bug fixes

* Fixed restoring paired BLE sensor when restoring settings;
* Fix large relay tiles and unregistered tile layouts;
* Fix restoring of screen brightness on `Ui.Screen.Set`;
* Fix `Ui.SetConfig` for screen `auto_off` settings;
* Fix DeviceID generation from new serial format;
* Fix missing Switch RPC service;
* Fix crash when BT speakers disconnected while playing and alarm interface is visible;
* Fix homepage T/H/C values when enabling them; fix layout for X1, X1i and X2i when disabled;

# 2.5.8

* Fix regression in thermostat schedules not turning the actuator on or off.

# 2.5.7

* Fix layout in cases where editing items' resize buttons are hidden.
* Added a "Close" button to close the alarm UI without stopping MediaPlayer.
* Attempt to fix regression with Home Assistant.
* Fixed thermostat logic for inverted outputs.
* Introduced a popup asking the user to allow RPC communication over BLE.

# 2.5.6

* Removed "Auto Advance Wizards" option. The wizard will now always auto-advance. Thus, the "Next" button was removed;
* Fixed unregistered and large relay tile layouts;
* Make the home screen items symmetrical in size;
* Fixed restore brightness on `Ui.Screen.Set`;
* Pause background running of WebView to avoid crashes on heavy HA dashboards.

# 2.5.5

* Fix radio alarm not stopping when paused via RPC command;
* Fix multi-channel roller commands in groups;
* Fix crash after changing device language list;
* Limit the lowest possible brightness for some devices;
* Add RSSI and WiFi band info to the Network settings page;
* Skip the bluetooth headset pairing confirmation dialog;
* Fix flip switch behaviour;
* Fix popup dialog sizes for different screens;
* Fix weather tile layout when no valid forecast has been downloaded.

# 2.5.4

* Fix weather tile layout when no valid forecast has been downloaded;
* Fix detached SW input actions in Thermostat mode;
* Updated timezone information to 2025b;
* Fixed the layout of X2 in landscape mode;

# 2.5.3

* Added "Toggle relay state" to first items in gesture action selector;
* More robust checking of WebView version for old X1 devices;

# 2.5.2

* Further fix thermostat schedules not firing when needed after being overridden;
* Fix "Turn screen off when idle" not working;

# 2.5.1

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

# 2.4.5

* Fix MQTT connection problems.

# 2.4.4

* Fix deleting saved WiFi networks when selected form the list, but not available;
* XL portrait mode;
* notifyInputEvent for XL buttons (1..4). Only `single_push` supported.

# 2.4.3

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

# 2.4.2

* Fix PV tile not updating when devices in your PV configuration exist as tiles on the home screen.

# 2.4.1

* Fix crash caused by the new PV tile in certain conditions.

# 2.4.0

## New features

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

## Improvements

* Save tile expanded state so that when the device is restarted, they will retain their state;
* Added support for the new style of device icons;
* Implemented SpeedTest - in `Settings`/`Network` you can instantiate a speed test from an European or Chinese server;
* Fixed the display of radio station icons when said icons are .svg files;
* WebUI: Media items now have checkboxes for deleting multiple items at once. (Default `RINGTONE`s can not be deleted);
* Fixed Media Library reload when a USB stick is unplugged;
* The "Updating device information" loading screen will now show only once on app startup. Consequent reconnects will not trigger them;
* Improved the speed of firmware update downloads. Interestingly, the majority of the delay was caused by updating the UI. The progress is now only reported on every 25%;
* Implemented timeouts for WiFi connections, with appropriate suggestions when fired;
