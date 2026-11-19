# Globe Electric (Tuya) Lights in Home Assistant

Globe Electric smart bulbs and fixtures run on Tuya hardware. The **Globe Suite** app is a
white-labelled Tuya app, and that is where most of the setup pain comes from: Home Assistant's
Tuya integration and the Tuya developer portal can't log in to a Globe Suite account. The fix is
to move the lights into the **Smart Life** app (the generic Tuya app). After that, every
integration works.

## Recommended path (local control, no developer account)

### 1. Move the lights from Globe Suite to Smart Life
1. Install **Smart Life - Smart Living** (publisher **Volcano Technology Limited**, Tuya's own app; **Tuya Smart** by Tuya Inc. also works) on your phone and create an account. Pick the **same
   country/region** you'll use everywhere else. Region mismatch is the most common cause of
   "no devices found".
2. In Globe Suite, open each light, tap **✎ / settings → Remove Device**.
3. Factory-reset the bulb into pairing mode: turn it **off/on 3 times** (about 1 second each) until it
   blinks quickly. For a slow blink (AP mode), do it 3 more times.
4. In Smart Life, tap **+ → Add Device**. It normally auto-discovers the bulb over Bluetooth/Wi-Fi.
   Your phone must be on **2.4 GHz** Wi-Fi during pairing (Globe bulbs don't support 5 GHz).
5. Check that you can control each light from Smart Life before going further.

> Tip: give the lights their final names in Smart Life now. The names carry over into Home Assistant.

### 2. Install Tuya Local via HACS
[Tuya Local](https://github.com/make-all/tuya-local) controls the bulbs over your LAN, so the
lights keep working when your internet or Tuya's cloud is down, and they respond quickly.

1. Install [HACS](https://hacs.xyz) if you don't already have it.
2. HACS → search **Tuya Local** → Download → **restart Home Assistant**.

> ⚠️ Don't confuse it with **LocalTuya** (rospogrigio/localtuya), a different integration with a similar
> name that only offers a Tuya IoT developer-account login. In HACS, the one you want is
> **Tuya Local** by **make-all**.

### 3. Add the lights
1. **Settings → Devices & Services → Add Integration → Tuya Local**.
2. Choose **Smart Life cloud-assisted setup**. Enter the **User Code** from Smart Life
   (**Me → ⚙ Settings → Account and Security → User Code**), then scan the QR code shown in
   Home Assistant with the Smart Life app.
3. Pick a light. Tuya Local fetches its device ID and **local key** for you, finds it on your
   network, and suggests a matching device config. Globe bulbs usually match
   *"RGBCW light"* or *"Simple light / dimmable"*. Accept the suggestion and repeat for each light.

Once a light is added, it no longer needs the cloud. The cloud login is only used to fetch the keys.

## Alternative: the official Tuya integration (cloud)
Easiest to set up, but every command goes through Tuya's cloud.

1. Complete **step 1** above (the lights must be in Smart Life, not Globe Suite).
2. **Settings → Devices & Services → Add Integration → Tuya**, enter your Smart Life **User Code**,
   and scan the QR code with Smart Life.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Device shows as unavailable in Tuya Local | Give the bulb a **DHCP reservation** in your router. Tuya Local connects by IP. |
| "Invalid key" / connection fails after it worked | The bulb was re-paired, which issues a **new local key**. Delete the device in HA and add it again. |
| Can't pair to Smart Life | Turn off your phone's mobile data and disable 5 GHz/band-steering (or use a 2.4 GHz-only SSID) during pairing. |
| HA and bulbs on different VLANs | Allow TCP **6668** and UDP **6666/6667** from HA to the IoT VLAN. |
| Colours/white temp behave oddly | In Tuya Local, reconfigure the device and pick a different suggested config (e.g. RGBCW vs RGBW). |
| Smart Life app won't control it locally either | Check the bulb has the latest firmware (Smart Life → device → ⋯ → Check for firmware update). |

## Manual key extraction (fallback)
If cloud-assisted setup fails, get the keys with [tinytuya](https://github.com/jasonacox/tinytuya):

1. Create a free project at [iot.tuya.com](https://iot.tuya.com) (Cloud → Development →
   Create Cloud Project, using the same data centre as your Smart Life region). Then under
   **Devices → Link App Account**, scan the QR code with **Smart Life**.
2. On a computer on the same network:
   ```bash
   pip install tinytuya
   python -m tinytuya wizard   # enter the project's Access ID / Secret and region
   ```
3. `devices.json` now lists each bulb's `id`, `key` (local key), `ip` and protocol `version`.
   In Tuya Local, choose **manual setup** and enter those values.

Keep `devices.json` private, because the local keys give full control of your lights.
