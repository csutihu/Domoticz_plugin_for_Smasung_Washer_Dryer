# Samsung SmartThings Washer / Dryer – Universal Domoticz Plugin

This Domoticz Python plugin retrieves and displays the **status of Samsung washing machines and dryers**
via the **SmartThings Cloud API**.

The plugin is **universal**:
- it can handle **only a washer**
- **only a dryer**
- or **both washer and dryer**
based solely on which **Device IDs** are configured.

Authentication is handled via **SmartThings OAuth 2.0** (Access Token + Refresh Token with automatic refresh).

---

## Key Features

- **Universal plugin** – washer only / dryer only / washer + dryer
- **Single OAuth2 integration** for all supported devices
- **Stable remaining time logic**  
  Remaining time is shown **only when a cycle is active**, otherwise **0 min**
- **Automation-friendly**  
  Cycle / JobState is exposed as a Domoticz *Text device* (ideal for dzVents)
- **Low API usage** with configurable ON / OFF polling intervals

---

## Operating Modes (Important)

The plugin behavior is controlled by the Device ID fields:

- **Washer only**
  - Fill: *Washer Device ID*
  - Leave empty: *Dryer Device ID*

- **Dryer only**
  - Fill: *Dryer Device ID*
  - Leave empty: *Washer Device ID*

- **Washer + Dryer**
  - Fill **both** Device ID fields

If a Device ID field is empty (or set to `None`), that appliance is **disabled**
and **will not be queried**.

---

## SmartThings OAuth 2.0

For the plugin to work, you require a **Client ID**, **Client Secret**, an initial **Access Token**, and a **Refresh Token** from the SmartThings API.

**GUIDE TO ACQUIRE ALL TOKENS:**

The best guide for generating the OAuth 2.0 credentials and acquiring the necessary tokens is provided in this article (Credit: Shashank Mayya):

➡️ **[SmartThings API: Taming the OAuth 2.0 Beast](https://levelup.gitconnected.com/smartthings-api-taming-the-oauth-2-0-beast-5d735ecc6b24)**

> 📌 **Note:** A PDF copy of this guide (e.g., named `SmartThings_Oauth2.0_by_Shashank_Mayya.pdf`) is available in the GitHub repository alongside the source files, in case the original webpage becomes inaccessible. Windows CLI can be used successfully for the steps described.
---

## Installation

1. Copy files into a Domoticz plugin folder, for example:

   `/home/domoticz/plugins/SmartThings_Washer_Dryer/`

   Required files:
   - `plugin.py`
   - `token_manager.py`

2. Create `st_tokens.json` in the same folder:

```json
{
  "access_token": "YOUR_INITIAL_ACCESS_TOKEN",
  "refresh_token": "YOUR_REFRESH_TOKEN",
  "expiry": 0
}
```

Setting `"expiry": 0` forces an immediate refresh at startup.

3. Restart Domoticz.

---

## Plugin Configuration (Domoticz Hardware)

| Field | Parameter | Description |
|---|---|---|
| SmartThings API URL | Address | Usually `https://api.smartthings.com` |
| Debug | Port | `0` = off, `1` = on |
| ON State Polling Interval (sec) | Mode1 | Polling when **any configured device is ON** |
| OFF State Polling Interval (sec) | Mode5 | Polling when **all configured devices are OFF** |
| SmartThings Client ID | Mode2 | OAuth Client ID |
| SmartThings Client Secret | Mode3 | OAuth Client Secret |
| Washer Device ID | Mode4 | Optional – enable washer |
| Dryer Device ID | Mode6 | Optional – enable dryer |

### Polling Logic
- If **any enabled device** is ON → ON interval
- If **all enabled devices** are OFF → OFF interval

---

## Created Domoticz Devices

### Washer (if enabled)

| Device ID | Name | Type | Description |
|---|---|---|---|
| `WM_Power` | Washer Status (ON/OFF) | Switch | Power state |
| `WM_JobState` | Washing Cycle | Text | Cycle / Job state |
| `WM_Remaining` | Washer Remaining Time (min) | Text | Remaining time, **0 min if idle** |

### Dryer (if enabled)

| Device ID | Name | Type | Description |
|---|---|---|---|
| `DR_Power` | Dryer Status (ON/OFF) | Switch | Power state |
| `DR_JobState` | Drying Cycle | Text | Cycle / Job state |
| `DR_Remaining` | Dryer Remaining Time (min) | Text | Remaining time, **0 min if idle** |

---

## Data Source (SmartThings API)

All data is retrieved from:

`GET /v1/devices/{deviceId}/status`

### Washer
- Power: `components.main.switch.switch.value`
- JobState: `components.main.samsungce.washerOperatingState.washerJobState.value`
- Remaining time: `components.main.samsungce.washerOperatingState.remainingTime.value`

### Dryer
- Power: `components.main.switch.switch.value`
- JobState: `components.main.samsungce.dryerOperatingState.dryerJobState.value`
- Remaining time: `components.main.samsungce.dryerOperatingState.remainingTime.value`

### Remaining Time Rule
Remaining time is considered **valid only if**:
- `JobState != "none"`

Otherwise it is forced to **0 min** to avoid stale or default values.

---

## Automation Example (dzVents / Voice Notification)

Typical usage:
- Trigger on **Washing Cycle** or **Drying Cycle** device
- Detect transition from active state → *No active wash* / *No active dry*
- Execute a shell script that plays a prepared MP3 via Sonos or another TTS system

This keeps the plugin focused on data retrieval
and moves notification logic to dzVents.

---

## Troubleshooting

### Device IDs appear empty in logs
- Ensure the plugin `<plugin key>` contains **no spaces**
- If the plugin definition was changed, remove and re-add the hardware in Domoticz

### Token refresh problems
- Verify `st_tokens.json`
- Ensure OAuth scopes include device read access (e.g. `r:devices:*`)

### Remaining time always 0
- Check that `JobState` is not `"none"`
- Some models report a default remaining time while idle; this is intentionally ignored

---

## Changelog

- **v3.9** – Initial washer + dryer integration
- **v4.0** – Universal plugin (washer only / dryer only / both)
- **v4.1** – Stability fixes, correct parameter handling, JobState-based remaining time

---

## License / Credits

- Built for the Domoticz Python plugin framework
- Uses Samsung SmartThings Cloud API
- Original washer plugin concept by csutihu
