[README.md](https://github.com/user-attachments/files/29420798/README.md)
# 🤖 Mini VibroBot — Single-Motor BLE Micro Robot

<div align="center">

![Status](https://img.shields.io/badge/Status-In%20Development-orange)
![MCU](https://img.shields.io/badge/MCU-BL602%20%7C%20Ai--WB2--M1--I-blue)
![Control](https://img.shields.io/badge/Control-BLE%205.0-purple)
![Size](https://img.shields.io/badge/Size-~20mm%20×%2020mm-green)
![Motor](https://img.shields.io/badge/Motor-Coin%20ERM%2010mm-red)

**A stamp-sized BLE-controlled vibrobot that steers using only one reversible ERM motor — no wheels, no servos, no gears.**

[How It Works](#-how-it-works) · [Math](#-the-physics) · [Hardware](#-hardware) · [Build](#-build-guide) · [Firmware](#-firmware)

</div>

---

## What Is This?

Most robots use multiple motors, wheels, or servos to steer. This project does it with **one motor** and **pure software**.

A single coin ERM (Eccentric Rotating Mass) motor — the flat pancake type from phone vibration alerts — is mounted at a 20-30° tilt on a chassis the size of a postage stamp. The tilt is critical: it converts vertical vibration into a directional horizontal force. By controlling **when** the motor spins clockwise vs counterclockwise, and **for how long**, the robot can:

- Move **forward** (rapid CW↔CCW switching)
- Curve **right** (sustained CW)  
- Curve **left** (sustained CCW)
- **Stop** (motor off)

All controlled wirelessly over **BLE 5.0** from a phone.

---

## 📐 How It Works

### The Stick-Slip Principle

When the ERM motor spins, the eccentric mass creates a rotating centrifugal force. Because the motor is **tilted at an angle** (30–45°), this force has two components:

```
    Coin ERM Motor (tilted 20-30° above chassis)
         _____
        /  ○  \   ← coin motor (flat pancake, 10mm dia)
       /________\
          ╱ ← tilt angle 20-30°
         ╱
━━━━━━━━╱━━━━━━━━━━  ← chassis (2cm × 2cm)
●   ●   ●   ●   ●   ← pen nib contacts (pentagon)
```

| Phase | What Happens |
|-------|-------------|
| **Slip** | Horizontal force > static friction → robot slides forward |
| **Stick** | Force drops below friction → robot holds position |
| **Net result** | Robot crawls forward one tiny step per vibration cycle |

### Steering Without a Second Motor

This is the key insight. When the ERM spins **clockwise**, the rotating force vector sweeps in one direction, creating a **yaw torque** bias — the robot curves right. Reverse to **counterclockwise** and the yaw torque mirrors — robot curves left.

Rapid CW↔CCW switching cancels the yaw torques out → straight line.

```
Sustained CW  →  curves RIGHT  ↷
Sustained CCW →  curves LEFT   ↶
Rapid CW/CCW  →  STRAIGHT      ↑
```

> **Validated by:** SimoBot (4.76g, 20mm diameter — peer reviewed), Pro Know single-motor robot (ESP-based, open source)

---

## 🔬 The Physics

### Simple Version

The eccentric mass spins in a circle. That spinning creates a centrifugal force that pushes the robot. The tilt angle decides which direction the push goes.

### Full Mathematical Treatment

#### 1. Centrifugal Force

$$F_c = m_e \cdot \omega^2 \cdot r$$

Where:
- $m_e$ = eccentric mass (kg)
- $\omega = \frac{2\pi \cdot \text{RPM}}{60}$ = angular velocity (rad/s)
- $r$ = eccentricity (distance of mass from center, m)

For a 4mm ERM at ~12,000 RPM:
$$\omega = \frac{2\pi \times 12000}{60} \approx 1257 \text{ rad/s}$$

#### 2. Force Decomposition (Tilted Motor)

$$F_{\text{vertical}} = F_c \cdot \cos(\theta)$$
$$F_{\text{horizontal}} = F_c \cdot \sin(\theta)$$

At tilt angle $\theta = 25°$ (coin ERM optimal):
- $F_h = F_c \cdot \sin(25°) \approx 0.423 \cdot F_c$ (propulsion)
- $F_v = F_c \cdot \cos(25°) \approx 0.906 \cdot F_c$ (modulates normal force)

> **Note:** Coin ERM vibrates primarily in the Z-axis when flat. The 20-30° tilt converts this into a usable horizontal propulsion component. **Never mount flat — it kills directional control.**

#### 3. Stick-Slip Condition

Robot moves only when horizontal force exceeds static friction:

$$F_c \cdot \sin(\theta) > \mu_s \cdot (Mg - F_c \cdot \cos(\theta))$$

Solving for minimum ω:

$$\omega_{\min} = \sqrt{\frac{\mu_s \cdot Mg}{r(sin\theta + \mu_s \cos\theta) \cdot m_e}}$$

**Physical meaning:** Too slow = robot just sits there vibrating. Above $\omega_{\min}$ = robot moves.

#### 4. Net Displacement Per Cycle

$$\Delta x = \frac{(F_h - \mu_k \cdot N) \cdot t_{\text{slip}}^2}{2M}$$

Where $N = Mg - F_c\cos(\theta)$ is the reduced normal force during slip phase.

#### 5. Optimal CW↔CCW Switching Frequency

The motor needs time to reach full speed before reversing:

$$f_{\text{opt}} = \frac{1}{2\tau_m}$$

where $\tau_m \approx 50\text{–}100\text{ ms}$ for a 4mm ERM, giving:

$$f_{\text{opt}} \approx 2\text{–}4 \text{ Hz} \quad (250\text{–}500\text{ ms per phase})$$

**Confirmed by SimoBot:** 500ms CW / 500ms CCW produced straight-line motion.

#### 6. Yaw Torque (Steering)

$$\tau_{\text{yaw}} = F_c \cdot d_{\text{offset}}$$

Where $d_{\text{offset}}$ is lateral distance from motor to chassis centroid. Reversing $\omega$ direction reverses $\tau_{\text{yaw}}$ sign → opposite curve direction.

#### 7. Power Budget

| Source | Calculation | Value |
|--------|-------------|-------|
| Battery energy | 80mAh × 3.7V | 296 mWh |
| HT7737C boost | 3.7V → 5V at ~85% eff | ~4.25V output |
| CP6208 drop | 1.0–1.6V at 100mA | Motor gets ~2.6–3.4V |
| Motor current | ~80–120mA | — |
| BL602 BLE | ~15–20mA | — |
| Total draw | ~100–140mA peak | — |
| Runtime estimate | 80mAh ÷ 120mA | ~40 min |

---

## 🔧 Hardware

### Bill of Materials

| Component | Part | Purpose |
|-----------|------|---------|
| MCU | Ai-WB2-M1-I (BL602) | BLE 5.0 + WiFi brain |
| H-Bridge | CP6208DTR | Motor CW/CCW control |
| Motor | Coin ERM (10mm dia, 3V) | Stick-slip locomotion, low noise |
| Boost | HT7737C SOT-89-3 | 3.7V → 5V for motor |
| Battery | 80mAh 20-30C LiPo | Power |
| Contacts | 5× ballpoint pen nibs | Pentagon ground contact |
| Cap | 100µF | Motor spike buffer |
| Cap | 0.1µF | CP6208 bypass |

### Circuit Connections

```
LiPo (+) ──→ HT7737C IN ──→ 5V OUT ──→ CP6208 VCC
LiPo (+) ──→ BL602 Pin3 (VCC 3.3V via LDO)
LiPo (-) ──→ Common GND

BL602 Pin19 (IO3) ──10kΩ──GND
                  └──────────→ CP6208 AIN
BL602 Pin26 (IO1) ──10kΩ──GND  
                  └──────────→ CP6208 BIN

CP6208 AOUT ──→ Motor (+)
CP6208 BOUT ──→ Motor (-)
```

### Motor Control Truth Table

| IO3 (AIN) | IO1 (BIN) | Motor | Robot |
|-----------|-----------|-------|-------|
| HIGH | LOW | CW | Curves Right |
| LOW | HIGH | CCW | Curves Left |
| Rapid toggle | Rapid toggle | Oscillating | Straight |
| LOW | LOW | Stop | Stop |

### Chassis Design

```
    Top view (20mm × 20mm)
    ┌─────────────────┐
    │   [BL602 PCB]   │
    │   ___           │
    │  /○ \  ← coin   │
    │ /____\ motor    │
    │  ╱ tilted 25°   │
    └─────────────────┘

    Side view — motor tilt is everything
    ┌──────┐
    │ coin │ ← 10mm dia, 3mm thick
    └──┬───┘
       │ ← tilted 20-30° from horizontal
    ───┴──────────────  chassis
    ●  ●  ●  ●  ●      pen nib pentagon

    Bottom view — pentagon nib contacts
         ●
       ●   ●
       ●   ●
    (omnidirectional low-friction steel balls)
```

> **Critical mounting note:** Coin ERM must be tilted 20-30° off horizontal. Flat mounting = only vertical vibration = no steering, weak locomotion. The tilt is the entire physics basis of the project.

---

## 🏗 Build Guide

### Phase 1 — Bench Test (Current Stage)
- [x] BL602 module communication via UART confirmed
- [x] AT firmware responding (BLE advertising confirmed)
- [x] CP6208 wired and tested
- [ ] Custom GPIO firmware flashed
- [ ] Motor CW/CCW bench tested
- [ ] Locomotion direction verified

### Phase 2 — Robot Assembly
- [ ] Pentagon pen nib contacts attached
- [ ] ERM motor mounted at 35° angle
- [ ] HT7737C boost circuit integrated
- [ ] LiPo connected
- [ ] Chassis weight measured

### Phase 3 — BLE Control
- [ ] Custom BLE firmware flashed
- [ ] Phone app / web BLE controller built
- [ ] Forward/Left/Right/Stop commands working
- [ ] Switching frequency tuned for surface

---

## 💻 Firmware

### Planned Control Protocol (BLE)

```
Command byte → Motor action
0x46 ('F') → Forward (rapid CW/CCW 250ms intervals)
0x52 ('R') → Right (sustained CW)
0x4C ('L') → Left (sustained CCW)
0x53 ('S') → Stop
```

### GPIO Control Logic (Pseudo)

```c
void forward() {
    while(moving) {
        gpio_write(IO3, HIGH); gpio_write(IO1, LOW);  // CW
        delay(250);
        gpio_write(IO3, LOW);  gpio_write(IO1, HIGH); // CCW
        delay(250);
    }
}

void right() {
    gpio_write(IO3, HIGH);
    gpio_write(IO1, LOW);  // Sustained CW
}

void left() {
    gpio_write(IO3, LOW);
    gpio_write(IO1, HIGH); // Sustained CCW
}
```

---

## 📚 References

1. **SimoBot:** "SimoBot: An underactuated miniature robot driven by a single motor" — 4.76g, 20mm, peer-reviewed validation of this exact concept
2. **Pro Know Robot** — Open source ESP-based single-motor vibrobot using DRV8212 + pager motor
3. **PufferFace Robot (PFR)** — Soft-body vibrobot using ERM for pipeline navigation
4. **Hexbug Nano** — Commercial bristlebot (unidirectional, for comparison)

---

## 👤 Author

**Ishant Jaiswal** — B.Tech Robotics & Automation, 3rd Year  
GitHub: [@Ishu1519](https://github.com/Ishu1519)

---

## 📄 License

MIT License — build it, improve it, share it.
