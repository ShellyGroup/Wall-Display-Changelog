# The `Matter` namespace in QuickJS

Reference for the `Matter` global available to Wall Display scripts. It presents commissioned Matter
devices as ordinary Shelly components, so a script written for a relay reads the same for a Matter bulb:
`switch:0`, `output`, `brightness` in percent, `tC` — not endpoints, clusters, levels in 0..254 and
hundredths of a degree.

The mapping from Matter to components is the same one the dashboard tiles use, so a script and a tile can
never disagree about what a device is. What it maps *from* is the set of control shapes detected on each
endpoint when the device was added — see [matter.md](matter.md#what-kinds-of-device-work).

## Contents

- [Entry points](#entry-points)
- [The handle](#the-handle)
- [Components](#components)
- [Reading state](#reading-state)
- [Control](#control)
- [`Matter.Invoke`, the general form](#matterinvoke-the-general-form)
- [Live updates](#live-updates)
- [Errors](#errors)
- [Things worth knowing](#things-worth-knowing)
- [Examples](#examples)

## Entry points

```js
Matter.supported()                      // → bool: this display has a Matter stack of its own
Matter.list()                           // → array of nodes (see below)
Matter.getHandle( nodeId )              // → handle for the whole node, or null
Matter.getHandle( nodeId, endpointId )  // → handle for one endpoint, or null
```

`Matter.supported()` is **not** a gate on the rest. A legacy Wall Display (Wall Display, Wall Display X2)
has no Matter controller and answers `false`, but it can still list and drive the devices a sibling display
shares with it, exactly as a newer display does.

`Matter.list()` is a directory, not a state dump — read values through a handle:

```js
[ { node_id: 3,
    name: "IKEA Plug 1",
    online: true,
    remote: false,                      // true when a sibling Wall Display owns it
    components: [
      "switch:0"
    ] } ]
```

`getHandle()` returns `null` for an unknown node, or for an endpoint the node does not have — the same
convention `Virtual.getHandle()` uses.

## The handle

```js
let node = Matter.getHandle( 3 );       // whole node
let ep   = Matter.getHandle( 3, 1 );    // one endpoint of it

node.node_id                            // 3
ep.endpoint                             // 1  (absent on a node handle)
```

Only the node's **identity** is a property. Everything that changes is behind a method, so a handle held
across a state change never reports a stale value — the same division `Virtual`'s handle makes between
`key`/`type`/`id` and `getValue()`.

| Method                       | Purpose                                                                          |
|------------------------------|----------------------------------------------------------------------------------|
| `getStatus()`                | every component's status (node handle), or this endpoint's own (endpoint handle) |
| `getStatus( key )`           | one named component's status, wherever it sits on the node                       |
| `getComponents()`            | what the node presents, without the values                                       |
| `getDeviceInfo()`            | the node as `Shelly.GetDeviceInfo` would describe it                             |
| `set( [key,] params[, cb] )` | drive a component                                                                |
| `toggle( [key][, cb] )`      | flip a component's on/off                                                        |
| `on( event, cb )`            | subscribe; returns a listener id                                                 |
| `off( listenerId )`          | unsubscribe                                                                      |

## Components

Each endpoint becomes zero or more Shelly components. Ids are allocated per type in ascending endpoint
order, so a single-relay plug is `switch:0` whichever endpoint number it happens to use; each component
keeps its `endpoint` so nothing is lost.

| Matter                                                       | Component                                          | Status fields                                                                                 |
|--------------------------------------------------------------|----------------------------------------------------|-----------------------------------------------------------------------------------------------|
| OnOff                                                        | `switch:N`                                         | `output`, `source: "matter"`, plus `apower`/`voltage`/`current` when the same endpoint meters |
| LevelControl                                                 | `light:N`                                          | `+ brightness` (0..100)                                                                       |
| ColorControl, hue/sat or xy                                  | `rgb:N`                                            | `+ rgb: [ r, g, b ]`                                                                          |
| ColorControl, CCT only                                       | `cct:N`                                            | `+ ct` (kelvin), `ct_range: [ warm, cold ]`                                                   |
| WindowCovering                                               | `cover:N`                                          | `current_pos` (percent **open**), `state`: `open`/`closed`/`opening`/`closing`/`stopped`      |
| FanControl                                                   | `fan:N`                                            | `percent`, `output`, `rock_support`/`rock`, `wind_support`/`wind` — non-standard, Shelly has no fan component |
| Thermostat                                                   | `thermostat:N`                                     | `target_C`/`target_F`, `current_C`/`current_F`, `target_range_C`                              |
| RVC Run Mode + RVC Clean Mode + RVC Operational State        | `rvc:N`                                            | `run_mode`/`run_options`, `clean_mode`/`clean_options`, `state`/`states`, `state_name`        |
| Microwave Oven Control + Operational State                   | `microwave:N`                                      | `cook_time`, `max_cook_time` (seconds), `state`, `running`, `state_name`                      |
| any other ModeBase cluster — dishwasher, oven, washer, …     | `mode:N`                                           | `current` (the mode number), `options` (the device's own list)                                |
| any other OperationalState cluster                           | `operational:N`                                    | `state` (the state number), `states` (the device's own list)                                  |
| Temperature                                                  | `temperature:N`                                    | `tC`, `tF`                                                                                    |
| RelativeHumidity                                             | `humidity:N`                                       | `rh`                                                                                          |
| Illuminance                                                  | `illuminance:N`                                    | `lux`                                                                                         |
| OccupancySensing                                             | `occupancy:N`                                      | `value` (bool)                                                                                |
| BooleanState                                                 | `input:N`                                          | `state` (bool)                                                                                |
| PowerSource                                                  | `devicepower:N`                                    | `battery: { percent }`, `external: { present: false }`                                        |
| Electrical Power, **with no actuator to attribute it to**    | `pm1:N`                                            | `apower`, `voltage`, `current`                                                                |
| PressureMeasurement                                          | `pressure:N`                                       | `value` (hPa)                                                                                 |
| FlowMeasurement                                              | `flow:N`                                           | `value` (m³/h)                                                                                |
| SoilMeasurement                                              | `moisture:N`                                       | `value` (%)                                                                                   |
| AirQuality                                                   | `air_quality:N`                                    | `value` — the 1..6 band, not a concentration                                                  |
| HEPA / activated carbon filter, water tank                   | `hepa_filter:N`, `carbon_filter:N`, `water_tank:N` | `value` (%)                                                                                   |
| PM2.5, PM10, PM1, formaldehyde                               | `pm25:N`, `pm10:N`, `pm1p0:N`, `formaldehyde:N`    | `value` (µg/m³)                                                                               |
| TVOC                                                         | `voc:N`                                            | `value` (µg/m³, as isobutylene — see below)                                                   |
| CO, CO2, smoke concentration                                 | `co:N`, `co2:N`, `smoke_concentration:N`           | `value` (ppm)                                                                                 |
| NO2, ozone                                                   | `no2:N`, `ozone:N`                                 | `value` (ppb)                                                                                 |
| Radon                                                        | `radon:N`                                          | `value` (Bq/m³)                                                                               |
| a reading this display's sensor table names no component for | `sensor:N`                                         | the readings by name, plus a `units` map                                                      |

**The left-hand column is a shape, not a cluster list.** Which component an endpoint becomes is decided
from what its clusters say about themselves — the attributes they implement, the commands they accept
and the ones they answer with — rather than from a table of cluster ids somebody typed in. For the
singletons that is a distinction without a difference: the specification defines exactly one OnOff
cluster, so its id *is* the test. It matters for the families. Every derivation of ModeBase becomes
`mode:N` and every derivation of OperationalState becomes `operational:N`, including derivations
published after this firmware was built, because the detector is the shape of the cluster and not its
number.

Two appliances fold several shapes into one component, deliberately, because each is one machine to a
person. A robot vacuum's two mode clusters and its operational state — what it is doing, how it is doing
it, and where that has got to — arrive together as `rvc:N`, and a microwave's cooking time and its
operational state arrive together as `microwave:N`. The vacuum's clusters are named by id rather than
fingerprinted, because RVC Run Mode and RVC Clean Mode genuinely *are* ModeBase derivations and nothing
about their shape tells the two apart.

Where one endpoint presents several shapes, the component is the highest-priority one — `cover`, then
`rvc`, `thermostat`, `fan`, `microwave`, `rgb`/`cct`, `light`, `switch`, `mode`, `operational`. That ordering,
like the shapes themselves, is part of the display's device configuration, which is updated from Shelly's
servers and does not need a firmware update to change.

**A `mode` renders in the device's own words.** `options` is the list the device published, each entry
carrying the number to send back, the label the device chose for it, and any standardised mode tags. A
dishwasher, for instance:

```js
node.getStatus( "mode:0" )
// { current: 1,
//   options: [ { mode: 0, label: "Normal", tags: [ 16384 ] },
//              { mode: 1, label: "Heavy",  tags: [ 16385 ] },
//              { mode: 2, label: "Light",  tags: [ 16386 ] } ],
//   source: "matter", id: 0 }
```

A vacuum's `run_options` and `clean_options` have the same shape.

Nothing in the display knows what any of those mean, and it does not need to: a control shows the
labels and sends back the number beside whichever was picked. `operational:N` works the same way, with
`states` carrying `{ id, label }` and `state` naming the current one. The lists are read once when the
device is discovered; only `current` and `state` are subscribed, so a mode moving costs the same as a
switch moving.

**A shape can be readable and not writable, and that is a real answer.** Which verbs an endpoint offers
comes from its own accepted-command list: a microwave publishes a mode it will not let you change, an
oven cavity starts and stops but cannot be paused. `set()` on a verb the device does not accept fails
with `-103` and says so, rather than sending a command that would be refused on the wire.

Every status carries `id`. Metering is reported **inside the actuator it measures**, as `apower`, `voltage`
and `current` on that component — the Shelly convention, where a metered plug is one `switch:0` and not a
switch plus a meter.

That holds even when the device puts the two on separate endpoints, which many do: an IKEA GRILLPLATS has
OnOff on endpoint 1 and Electrical Power on endpoint 2, and its power still arrives as `switch:0.apower`. The
outlet a reading belongs to is taken from the node's own composition (`Descriptor.PartsList`, which is how an
endpoint says what is composed under it), or, where the device declares none, from there being a single
actuator on the node for it to belong to. The component names the endpoints it took readings from in
`metering_endpoints`, so a script can still see where a value was measured.

Only a reading that neither test can attribute — several actuators on the node and no composition saying
which one is metered, or a node that meters without switching anything, such as a standalone Matter energy
meter — stands alone as `pm1:N`. Guessing further would produce an attribution indistinguishable from a
true one.

**Where these component names and shapes come from.** From the same sensor table a Bluetooth BTHome sensor's
readings are translated through. A CO2 reading therefore looks the same whether it reached the display over
Bluetooth or over Matter, and a reading BTHome has no id for still gets its component from that table.

**Nothing on the wire says what unit a value is in — the component name does.** `pressure:N` is hPa the way
the Shelly Weather Station's is, `pm25:N` is µg/m³, `co2:N` is ppm. That matters because Matter lets a device
choose: the concentration clusters report a `MeasurementUnit`, so the same sensor type arrives as ppb on one
device and µg/m³ on another. Those are converted before publishing, using the molar mass of the gas and a
molar volume of 24.45 L/mol (25 °C, 1 atm). **A TVOC figure is only meaningful against a reference compound**,
and the one used here is isobutylene, the industry convention — so `voc:N.value` is µg/m³ *of isobutylene
equivalent*, which is what a Sensirion or ams part already means by it.

A reading whose reported unit cannot be converted to the one its component name means — a CO2 endpoint
reporting Bq/m³, or a particulate reported in ppb, which no molar mass can convert — is **not** published
under that name with a wrong number. It falls back to `sensor:N`, which groups whatever the table names no
component for, with its real unit in a `units` map. The same happens for a reading newer than the table on
this display, since the table is updated from Shelly's servers.

Two names are deliberately not the obvious ones. `pm1p0:N` is PM1 particulate — `pm1` is already Shelly's
single-phase power meter, above. And `smoke_concentration:N` is a number, while `smoke` stays what it is
everywhere else on this display: the boolean alarm.

```js
node.getComponents()
// [ { key: "switch:0", type: "switch", id: 0, endpoint: 1, label: "Plug",
//     metering_endpoints: [ 2 ] } ]
```

`label` is, in order of preference: the name the user gave the channel on this display; the name a bridge
gives the device behind it; the device's own UserLabel/FixedLabel; an English name for the endpoint's Matter
device type; or `"Endpoint N"`. A rename made on the display survives the device being read again.
`metering_endpoints` is present only on a component carrying another endpoint's metering.

## Reading state

**Reads are synchronous.** The values are already held on the display, kept current by the device's own
reports, so there is no RPC, no callback and no round trip:

```js
let s = Matter.getHandle( 3 ).getStatus();
// { "switch:0": { source: "matter", output: true, apower: 1.8, voltage: 233, current: 0.007, id: 0 },
//   online: true }

Matter.getHandle( 3 ).getStatus( "switch:0" ).output          // true
Matter.getHandle( 3, 1 ).getStatus().apower                   // 1.8 — endpoint handle needs no key
Matter.getHandle( 3, 2 ).getStatus().apower                   // 1.8 — the metering endpoint answers with the
                                                              //       outlet its readings were folded into
```

`getDeviceInfo()`:

```js
{ id: 0, node_id: 3, device_type: "switch", name: "IKEA Plug 1", model: "GRILLPLATS Plug",
  manufacturer: "IKEA of Sweden", vendor_id: 4476, product_id: 4096,
  serial: null, matter_id: "CA2E09B588365C8A", ver: "1.4.6", hw_ver: "P2.0",
  online: true, remote_host: null }
```

- `id` is the device's number as a `matter:<id>` component in `Shelly.GetStatus` over the cloud — not the
  node id. It is `-1` on a device a sibling Wall Display owns.
- `device_type` is what the device is as a whole: `sensor`, or the type of the component it is driven
  through — `switch`, `light`, `rgb`, `cct`, `cover`, `thermostat`, `fan`, `rvc`, `microwave` and so on. It is
  absent for a device that has nothing on it to drive or read, such as a wall switch that only tells other
  devices what to do.
- `matter_id` is the unique id the device itself publishes.
- `remote_host` is set only on a device a sibling Wall Display owns and shares.

## Control

Writes are asynchronous. The callback is the engine's usual
`cb( result, error_code, error_message )` — the same shape `Shelly.call` uses, so callbacks are drop-in.

```js
let node = Matter.getHandle( 3 );

node.set( "switch:0", { on: true }, ( r, code, msg ) => print( code ? "failed: " + msg : "on" ) );
node.set( "light:0",  { brightness: 40 } );          // also drives an rgb: or cct: component
node.set( "rgb:0",    { rgb: [ 255, 0, 0 ] } );      // or { r: 255, g: 0, b: 0 }
node.set( "cct:0",    { ct: 4000 } );                // kelvin
node.set( "cover:0",  { pos: 25 } );                 // or { state: "open" | "close" | "stop" }
node.set( "fan:0",    { percent: 50 } );             // or { on: true }
node.set( "thermostat:0", { target_C: 21.5 } );      // and/or { on: false }
node.set( "mode:0",   { mode: 2 } );                 // a number from that component's own `options`
node.set( "operational:0", { state: "pause" } );     // start | stop | pause | resume | go_home
node.toggle( "switch:0" );
```

Accepted parameters per component type:

| Component             | Parameters                                                                |
|-----------------------|---------------------------------------------------------------------------|
| `switch`              | `on`                                                                      |
| `light`, `rgb`, `cct` | `on`, `brightness` (0..100), `rgb: [r,g,b]` or `r`/`g`/`b`, `ct` (kelvin) |
| `cover`               | `pos` (percent open), or `state`: `open` \| `close` \| `stop`             |
| `fan`                 | `percent` (0..100), or `on`                                               |
| `thermostat`          | `target_C`, `on`, or both                                                 |
| `mode`                | `mode` — a number taken from that component's `options`                   |
| `operational`         | `state`: `start` \| `stop` \| `pause` \| `resume` \| `go_home`            |

The last two rows name nothing about the device. `set( "mode:0", { mode: 2 } )` resolves the cluster and
the command from the shape detected on that endpoint, so the same line drives a vacuum's run mode, a
dishwasher's programme and whatever the CSA derives from ModeBase next. Read the number out of
`options` rather than hardcoding it — the numbering is the device's, and two vacuums do not agree about
it.

An `operational` action the device does not accept answers `-103` naming the action, which is how "this
one cannot be paused" is told apart from "the pause failed". `go_home` exists only where the device accepts
it.

**Fans.** `{ on: true }` starts a fan at the speed it last ran at, as the tile does, not at full speed;
`{ on: false }` stops it. The `rock` and `wind` fields are bitmaps of the oscillation and wind settings, and
`rock_support` / `wind_support` say which bits the fan accepts. They are readable here but set only over the
local API (`Matter.SetFanRock`, `Matter.SetFanWind`).

**Thermostats.** `on` switches a thermostat on or off: with its on/off control where it has one, as an air
conditioner does, and otherwise by its system mode, which is how Matter turns a thermostat off. `target_C`
and `target_range_C` are the **cooling** setpoint and its limits on a cooling-only thermostat, or on one
that heats and cools while it is set to cool; otherwise they are the heating setpoint. The status carries
no `output`, so a script can switch a thermostat but cannot read from it whether it is on.

**Robot vacuums and microwaves cannot be driven from a script.** `rvc:N` and `microwave:N` are readable, but
`set()` answers `-103` for both (the message calls them a reading). Their controls are on the tile, and
over the local API through `Matter.Invoke`.

Sensor components are read-only and answer `-103` if you try to set one. `toggle()` on a component that
has nothing to switch — a cover, a mode, a sensor — answers `-103` as well.

**Whole-node control.** With no key, an endpoint handle drives its own component and a node handle switches
every controllable endpoint together — what the whole-node dashboard tile does:

```js
node.set( { on: true } );      // every controllable endpoint
node.toggle();                 // ...to the opposite of "is any of them on?"
```

Each endpoint is switched the way it switches on its own: a fan by its speed, a thermostat with no on/off
control by its system mode, everything else by on/off. That includes thermostats whatever the tile setting
*Allow turning thermostats on or off* says — a script asking for the whole node gets the whole node.
"Is any of them on" is judged the same way, endpoint by endpoint.

A whole-node `set`/`toggle` fans out to one command per endpoint, so **the callback is invoked once per
endpoint**, not once.

A `light:N` key also finds an `rgb:N` or `cct:N` component, because dimming a colour bulb has nothing to do
with its colour. The alias goes one way only — a `switch` is not a light.

`toggle()` reads the current state and commands the opposite rather than sending a Matter Toggle: that is
what the tiles do, and the only thing that works for a device a sibling owns.

**Devices another Wall Display owns are driven the same way.** The same call drives a device this display
added and one a sibling display shares with it; the display routes it to whichever owns the device.

## `Matter.Invoke`, the general form

`set()` covers the shapes that have a Shelly component. Underneath it is an RPC that covers everything
else: `Matter.Invoke` sends any command to any cluster of a commissioned node.

**It is not available to scripts.** `Matter.Invoke` is answered only on the display's local WebSocket and
HTTP interfaces, and always needs the device password or a Wall Display pairing, even on a display with
no password set — see [matter.md](matter.md#access-rules). From a script, `Shelly.call( "Matter.Invoke", … )`
answers as though the method did not exist. It is documented here because it is the general form of what
`set()` does, and because it is how a robot vacuum or a microwave is driven from outside the display.

Two argument forms. **By shape**, which is what a caller should normally use:

```json
{ "id": 1, "method": "Matter.Invoke",
  "params": { "node_id": 5, "endpoint": 1,
              "archetype": "rvc_run", "verb": "change_to_mode", "args": { "mode": 1 } } }
```

The cluster, the command id and the field numbering are all resolved from the shapes detected on that
endpoint, so the call names what it wants rather than where it lives. The shapes and the verbs each
offers:

| `archetype`                    | Verbs                                                                                        |
|--------------------------------|----------------------------------------------------------------------------------------------|
| `binary`                       | `on`, `off`, `toggle`                                                                        |
| `ranged`                       | `move_to_level`, `move_to_level_with_on_off`                                                 |
| `chromatic`                    | `move_to_hue_and_saturation`, `move_to_colour`, `move_to_colour_temperature`                 |
| `positional`                   | `up_or_open`, `down_or_close`, `stop`, `go_to_lift_percentage`                               |
| `setpoint`                     | `setpoint_raise_lower`                                                                       |
| `mode`, `rvc_run`, `rvc_clean` | `change_to_mode`, args `{ mode }`                                                            |
| `operational`                  | `start`, `stop`, `pause`, `resume`, `go_home`                                                |
| `cook`                         | `set_cooking_parameters`, args `{ cook_time }`; `add_more_time`, args `{ time_to_add }` (seconds) |

`airflow` (a fan) has no verbs: a fan is driven by writing its speed, not by a command, which is what
`Matter.SetFanPercent` does. Only the verbs that list `args` take named arguments; the rest are sent
without fields, so for a level, a colour, a position or a setpoint use the named control methods
(`Matter.SetLevel` and the rest) or the address form below.

Which of them a given device accepts is in the device's own metadata, and a verb it does not accept is
refused before anything is sent.

**By address**, which names the cluster and command directly:

```json
{ "id": 1, "method": "Matter.Invoke",
  "params": { "node_id": 5, "endpoint": 1, "cluster": "0x0006", "command": "0x00" } }
```

`params` is optional and is already in Matter's JSON form, whose keys are positional and typed:
`{"0:UINT": 2}` is field 0 of the command. Ids may be given as numbers or as `0x` strings, and both
forms work over the Gen2 GET route as well, which is what makes this usable from a browser address bar:

```
/rpc/Matter.Invoke?node_id=5&endpoint=1&cluster=0x0006&command=0x00
```

The answer distinguishes a command that failed from one the device understood and declined:

```json
{ "ok": false, "status": 1, "cluster_status": 3, "message": "Dust bin missing" }
```

`status` is the interaction-model status — `0` success, `0x81` unsupported command, `0x7e` unsupported
access. `cluster_status` is present only when the cluster elaborated, and it means whatever that cluster
says it means: a `1` from RVC Run Mode and a `1` from Dishwasher Mode are unrelated. `response` carries
the body of a command that returns one, which is where `ChangeToModeResponse` puts the sentence the
device wrote for a person to read. The named control verbs flatten all of this to a bare failure; this one
does not, which is the reason it exists in this shape.

Two refusals come from the display rather than from the device:

- **The administrative clusters are never driven** — General Commissioning, Network Commissioning,
  Administrator Commissioning and Operational Credentials, plus the whole root endpoint. This holds for
  every caller. Nothing here can un-pair a device or move it to another network.
- **A paired sibling Wall Display is additionally held to the commands the device advertises.** The
  cluster and command must belong to a shape actually detected on that endpoint. A caller holding the
  device password is not narrowed that way; it already owns the display.

## Live updates

Pushed, not polled. Nothing needs a timer.

```js
let id = node.on( "change", ( e ) => print( e.component + ": " + JSON.stringify( e.delta ) ) );
node.on( "removed", ( e ) => print( "node " + e.node_id + " left the fabric" ) );
node.off( id );
```

A `change` frame is `{ component, name, id, delta }` — the shape `Shelly.addStatusHandler` delivers, so a
handler written for one reads the other unchanged. `delta` carries **only the fields that changed** since
that listener last heard from the node:

```
{"component":"switch:0","name":"switch","id":0,"delta":{"apower":2.1,"voltage":233,"current":0.009}}
```

One report can produce several frames. A sensor that reads temperature, CO2 and PM2.5 has a component for
each, so a handler on it hears one frame per reading that moved rather than a single grouped one — and an
endpoint handle on such a sensor is subscribed to all of them. `getStatus()` on an endpoint handle answers
with the endpoint's actuator if it has one, otherwise with the first of its components, so a script that
wants a particular reading should name it.

Node-level changes arrive under the node's own key, because `online` and `name` belong to no component and a
node going offline would otherwise produce no frame at all:

```
{"component":"matter:3","name":"matter","id":3,"delta":{"online":false}}
```

`removed` fires once when the node leaves the fabric, with `{ component, name, id, event: "device_removed",
node_id }`. Watch for it: a subscriber that only listens for `change` cannot tell a removed device from a
quiet one.

An **endpoint handle** hears only about its own component, plus the node-level frame (an endpoint of an
offline node is exactly as unreachable as the node).

### The generic channel

Matter nodes also appear in `Shelly.addStatusHandler`, keyed `matter:<node_id>`, where `delta` is the node's
whole record in the raw Matter shape, endpoints and all — the same thing `Matter.GetNode` answers with over
RPC. Useful for a script that wants everything without holding handles:

```js
Shelly.addStatusHandler( ( e ) => {
  if( "matter" !== e.name ) return;
  print( "node " + e.id + " changed: " + JSON.stringify( e.delta ) );
} );
```

Neither payload is built unless something is waiting for it, so an unused channel costs nothing — which
matters, because a metered device reports about once a second.

Note that `matter:<n>` means two different numbers in two places. In a script, both here and in the
node-level `change` frame, `n` is the **node id**. In `Shelly.GetStatus` and the status notifications sent
over the cloud, `n` is the device's **component id**, which is `getDeviceInfo().id`.

## Errors

Read methods answer `undefined` for something that does not exist. Control methods and `getHandle` return an
error object, or call back with a code — the standard Shelly RPC codes:

| Code   | Meaning                                                                  |
|--------|--------------------------------------------------------------------------|
| `-103` | invalid argument: nothing to set, a bad value, or a read-only component  |
| `-114` | no such node, endpoint or component; or the command failed on the device |

## Things worth knowing

- **`Matter.list()` includes devices a sibling display shares** (`remote: true`). This differs from the
  `Matter.ListNodes` RPC, which is deliberately local-only so that a peer never believes we own its device.
- **The same component view is available over RPC** as `Matter.GetNodeStatus { node_id, endpoint? }`, for
  devices this display added. It answers what `getStatus()` answers, plus `node_id` and `component_id`.
- **Ids are per type, not per endpoint.** Two endpoints of one node give `switch:0` and `switch:1`; the
  `endpoint` field on each component is what a command actually addresses.
- **`brightness: 0` turns a light off** — the controller sends the level with `MoveToLevelWithOnOff`, which
  is also how `Light.Set` behaves on the display's own dimmer.
- **`ct_range` is `[ warm, cold ]` in kelvin**, from the device's reported physical limits (mireds run the
  other way to kelvin, hence the order), or a sensible default before it has reported them.
- **A node's component list can change** if the device's composition changes under us — and now also
  after a `Matter.Refresh`, since a metadata read that missed a cluster the first time leaves that
  cluster's controls out until it is re-read. Resolve keys against a fresh `getComponents()` rather than
  caching them for the life of a script.
- **Capability is read off the device, so it is only as good as the last discovery.** A family of
  clusters that answered nothing during discovery is deliberately treated as having no verbs rather than
  being offered the ones it usually has. A missing control is visible and recoverable; an invented one is
  neither.
- **Mode numbering belongs to the device.** Two vacuums do not agree about which number is "cleaning".
  Read `options` and match on `label` or on a mode tag, never on the number.
- **Sensor-typed endpoints present no actuator**, even when they expose an incidental control cluster — an
  air-quality sensor with an OnOff for its own display stays a sensor. Identity is decided by Matter device
  type, exactly as the tiles decide it.
- **A bridge's devices are components of one node.** Everything behind a bridge — IKEA DIRIGERA, a Hue
  Bridge — is one node with an endpoint per device, so address a bridged device with an endpoint handle or
  by its component key. Each takes its `label` from the name the bridge gives it, and its battery from its
  own entry on the bridge.
- **No energy counters.** `apower`/`voltage`/`current` come from Electrical Power Measurement; Electrical
  Energy Measurement (`0x0091`) is not read yet, so there is no `aenergy`.

## Examples

```js
print( "Matter controller: " + Matter.supported() );
print( "Matter nodes: " + JSON.stringify( Matter.list() ) );

let node = Matter.getHandle( 1 );
if( null !== node ) {
  print( "node 1 info:       " + JSON.stringify( node.getDeviceInfo() ) );
  print( "node 1 components: " + JSON.stringify( node.getComponents() ) );
  print( "node 1 status:     " + JSON.stringify( node.getStatus() ) );
  print( "node 1 switch:0:   " + JSON.stringify( node.getStatus( "switch:0" ) ) );

  // Control. Whichever Wall Display owns the device, the same call drives it.
  node.set( "switch:0", { on: true }, ( r, code, msg ) => print( code ? "failed: " + msg : "on" ) );
  node.set( "light:0", { brightness: 40 } );          // also drives an rgb: or cct: component
  node.set( "rgb:0", { rgb: [ 255, 0, 0 ] } );
  node.set( "cct:0", { ct: 4000 } );
  node.set( "cover:0", { pos: 25 } );                 // or { state: "open" | "close" | "stop" }
  node.set( "fan:0", { percent: 50 } );
  node.set( "thermostat:0", { target_C: 21.5 } );
  node.set( "thermostat:0", { on: false } );          // OnOff, or system mode Off where it has no OnOff
  node.toggle( "switch:0" );
  node.set( { on: true } );                           // every controllable endpoint at once
  node.toggle();

  // A mode, in the device's own vocabulary: find the option, send its number back.
  let m = node.getStatus( "mode:0" );
  if( undefined !== m ) {
    let quiet = m.options.find( ( o ) => "Quiet" === o.label );
    if( quiet ) node.set( "mode:0", { mode: quiet.mode } );
  }
  node.set( "operational:0", { state: "pause" } );    // -103 if this one cannot be paused

  // A robot vacuum is read here and driven from its tile or over the local API.
  let v = node.getStatus( "rvc:0" );
  if( undefined !== v ) print( "vacuum: " + v.state_name + ", run mode " + v.run_mode );

  // Live updates, pushed — no polling. The delta carries only what changed; the node's own
  // matter:<id> frame is where `online` lives.
  let changed = node.on( "change", ( e ) => print( "changed " + e.component + ": " + JSON.stringify( e.delta ) ) );
  node.on( "removed", ( e ) => print( "node " + e.node_id + " left the fabric" ) );
  Timer.set( 60000, false, () => { node.off( changed ); print( "stopped watching node 1" ); } );
}

// One endpoint of a node, rather than the whole thing: getStatus()/set()/toggle() need no key, and
// "change" only reports that endpoint's component (plus the node going offline).
let ep = Matter.getHandle( 1, 1 );
if( null !== ep ) {
	print( "node 1 endpoint 1: " + JSON.stringify( ep.getStatus() ) );
	ep.toggle();
}
```
