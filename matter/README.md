# Matter on the Shelly Wall Display

A guide to everything Matter-related on a Wall Display: what each model can do, how to add devices,
how to add the display itself to Google Home or Apple Home, how two displays share Matter devices
between them, and which parts of the local API are open, which need a credential, and why.

For writing scripts against Matter devices, see [matter-scripting.md](matter-scripting.md).

## Contents

- [The three roles](#the-three-roles)
- [What your model supports](#what-your-model-supports)
- [Where it lives in the menus](#where-it-lives-in-the-menus)
- [Role 1 — the display as a Matter controller](#role-1--the-display-as-a-matter-controller)
- [Role 2 — the display as a Matter device](#role-2--the-display-as-a-matter-device)
- [Role 3 — sharing Matter devices between Wall Displays](#role-3--sharing-matter-devices-between-wall-displays)
- [Backup, restore and factory reset](#backup-restore-and-factory-reset)
- [The local API](#the-local-api)
- [Integrating with the API](#integrating-with-the-api)
- [What is never available over the API](#what-is-never-available-over-the-api)
- [Privacy and security summary](#privacy-and-security-summary)
- [Limits and known behaviour](#limits-and-known-behaviour)
- [Troubleshooting](#troubleshooting)

## The three roles

Matter is not one feature on a Wall Display but three, and they are independent — you can use any
of them without the others:

| Role                | What it means                                                                                                        |
|---------------------|----------------------------------------------------------------------------------------------------------------------|
| **Controller**      | The display adds other Matter devices and drives them: plugs, bulbs, blinds, sensors                                 |
| **Matter device**   | The display itself is added to Google Home, Apple Home, SmartThings, IKEA Home Smart or Alexa, which then control it |
| **Display sharing** | One display owns the Matter devices; other Wall Displays on the same network use them through it                     |

The first two speak real Matter to the outside world. The third is a Shelly-to-Shelly arrangement
that exists because a Matter device belongs to exactly one controller — the display that added it —
and every other display in the house still wants to show it.

## What your model supports

| Model                              | Adds Matter devices | Is a Matter device | Uses another display's devices |
|------------------------------------|---------------------|--------------------|--------------------------------|
| Wall Display (SAWD-0A1XX10EU1)     | no                  | no                 | **yes**                        |
| Wall Display X2 (SAWD-2A1XX10EU1)  | no                  | no                 | **yes**                        |
| Wall Display XL (SAWD-3A1XE10EU2)  | yes                 | yes                | yes                            |
| Wall Display X1i (SAWD-6A1XX10EU0) | yes                 | yes                | yes                            |
| Wall Display X2i (SAWD-5A1XX10EU0) | yes                 | yes                | yes                            |

The first two generations run older system software with no Matter stack in the firmware at all. They are
not left out, though: they are full participants in display sharing, so a first-generation display in
the hallway can show and control the Matter devices a newer display added.

Certification by the Connectivity Standards Alliance is what an ecosystem checks as it adds the display,
and it is still under way on the X2i, X1i and D1. Those three say so as they go in — SmartThings, Google
Home, Apple Home and Alexa each word the warning differently — and let you add the display anyway, after
which it is used exactly as a certified one is. The XL and the U1 already carry a CSA-issued declaration
and are added without a word about it. A future update carries the certification for the other three,
and nothing has to be re-paired when it does.

## Where it lives in the menus

**Settings → Network → Matter**, then:

- **Matter controller** (Control other Matter devices) — Adding, listing, renaming and removing Matter
  devices, plus everything to do with display sharing.
- **Matter accessory** (Control this device from another controller) — The display's own Matter identity.
  Shown on every current-generation model.

The Matter entry appears on every model, including the ones that cannot add devices themselves, because
that is also where display sharing is set up.

Matter devices are *controlled* from the home screen, not from Settings. The Settings list is for
management: see a device's details, rename it, share it with another system, remove it.

## Role 1 — the display as a Matter controller

### Adding a device

**Settings → Network → Matter → Matter controller → Commission a device.**

You are asked for the device's **11-digit setup code** — the manual pairing code printed on the device,
on its box, or shown in its own app (for example `3497-011-2332`). Then the display asks *"Already on the
network?"*:

- **Yes** — the device is already on your Wi-Fi. The display finds it on the network and pairs with it.
- **No** — the display pairs over Bluetooth first, and asks for your **Wi-Fi network name and password**
  so it can hand them to the device. This is the normal path for a device straight out of the box.

Pairing takes anything from a few seconds to about a minute. Only one device can be added at a time.

If the device you are adding is a **Shelly** device that this display already knows, its channel names
are copied across automatically, so the tiles read the same as they do for the native device.

### What kinds of device work

Matter devices are presented as ordinary Shelly components, so a Matter bulb behaves on the home screen
and in scripts exactly like a Shelly bulb.

**The display works out what a device can do by asking it, rather than by recognising it.** Every Matter
cluster is required to publish which attributes it implements, which commands it accepts and which it
answers with, and the display reads all of that when a device is commissioned. What it looks for in the
answer is a **control shape** — something with an on and an off, something with a percentage and a
command to move it, something with a list of modes and a way to pick one — rather than a particular
product. A kind of device nobody anticipated is therefore operable as long as its clusters are shaped
like something the display already knows how to draw, and a cluster the Connectivity Standards Alliance
publishes after this firmware was built needs no update to work.

| What the device has  | What you get                                                                                                                                                       |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| an on and an off     | a switch — plugs, relays, sockets — with power, voltage and current when it meters                                                                                 |
| a level              | brightness in percent                                                                                                                                              |
| colour               | full colour, and tunable white in kelvin; a light with both has a colour / white switch                                                                            |
| a position           | blinds, shades and roller shutters: open, close, stop, or an exact position                                                                                        |
| a fan speed          | speed in percent, plus oscillation and wind mode where the fan has them                                                                                            |
| a target temperature | a thermostat or air conditioner: target and current temperature, and on/off                                                                                        |
| a list of modes      | that list — a vacuum's cleaning mode, a dishwasher's programme, an oven's setting                                                                                  |
| a run state          | start, stop, pause and resume, and the device's own word for what it is doing                                                                                      |
| a cooking time       | a microwave: set the cooking time, start and stop                                                                                                                  |
| readings             | temperature, humidity, illuminance, occupancy/motion, contact, air quality and its individual pollutants, pressure, flow, soil moisture, filter and tank condition |
| a battery            | battery percentage                                                                                                                                                 |

**Modes and run states are shown in the device's own words.** A device with modes publishes the list
itself, each with a label it chose — "Quiet", "Turbo", "Eco" — and the display shows exactly those,
interpreted by nobody. The same holds for a run state: what a machine says it is doing is what appears.
That is what lets a device the display has never heard of still be driven usefully, and it is why there
is no list of supported appliances here to fall out of date.

**A control is offered only where the device says it accepts it.** Reading something is not the same as
being able to set it, and Matter keeps the two apart: a microwave publishes its cooking mode and takes
no command to change it, an oven cavity starts and stops but cannot be paused. The display asks each
device which commands it takes and offers those, so a control that appears works, and one that would
have failed is simply absent rather than broken.

**What is never driven.** The clusters that administer a device's identity and its place on the network
— commissioning, network configuration, fabric membership — are refused however the request arrives:
from a tile, from a script, from a sibling Wall Display, or over the API. Nothing can send one of your
devices a raw command that un-pairs it or moves it to another network. Removing a device and sharing it
with another system are separate, dedicated actions (see [Renaming, sharing and
removing](#renaming-sharing-and-removing)), not commands passed through to the device.

Every reading arrives as a Shelly component of its own, named after the reading — `co2:0`, `pm25:0`,
`pressure:0` — the same way the Shelly Weather Station reports the readings Shelly has no dedicated
component for. The names and shapes come from the same sensor table a Bluetooth BTHome sensor's readings
are translated through, so a CO2 reading looks the same whichever radio it arrived over, and each value is
converted into the unit its component name means rather than whatever unit the device happened to choose.
The full mapping, component by component, is in [matter-scripting.md](matter-scripting.md).

**Thread devices need a border router of their own.** The display commissions over Wi-Fi and Bluetooth
and provides Wi-Fi credentials; it is not a Thread border router and has no Thread network to offer, so it
cannot take a Thread device straight out of the box. A Thread device that is already running under
another system's hub — IKEA DIRIGERA, say — can still be added: turn on that system's option for using the
device with other systems (the wording varies), which gives you a Matter pairing code, and add it here
with **"Already on the network?" → Yes**. The display then reaches it through that hub's border router.
Wi-Fi and Ethernet Matter devices work directly. A device that supports both Thread and Wi-Fi will be
added over Wi-Fi.

### Using a device

Add a tile the same way you add any device: **swipe down from the top of the home screen to open the
layout bar, tap "+" → Matter →** pick the device, then what the tile shows, then a tile size:

- **Use default layout** puts the whole device on one tile. A tap switches everything on it together, and
  its sliders, colour controls and readings all appear on that one tile.
- **One of its channels** gives a tile for that channel only — one socket of a multi-socket plug, say.
  Run the wizard again for each further channel you want on its own tile.

**Bridges and hubs** — IKEA DIRIGERA, a Philips Hue Bridge and the like — do not offer the default layout:
one tile for everything behind a hub would be of no use, so each device behind it gets a tile of its own,
listed under its own name. A battery-powered device behind a bridge shows its own battery level.

Thermostats can also be switched on and off by tapping their tiles once **Settings → General → Allow
turning thermostats on or off** is turned on. That setting also decides what a default-layout tap does on
a device that has both a relay and a thermostat, such as another Wall Display: switch both, or the relay
alone. Air conditioners always have their own on/off control.

State is **pushed by the device, not polled**, so a light someone switches at the wall updates on the
display immediately. A device that loses power is shown as offline within roughly half a minute.
Battery-powered sensors are deliberately given a much slower heartbeat — waking them constantly to ask
whether they are still there would flatten them — so they can take a few minutes to be shown as offline.

### Renaming, sharing and removing

Tap a device in the controller list for its details: vendor, product, the Shelly device it is (when it
is one this display also knows natively), and each of its channels with what kind of channel it is.

**Rename device** renames the whole device; tap a channel to rename that channel alone. A rename is
written to the device itself, so other apps see it too, and Wall Displays using this device through
[display sharing](#role-3--sharing-matter-devices-between-wall-displays) pick the new name up on their own.
Matter limits a device name to 32 characters and a channel name to 16, and longer names are cut short.

**Use this device in other systems** shows a Matter pairing code, as a QR code and as a manual code, for
adding the same device to Google Home, Apple Home, SmartThings or the device maker's own app alongside
this display. This is Matter's multi-admin: the device stays on this display too. The code is live for
**three minutes** and then lapses by itself. It is offered only for devices this display added itself; a
device shared to you by another Wall Display has to be shared from that display.

**Remove device** unpairs it properly: the display tells the device to forget it. The device then has to
be commissioned again to be used. If it cannot be reached — already reset, or thrown away — the display
offers to forget it locally instead.

Deleting a *tile* does not unpair anything. Removal only happens in Settings.

## Role 2 — the display as a Matter device

This is the other direction: the display appears in Google Home, Apple Home, SmartThings or Alexa as a
device those systems can read and control.

### Turning it on

**Settings → Network → Matter → Matter accessory → Enable Matter communication.**

The first time you turn this on, the display fetches its own Matter identity from Shelly's provisioning
service, so it needs a **working internet connection just for that one step**, and the display restarts
once afterwards. It is a one-off: after that everything is local, and Matter continues to work with no
internet at all. Until the identity has been fetched, the display does not offer itself to other systems.

Turning the toggle back off makes the display unreachable to those systems but **does not un-pair them**.
Turn it on again and every ecosystem you had added is still there.

### What the ecosystem sees

The display presents:

- **A thermostat**, if it has been enabled
- **An output** for each relay or dimmer the fitted power base actually has — a switch, or a light where
  the base is a dimmer
- **An occupancy sensor**, always: the proximity sensor that wakes the screen, reporting whether somebody
  is in front of the display. On the XL, which senses with radar, presence clears a short time after the
  room empties; the other models clear it as soon as you step away.
- **Temperature and humidity**, while an external Shelly Blu H&T is paired and reporting

The outputs can be saved into the ecosystem's scenes and recalled from them.

The display is announced under the **name you gave it in Settings**, both while an ecosystem is looking for
it and once it has been added. Set the name before adding it: an ecosystem keeps the name it saw at that
point, and a rename made in the ecosystem afterwards stands.

These come and go as you configure the display, on purpose: pair a Blu H&T and temperature and humidity
appear; set up a room thermostat and a thermostat appears. Two deliberate withdrawals are worth knowing
about:

- The relay a thermostat uses as its **actuator** is hidden. The display will not let an outside system
  fight its own thermostat for that relay, and a switch that refuses every command would be worse than
  no switch at all.
- The **temperature** reading is hidden when a thermostat is already reporting that same sensor, so the
  ecosystem does not see one reading twice and treat it as two rooms.

Switching a thermostat between heating and cooling needs no re-pairing.

### Adding it to a system

**Add this Wall Display to another system** shows a QR code and a manual pairing code. Scan or type it
in Google Home, Apple Home, SmartThings or Alexa. The code is live for **three minutes**; cancel the
dialog and it stops immediately.

The display can belong to **several ecosystems at once** — Apple Home and Google Home together is fine.
Each one is listed under *Other systems controlling this Wall Display*, by name where the name is
recognisable. Removing one takes control away from that system only; anything you had built there using
the display stops working.

The display is **commissionable on the network only**, never over Bluetooth. That is a platform
limitation of the hardware, not a setting. Ecosystems that insist on Bluetooth-based setup will not find
it; every one listed above accepts a typed or scanned code.

### Every ecosystem needs a hub of its own

This one catches everybody, so it is worth stating before you start: **the phone app is not the
ecosystem.** In all four systems the app only reads the code and runs the handshake; the credentials for
the Matter network, and the connection that actually talks to the display afterwards, live on a piece of
that vendor's hardware. With no such hardware on your network there is nothing for the display to be
added *to*, and the app refuses before it even looks for it.

- **SmartThings** — SmartThings Station, Aeotec Smart Home Hub (the old SmartThings v3), a 2022-or-later
  Samsung TV with SmartThings Hub built in, a Family Hub fridge, or an M7/M8/Odyssey Smart Monitor
- **Apple Home** — HomePod, HomePod mini or Apple TV
- **Google Home** — Nest Hub, Nest Hub Max, a Nest speaker or display, or Chromecast with Google TV
- **Amazon Alexa** — a Matter-capable Echo
- **IKEA Home Smart** — DIRIGERA

Two things this is *not*:

- **It is not a Shelly requirement**, and switching apps will not avoid it. Every ecosystem in the table
  works this way for every Matter device, not only for a Wall Display.
- **It is not about Thread.** The usual "Matter needs a hub" advice concerns Thread devices and their
  border routers. The Wall Display is commissioned over the network, so no Thread radio is involved
  anywhere: the hub only has to be on the **same network and subnet** as the display.

Hubs are also not interchangeable between ecosystems. A Dirigera anchors IKEA Home Smart and nothing
else, so it will not satisfy SmartThings — it would let you add the display to IKEA Home Smart instead.

**Without buying any of them**, two routes give you the same thing:

- **Another Wall Display**, through [display sharing](#role-3--sharing-matter-devices-between-wall-displays)
  — no hub, no ecosystem, no account.
- **Home Assistant** with its Matter add-on, where whatever Home Assistant runs on is the hub.

## Role 3 — sharing Matter devices between Wall Displays

A Matter device is added by one controller. If you have three Wall Displays, only the one that added the
plug can talk to it — and older displays cannot add devices at all. Sharing closes both gaps: one display
**shares** its Matter devices, and the others **use** them, over your local network.

Both sides are under **Settings → Network → Matter → Matter controller**.

### Sharing yours

**Share my Matter devices** puts an **8-digit code** on the screen. It is valid for three minutes and
works once. Displays that have already paired with you are listed underneath as *Uses our Matter
devices*.

The option appears only if this display has Matter devices of its own to share.

### Using someone else's

**Use another Wall Display's devices** searches the network and lists the Wall Displays it finds that
have Matter devices. Pick one, type the 8 digits shown on *its* screen, and its devices arrive on this
display: tiles, control, live state and scripting, the same as devices added here.

The code is a one-time secret, never anything derived from a serial number, and it is only ever shown on
the screen — it cannot be read out of the display over the network. Wrong codes earn a growing pause
rather than closing the window, so a neighbour guessing cannot spend your pairing attempt for you.

### What a paired display can and cannot do

A display using another's devices can **read them and control them** — on/off, brightness, colour,
covers, fans, thermostat setpoints. It cannot **add, remove, re-scan or share** devices on the owner's fabric.
Those stay with the display that owns them, driven by its own screen and its own WebUI.

Only the devices you have actually put on a tile are kept live, so a display showing two of twenty shared
devices is not fed the other eighteen.

### Ending it

Either side can end the relationship, and both directions take effect immediately:

- The **owner**: tap the paired display under *Uses our Matter devices* → stop sharing.
- The **user**: tap the display under *Shares its Matter devices with us* → stop using. Its tiles are
  removed from your dashboards.

If the owning display is rebooting or off the network, its shared devices show as offline and recover on
their own. If a device is removed on the owner, it disappears from everyone using it; a device added on
the owner appears on the others within seconds, ready to be put on a tile.

## Backup, restore and factory reset

A **settings backup** carries the Matter side too: the devices the display has added, the credentials
behind them, and its pairings with other Wall Displays. Those credentials belong to one physical display,
so they are restored **only onto the display the backup was taken from**. Restore the backup onto a
different Wall Display and everything else comes back as usual while the Matter part is left out; the
restore says which of the two happened.

A **factory reset** removes the display's Matter devices, its credentials, its pairings with other Wall
Displays and the ecosystems it had been added to, so a display that is passed on does not keep the previous
home's Matter setup.

## The local API

Everything above is also reachable over the display's RPC API, in the `Matter.*` namespace. Most of it
follows the same rule as the rest of the API. A small group of methods is stricter, and on top of that
the transport a call arrives on decides which methods it can reach at all. The access rules come first.

### Access rules

**1 — The device's own posture.** Almost the whole namespace behaves like the rest of the API, and like a
relay: open to any caller on the network if no device password is set, and behind the password if one is.

| What                    | Methods                                                                                                                                                                                                                                                       |
|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Status                  | `Matter.GetStatus`, `Matter.DiscoverPeers`                                                                                                                                                                                                                    |
| Reading devices         | `Matter.ListNodes`, `Matter.GetNode`, `Matter.GetNodeStatus`                                                                                                                                                                                                  |
| Controlling devices     | `Matter.SetOnOff`, `Matter.Toggle`, `Matter.SetLevel`, `Matter.SetColour`, `Matter.SetColourTemperature`, `Matter.CoveringOpen`, `Matter.CoveringClose`, `Matter.CoveringStop`, `Matter.CoveringSetPosition`, `Matter.SetFanPercent`, `Matter.SetFanRock`, `Matter.SetFanWind`, `Matter.SetThermostatSetpoint` |
| Administering devices   | `Matter.Commission`, `Matter.Remove`, `Matter.Refresh`, `Matter.Ping`, `Matter.RecoverDevices`, `Matter.ShareDevice`                                                                                                                                         |
| The display as a device | `Matter.GetAccessory`, `Matter.OpenCommissioningWindow`, `Matter.CloseCommissioningWindow`, `Matter.RemoveFabric`                                                                                                                                             |
| Display sharing         | `Matter.PeerConnect`, `Matter.PeerDisconnect`                                                                                                                                                                                                                 |

Controlling a Matter device over the API costs exactly what controlling the display's own relay costs. On
a display with no password, that means anyone on the local network can read and drive its Matter devices,
just as they can its relay. **Set a device password** if that is not what you want.

A **Wall Display pairing** (see [display sharing](#role-3--sharing-matter-devices-between-wall-displays)) is
a second credential that opens `Matter.GetStatus` and the reading and controlling rows on a display that
does have a password — which is how a paired display keeps working without ever being told the password. It does not
open the administration rows or the display-as-a-device rows: adding, removing, refreshing, recovering
and sharing devices, and anything to do with the display's own Matter identity, stay with the display that
owns them.

`Matter.GetStatus` is the probe a sibling display uses to ask *"are you a controller?"* before it holds
any credential. It reports whether Matter is supported and running, the display's name, **counts** —
devices, sibling displays it uses and that use it — and, for the display as a Matter device, how many
ecosystems it belongs to (`num_fabrics`) and whether its own commissioning window is open right now
(`commissionable`), named as the rest of the Shelly range names them. It never reports device names,
which displays it is paired with, or their addresses, and it says nothing about whether a
display-sharing code is on the screen.

`Matter.GetNodeStatus { node_id, endpoint? }` answers a device's state in the same Shelly component shape
scripts see — `switch:0`, `light:0`, `cover:0`, `temperature:0`, with `output`, brightness in percent and
temperatures in degrees — rather than as Matter endpoints and clusters, plus `online`. Add `endpoint` for
one channel of a multi-channel device. It answers for devices this display added itself; for a device
shared to it by another Wall Display, ask the display that owns it, or use `Matter.GetNode`, which answers
for both. `Matter.ListNodes`, likewise, lists only the devices this display added.

`Matter.ShareDevice { node_id }` is the API form of **Use this device in other systems**: it opens a
three-minute commissioning window on a device this display added and answers with the pairing code for it,
as `manual_code` and `qr_code`. The window cannot be closed early; it lapses on its own.

**2 — A credential always, even with no device password.**

- **`Matter.Invoke`** — the device password or a Wall Display pairing.
- **`Matter.PeerSubscribe`, `Matter.PeerUnpair`** — a Wall Display pairing only, never the password.

`Matter.Invoke` is the general form of the control verbs: it sends *any* command to *any* cluster of a
commissioned device, which is what makes a kind of device nobody wrote a verb for controllable at all. That
is a wider grant than every named verb put together, so it is the one control method that never answers
an anonymous caller, and it is gated twice:

- **Nobody drives the administrative clusters.** Commissioning, network configuration and fabric
  membership are refused for every caller, along with the whole root endpoint. This is the same refusal a
  tile or a script meets, applied where the command is issued rather than where it is composed.
- **A paired Wall Display is held to what the device advertises.** The cluster and command it names have to
  belong to a control shape actually detected on that endpoint — which means the device's own list of
  accepted commands put it there, not the caller. A shared device can therefore be driven through exactly
  the controls a tile would offer and no further.

A caller holding the **device password** is not narrowed the second way: it already owns the display
outright. The two argument forms are documented in
[matter-scripting.md](matter-scripting.md#matterinvoke-the-general-form).

`Matter.PeerSubscribe` and `Matter.PeerUnpair` act on the caller's *own* pairing, so the caller is always
identified by the credential it used, never by a parameter. A paired display can therefore only ever unpair
itself, and cannot silence or evict another one.

**The pairing exchange itself** (`Matter.PeerPairBegin`, `Matter.PeerPairFinish`) is necessarily exempt
from authentication — a display that has not paired yet has nothing to authenticate with. What protects
it is the code on the screen: three minutes, single use, and a growing pause after every wrong attempt.
`Matter.PeerPairBegin` never returns the code, only the values needed to prove you know it.

### Which transports reach what

The credential is checked by the transport that accepted the call, so the rules above describe what a
caller on the **local network** or over **Bluetooth** meets. Per channel:

| Channel                                                     | `Matter.*`           | Credential enforced                | `Matter.Invoke` | Peer and discovery verbs |
|-------------------------------------------------------------|----------------------|------------------------------------|-----------------|--------------------------|
| Local WebSocket / HTTP                                      | yes                  | yes, as described above            | yes             | yes                      |
| Bluetooth                                                   | yes                  | yes, as described above            | no              | no                       |
| Shelly Cloud                                                | yes                  | no, see below                      | no              | no                       |
| MQTT                                                        | no, refused outright | not applicable                     | no              | no                       |
| Scripts, schedules, webhooks, the display's own home screen | yes                  | no, this is the display's own code | no              | no                       |

A method held to another channel answers on this one as though it did not exist.

**Over Shelly Cloud the namespace is open.** A cloud frame arrives already authenticated as your account's
session — that is what the cloud connection is — and carries no Matter credential of its own, so no further
credential is asked of it. That is what lets the Shelly app show and drive a display's Matter devices. It
also means the account session is what stands in front of those devices on that route. `Matter.Invoke` and
the peer and discovery verbs are not offered over the cloud at all.

Over the cloud, and only there, `Shelly.GetConfig` and `Shelly.GetStatus` also list the display's own Matter
devices as `matter:<component_id>` entries, and a change to one is pushed as a `NotifyStatus` for that
component. `Matter.GetNodeStatus` answers `component_id` alongside `node_id`, so a caller can tie the two
together.

**Over MQTT it is refused entirely** — every verb, whether it exists or not, and the device's Matter status
is not published to the broker either. A broker is a third party you point the display at, on which every
subscriber is anonymous to the display, which is not a footing on which to offer someone's lights.

**The peer and discovery verbs are local-network only**: `Matter.DiscoverPeers`, `Matter.PeerConnect`,
`Matter.PeerPairBegin`, `Matter.PeerPairFinish`, `Matter.PeerSubscribe`, `Matter.PeerUnpair`,
`Matter.PeerDisconnect`. They exist for two Wall Displays on one network to find each other and share, and
each is answered on the strength of the connection it arrived on — an address to browse to, a code on the
screen you are standing in front of, or the identity of the link itself. None of that survives a trip
through the internet, so they are answered on the local WebSocket and HTTP interfaces and nowhere else.
`Matter.Invoke` is held to the same two interfaces, for the reason given above.

## Integrating with the API

- **Set a device password** and use the normal Gen2 digest authentication. It opens every method in the
  first group and `Matter.Invoke`, and it is what keeps anyone else on the network from doing the same.
- **Read devices in the Shelly shape** with `Matter.GetNodeStatus`, so a Matter bulb reads the same as a
  relay or a Shelly light, rather than translating `Matter.GetNode`'s endpoints and clusters yourself.
- **Pair as a Wall Display**, if you are building something that behaves like one: two RPCs and an
  8-digit code the user reads off the screen, after which every call is signed with a full-strength key
  derived from it, never with the code.
- **Run a script on the display.** A script already inside the display reads and controls Matter devices
  directly, with no network call and no credential — including devices another display owns. See
  [matter-scripting.md](matter-scripting.md). For most integrations on the display itself this is the
  simplest answer by a wide margin.
- **Go through Shelly Cloud**, if you are integrating from outside the house anyway. The namespace answers
  there with no Matter credential of its own, since the cloud session has already authenticated you.
  `Matter.Invoke` and the peer and discovery verbs are the exceptions and are not offered over the internet.

## What is never available over the API

**The display's own pairing codes, in either direction.** There is no method that returns them, and this
is not an oversight:

- The display's own **Matter pairing code** grants an ecosystem lasting control of its relays. Handing
  that to anything that merely reached the RPC server would be handing over the hardware. It is random
  and stored on the device — deliberately *not* derived from the serial number, which would make it
  computable by anyone who knows the serial and how the derivation works.
- The **display sharing code** is the same argument one step down. Pairing should be something a person
  does, standing in front of the screen, not something a program on the network can do to you.

Both appear on the display's own screen and nowhere else.

The one pairing code the API does return is the one `Matter.ShareDevice` mints for a device this display
added. That is a different thing: a code made on request for one third-party device, live for three
minutes, and of no use to anyone who cannot also reach that device on its own network. A paired Wall
Display cannot ask for one.

## Privacy and security summary

- The display's own Matter pairing code and the display-sharing code stay on the screen. The only code the
  API returns is a short-lived one for sharing a device this display added (`Matter.ShareDevice`).
- `Matter.GetStatus` reports counts and the display's own name, never device names, peer names or peer
  addresses.
- Reading and controlling Matter devices follows the display's password, exactly like its relay. With no
  password set, anyone on the local network can do both; set a password to close that.
- `Matter.Invoke`, the raw command form, always needs the password or a Wall Display pairing, and is only
  answered on the local network.
- Over Shelly Cloud the account session is what authorises the call.
- The namespace is not offered over MQTT at all, neither control nor status.
- Finding, pairing with and sharing between Wall Displays happens on the local network only.
- A paired display's key is derived from the 8-digit code but is not the code; the code itself is
  discarded by both sides after pairing and never travels over the network.
- Pairing windows are three minutes and single-use, and wrong attempts are rate-limited.
- Every state update pushed to a paired display re-checks that the pairing still exists, so revoking one
  takes effect at once rather than when the connection next drops.
- A pairing is revocable from either end and survives reboots on both. A factory reset removes it; a
  settings backup carries it, but restores it only onto the same display.
- When the display acts as a Matter device, it reports **one** network to the ecosystem — its own — rather
  than every Wi-Fi network it has ever joined, and refuses to be told to forget the network it is
  currently using.

**One honest limitation.** Someone who is on your network *and* captures the one display-sharing pairing
exchange as it happens could work the 8-digit code out afterwards. Nothing else in the protocol leaks it,
and after pairing the credential is full strength. Pair displays on a network you trust.

## Limits and known behaviour

- **Thread devices need another system's hub.** The display is not a Thread border router, so a Thread
  device can only be added once it is running under a hub that is, and has been shared from there (see
  [What kinds of device work](#what-kinds-of-device-work)).
- **Door locks are not supported**, neither on the display that adds them nor through display sharing.
- **The display cannot be commissioned over Bluetooth** as a Matter device. On-network only.
- **No energy totals.** Matter devices report live power, voltage and current; cumulative energy is not
  read yet, so there is no kWh figure for a Matter plug.
- **The display declares that it runs on mains and has no battery.** A few third-party apps read a
  battery attribute regardless of that declaration and render its absence as a critical battery level.
  This is cosmetic and a bug in those apps; reporting a fake battery would fail Matter's own conformance
  tests, so it is not done.
- **Endpoints appear and disappear** on the display-as-a-device as you pair a sensor, configure a
  thermostat or re-point a thermostat's actuator. This is intentional and ecosystems handle it; what
  never happens is renumbering, so a device that exists keeps the same identity.
- **One commissioning at a time**, in either role.
- **Robot vacuums and microwaves have tiles; other appliances do not yet.** A vacuum gets a tile with
  run/pause/resume, a dock button and its own mode list; a microwave gets a cooking-time slider and
  start/stop. A dishwasher, oven or laundry washer is detected and reaches scripts and RPC as a `mode` and
  an `operational` component, but has no tile control of its own yet.
- **A device that declines says so, and that is not a failure.** A vacuum will not start while it is
  seeking its charger and answers "This change is not allowed at this time". The display shows the
  device's own sentence rather than a generic error, because the refusal arrives at the protocol level
  as a *success*: the device understood perfectly well and said no.
- **A control that a metadata read missed comes back on the next refresh.** Capability is worked out
  from what a device answered during discovery, and a family of clusters that answered nothing is
  deliberately treated as having no controls rather than being offered ones that might not work. If a
  device shows fewer controls than it should, read it again with `Matter.Refresh { node_id }`, or remove it
  and add it again.

## Troubleshooting

**"Commissioning failed" when adding a device.** Most often the device's own pairing window has expired —
Matter devices stop accepting a new controller roughly 15 minutes after power-up. Power-cycle the device
and try again immediately. Otherwise: check the code, check that the device is not already paired to
another controller (it must be reset first, or shared from that controller), and, for a Thread device,
that it is running under a hub.

When a device that is already on the network cannot be added, the display looks at what is advertising on
the network and says which of these it is:

- **The device is not in pairing mode** — its commissioning window has closed. Open it again (from the
  device, or from the app or system that shared it) and retry.
- **No device matches the code** — devices are ready to pair, but none is the one the code belongs to.
  Check the code is the one for this device.
- **Nothing answers for the device's address** — something is advertising itself as ready to pair, but
  nothing on the network answers for it. For a Thread device this means the hub advertising it has
  stopped publishing its address: restart that hub and try again.
- **Pairing never started** — the device is ready and answers on the network, but the pairing did not
  begin. Try again, power-cycling the device first if it repeats.

**"Device already added."** The device is already on this display. There is nothing to do; find it in the
controller list.

**"Revoked certificate."** The device's security certificate, or that of its whole product line, has been
withdrawn by its manufacturer or by the Connectivity Standards Alliance. It may not be a genuine product.
You can add it anyway, but it is worth finding out why first.

**"This Wall Display can not add Matter devices itself."** A first- or second-generation display. Use
display sharing: pair it with a newer display and use that display's devices.

**No *Matter accessory* entry in the menu.** A first- or second-generation display. Those carry no Matter
stack at all, so there is nothing to configure; use display sharing instead.

**The system warns that the Wall Display is not certified.** Expected on the X2i, X1i and D1, whose
certification is still under way, and safe to accept: add the display and it behaves exactly as a
certified one does. A future update removes the warning, with nothing to re-pair.

**"This Wall Display cannot produce a Matter pairing code yet."** Its Matter identity has not been
fetched. Turn *Enable Matter communication* on with a working internet connection; it needs the network
only for that one step.

**"You need a hub" when adding the display to SmartThings** — or the equivalent message in Google Home,
Apple Home or Alexa. It means a hub belonging to *that* ecosystem, not any hub you happen to own: the
phone app only runs the handshake, and the Matter credentials live on the vendor's hardware. See
[Every ecosystem needs a hub of its own](#every-ecosystem-needs-a-hub-of-its-own) for what counts in each
system, why another vendor's hub (an IKEA Dirigera, say) cannot stand in, and the two hub-free routes.
Nothing about this is Thread-related — the display has no Thread radio and needs no border router.

**A newly added ecosystem cannot find the display.** Make sure both are on the same network and that the
network is not isolating clients from each other (guest networks and some mesh setups do). The display is
found over the network, never over Bluetooth.

**Shared tiles all went offline at once.** The display that owns them is rebooting or off the network.
They come back on their own — nothing to re-pair.

**A shared device vanished.** It was removed on the display that owned it. Removing a device unpairs it
for everyone.

**A tile does nothing.** If it is shared, the owner may have removed the device; the display re-checks and
drops the tile. If it is local, check the device is powered and on the network — a device the display cannot
reach is shown on its tile as offline.
