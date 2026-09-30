# Exhibit Voice: How It Works in a Real Museum

In the prototype, an operator drags the visitor around the room. In a real museum, the visitor's phone has to work out on its own how close it is to each exhibit. There are several ways to do that.

---

## 1. BLE beacons (main option, cheap and proven)

**What it is:** a small battery-powered Bluetooth Low Energy transmitter (about the size of a coin or a matchbox) placed next to each exhibit. Several times per second it broadcasts a short ID, using the **iBeacon** (Apple) or **Eddystone** (Google) format.

**How the phone measures distance:**
- The phone reads the **RSSI** (Received Signal Strength Indicator) of each beacon. A stronger signal means the beacon is closer.
- Each beacon broadcasts its **"Measured Power"**, the expected RSSI at 1 m. From that the phone can estimate distance:
  `distance ≈ 10 ^ ((MeasuredPower − RSSI) / (10 × n))`, where *n* ≈ 2–4 depending on the room.
- The raw signal is noisy (people's bodies absorb it, and walls and glass reflect it), so we **smooth it** with a moving average or a Kalman filter.
- In practice this gives zones rather than exact meters: **Immediate (<0.5 m) · Near (1–3 m) · Far (>3 m)**. That fits our idea well:
  - Far → ambient sound
  - Near → the sound grows louder
  - Immediate → the story starts

**Pros:** cheap (about €10–30 per beacon), batteries last 1–5 years, works on every smartphone, easy to install.
**Cons:** accuracy is about 1–3 m, the signal needs calibrating for each room, and the system can't tell which direction the visitor is facing.

**Important limitation:** iPhone browsers can't scan for Bluetooth beacons, and on Android it's still experimental. So a **pure web app with no install can't read beacons**. Options:
- a lightweight native app (Flutter or React Native), or
- the museum's own **rental devices** with the app preinstalled, or
- a web app plus one of the fallbacks in section 4.

---

## 2. Indoor positioning systems (IPS): "local GPS inside the museum"

Normal GPS **does not work indoors**. Satellite signals are blocked by the roof and walls, and the accuracy (about 5–10 m) is too low anyway. Instead, a museum can install its own local positioning system.

| Technology | How it works | Accuracy | Notes |
|---|---|---|---|
| **BLE trilateration** | The phone hears 3+ beacons and calculates its position from their signal strengths | 1–3 m | Same beacons as above, just placed more densely |
| **BLE 5.1 Direction Finding (AoA/AoD)** | Antenna arrays measure the *angle* of the signal | ~0.5–1 m | Newer standard; needs special receivers ("locators") |
| **UWB (Ultra-Wideband)** | Measures the signal's *time of flight* between the phone and fixed "anchors" | **10–30 cm** + direction | Most precise option. Supported by iPhone 11+ and many flagship Android phones. Anchors cost more (about €100–300 each) |
| **Wi-Fi RTT (802.11mc)** | Measures the round-trip time to Wi-Fi access points | 1–2 m | Android only; the access points must support it |
| **Wi-Fi fingerprinting** | A map of Wi-Fi signal patterns is recorded in advance; the phone matches what it currently sees | 2–5 m | Uses the existing Wi-Fi network, no new hardware |
| **Magnetic field positioning** | The steel in a building creates a unique magnetic "fingerprint" that the phone's compass can match | 1–3 m | No hardware at all (e.g. IndoorAtlas) |
| **Visual positioning (camera)** | The phone camera recognizes the exhibit or room | High | Visitors have to hold the phone up, which is less natural |

**UWB** is our "future" option. It knows exactly where the visitor is and **which way they're facing**, which makes real **spatial audio** possible: the sword clashes really come from the helmet on your left.

---

## 3. Recommended roadmap

1. **Hackathon (now):** a web app where the operator plays the beacon (Node.js + Socket.IO + Web Audio API).
2. **Pilot in one room:** 3–10 BLE beacons, a simple mobile app, zone-based audio (Far / Near / Immediate), and calibration for each exhibit.
3. **Production:** BLE across the whole museum, a curator dashboard for uploading sounds and setting zones, and analytics (heatmaps of where visitors stop and which exhibits lose them).
4. **Premium:** UWB anchors for precise position and direction, plus full spatial audio with head tracking (e.g. AirPods).

---

## 4. Fallbacks for "no app" and accessibility

- **QR / NFC tags** at each exhibit: tapping one instantly starts the story in the web app. This works on any phone without installing anything.
- **Rental devices** with headphones for visitors who don't have a smartphone.
- **Audio navigation for visually impaired visitors:** the soundscape itself guides them ("follow the sound"), combined with BLE zones.

---

**Pitch line:** *"In production, BLE beacons provide the position (UWB for centimeter precision); here our operator plays the beacon."*
