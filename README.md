# Biometric Access Control System (C Case Study)

A keypad, fingerprint and SD-logging door access system for apartments, with a neural-network risk engine, duress PIN and a resident alert panel. Built on an **Arduino MEGA 2560** and simulated in **Wokwi**.

**Team 7**

[▶ Run the simulation on Wokwi](https://wokwi.com/projects/477959871051657217)

---

## Problem

Typical low-cost door locks have these weaknesses:

- PINs are stored in plain text
- A stolen PIN always works
- There is no safe way to call for help under force
- Neighbours are unaware of break-ins

## Scope

- PIN, fingerprint and one-time guest PIN entry
- Admin menu for residents
- SD card logs and typing profiles
- Alerts to all flats, with one entrance door

## Existing vs Proposed

| Feature | Existing | Proposed |
| --- | --- | --- |
| PIN storage | Plain text or fixed code | Salted hash on SD card |
| Authentication | PIN only, always trusted | PIN, fingerprint, guest PIN + AI risk check |
| Under force | No option | Duress PIN, silent alarm to all flats |
| Brute force | Fixed or no lockout | Lockout 30 to 240 s |
| Break-in warning | Local buzzer | Siren, SMS to every flat, notice board |
| Visitors | Share resident PIN | One-time PIN, valid 5 minutes |

## Modules

1. **Authentication**: keypad PIN, fingerprint, guest PIN
2. **AI risk engine**: typing profile + neural network
3. **Alert and duress**: siren, SMS, silent alarm
4. **Lockout and tamper**: escalating lockout, fail-secure door
5. **Admin**: add, change, delete users, generate guest PIN
6. **Storage and display**: SD logs, LCD, notice board

## How It Works

`Read credential → Hash & identify → AI risk score → Decision`

- **Duress PIN**: your real PIN with the last digit +1 opens the door and sends a silent alarm to every other flat.
- **Guest PIN**: single use, expires in 5 minutes.
- **Lockout**: 3 failures trigger the siren, an alert to all flats and a lockout of 30 × 2^level seconds (30 → 60 → 120 → 240 s).
- **Tamper input**: the door is forced shut, an alert is sent and a 30 s lockout starts.

## AI Risk Engine

| Item | Detail |
| --- | --- |
| Inputs | Typing rhythm deviation, failed attempts / 3, tamper in last 60 s |
| Network | 3–4–1, ReLU hidden layer, sigmoid output |
| Output | Risk score from 0 to 1 |
| Learning | Rhythm learned per user at rate 0.2, saved to `PROFILES.TXT` |
| Activation | Active after 3 learned entries; entry rejected at risk ≥ 0.70 |

## Hardware / Simulation Circuit (Wokwi)

| Component | Connection |
| --- | --- |
| Arduino MEGA 2560 | Controller |
| 16×2 and 20×4 I2C LCDs | Addresses `0x27` / `0x26` |
| 4×4 keypad | Pins 9–2 |
| Servo (door lock) | Pin 10 |
| SD card module | CS pin 53 |
| Buzzer and 2 LEDs | Pins 11, 12, 24 |
| 3 push buttons | Pins 13, A0, 22 |

## Repository Contents

| File | Description |
| --- | --- |
| `sketch.ino` | Main Arduino code |
| `diagram.json` | Wokwi circuit wiring |
| `libraries.txt` | Required libraries |
| `wokwi-project.txt` | Wokwi project link |
| `finalone.zip` | Complete Wokwi project export |
| `TEAM7_PPT.pptx` | Project presentation |

**Libraries:** SD, Keypad, LiquidCrystal I2C, Servo, Fingerprint Scanner TTL

## How to Run

**Online (easiest):** open the [Wokwi simulation](https://wokwi.com/projects/477959871051657217) and click the green **Start** button.

**From the files:**

1. Go to [wokwi.com](https://wokwi.com) and create a new **Arduino Mega** project.
2. Replace the contents of `sketch.ino`, `diagram.json` and `libraries.txt` with the files from this repo (or import `finalone.zip`).
3. Click **Start Simulation**.

## Performance

- **1 in 10,000** chance of guessing a 4-digit PIN per try
- **~45 guesses per hour** worst case after lockouts
- **0.12 vs 0.97**: typical risk score for normal vs abnormal entry
- Non-blocking timers keep the keypad and tamper input live during alerts
- Hashed PINs mean SD card theft does not reveal codes

## Limitations

- SMS and fingerprint are simulated
- Neural network weights are set by hand
- 32-bit hash
- Maximum of 8 residents

## Future Work

- Real fingerprint module
- GSM or Wi-Fi alerts
- Real-time clock (RTC)
- Model trained on real typing data

## Conclusion

Hashed PINs, a duress code, a guest PIN and building-wide alerts are achievable on a low-cost board, and the AI risk check stops correct-but-suspicious entries.
