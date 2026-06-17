# 🌞 Multi-Window Thermal Protection for Home Assistant

Advanced Home Assistant automation that automatically protects a home from summer overheating by controlling blinds according to:

* the sun's orientation,
* the actual solar position,
* indoor temperature,
* UV index,
* heat-wave alerts,
* and per-window comfort thresholds.

This version is designed to be:

✅ modular
✅ reusable
✅ easy to adapt to other homes
✅ compatible with multiple blinds/windows
✅ observable thanks to a built-in DEBUG mode

---

# ✨ Features

* Centralized management of multiple windows/blinds
* Room-by-room thermal protection
* Real solar calculation via `sun.sun`
* Triggering based on:

  * solar azimuth
  * solar elevation
  * UV
  * indoor temperature
  * weather heat-wave alert
* Intelligent partial blind positioning
* Automatic return to the normal state when the sun no longer hits the window
* Detailed DEBUG notifications
* Architecture based on easily extensible variables

---

# 🧠 How It Works

Every minute, the automation:

1. Iterates through the list of windows defined in `windows`
2. Checks for each opening:

   * whether the sun is actually hitting the facade
   * whether the heat is becoming problematic
   * whether the blind is not already sufficiently closed
3. Activates an appropriate solar protection
4. Reopens automatically when the sun leaves the exposure zone

---

# ☀️ Blind Closing Conditions

Solar protection is activated only if:

## 1. The sun is actually hitting the window

Conditions:

* solar azimuth between `az_min` and `az_max`
* solar elevation greater than `el_min`

Example:

```yaml
az_min: 110
az_max: 280
el_min: 5
```

---

## 2. The heat is considered excessive

Protection is activated if:

```yaml
indoor temperature >= threshold
AND UV >= 4
```

OR if a heat-wave alert is detected.

---

## 3. The blind is not already sufficiently closed

```yaml
current_position > 35
```

---

# 🔄 Automatic Reopening

When the sun leaves the exposure zone:

```yaml
not sun_in_range
```

and the protection was active, the end-of-protection script is executed.

---

# 🧩 System Structure

The system is based on:

## 1. A main automation

It orchestrates the full logic.

---

## 2. A closing script

```yaml
script.protection_solaire_volet
```

Responsible for:

* closing the blind to a target position
* activating the associated helper

---

## 3. An end-of-protection script

```yaml
script.fin_de_protection_solaire_volet
```

Responsible for:

* restoring normal behavior
* deactivating the helper

---

## 4. `input_boolean` helpers

A helper per window:

```yaml
input_boolean.protection_solaire_cuisine_active
```

They are used to remember the protection state.

---

# ⚙️ Parameters to Adjust

## 📌 Window List

Everything is configured here:

```yaml
windows:
```

Each entry represents one opening.

---

# 🪟 Window Parameters

## `name`

Human-readable name used in logs/debug.

```yaml
name: cuisine
```

---

## `volet`

Blind entity.

```yaml
volet: cover.mon_volet
```

---

## `helper`

Helper associated with this window.

```yaml
helper: input_boolean.protection_solaire_cuisine_active
```

---

## `temp_sensor`

Indoor temperature sensor.

```yaml
temp_sensor: sensor.temperature_salon
```

---

## `threshold`

Temperature that triggers protection.

```yaml
threshold: 23
```

---

## `comfort_cover`

Target blind position.

Example:

```yaml
comfort_cover: 27
```

Corresponds to:

* 0 = closed
* 100 = open

---

## `az_min` / `az_max`

Solar exposure zone for the window.

```yaml
az_min: 110
az_max: 280
```

---

## `el_min`

Minimum sun height before activation.

```yaml
el_min: 5
```

Helps avoid triggers at sunrise/sunset.

---

# 🧭 How to Determine the Right Azimuths

## Simple Method

Use:

* Home Assistant Developer Tools
* the `sun.sun` entity
* observe the values when the sun is actually hitting the window

Useful attributes:

```yaml
azimuth
elevation
```

---

# 🐞 DEBUG Mode

The debug mode relies on:

```yaml
input_boolean.debug_protection_solaire
```

When it is enabled:

* a persistent notification is generated every minute
* all computed conditions are visible

Very useful for:

* calibrating azimuths
* adjusting thresholds
* understanding behaviors

---

# 🌡️ Heat-Wave Management

This automation uses:

```yaml
location: paris
sensor.{{ location }}_weather_alert
```

and checks the levels:

* yellow
* orange
* red

You can replace this sensor with your own weather source.

---

# 🔧 Dependencies

## Required Entities

### Sun

```yaml
sun.sun
```

---

### UV

Example:

```yaml
location: paris
sensor.{{ location }}_uv
```

---

### Indoor Temperatures

One sensor per room/window.

---

### Controllable Blinds

`cover.*` entities

---

# 🚀 Installation

## 1. Create the Helpers

Create one `input_boolean` per window.

Example:

```yaml
input_boolean:
  protection_solaire_cuisine_active:
```

---

## 2. Create the DEBUG Helper

```yaml
input_boolean:
  debug_protection_solaire:
```

---

## 3. Add the Scripts

Create:

* `script.protection_solaire_volet`
* `script.fin_de_protection_solaire_volet`

---

## 4. Add the Automation

Import the YAML in:

```text
Settings → Automations & Scenes
```

---

# 💡 Calibration Tips

## Start with DEBUG Enabled

Observe:

* azimuths
* elevations
* actual exposure times

---

## Adjust Gradually

Modify:

* `az_min`
* `az_max`
* `el_min`

until the behavior is perfect.

---

# 🔋 Possible Improvements

## Add Outdoor Temperature

To avoid unnecessary closing.

---

## Add Presence Detection

Protect only when the home is occupied.

---

## Add Weather Forecasting

Anticipate thermal peaks.

---

## Add a Winter Mode

Use free solar gains.

---

# 🏠 Real-World Use Case

This automation is particularly effective for:

* large bay windows
* highly glazed RT2012 homes
* through-apartments
* west-facing bedrooms
* houses with high thermal inertia

---

# 📜 License

MIT

---

# ❤️ Thanks

Thanks to the community:

* Home Assistant
* Zigbee2MQTT
* ESPHome
* and all fans of free home automation ☀️
