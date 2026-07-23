# My First Robot — Student Guide

**A 2-hour hands-on workshop.** By the end, your robot will drive on its own and stop before it hits a wall — and you'll know how to change its behavior with code.

---

## The Big Idea: Sense → Think → Act

Every robot — from a Roomba to a Mars rover — does the same three things:

| Job | What it means | The part on YOUR robot |
|---|---|---|
| **SENSE** | Notice the world | VL53L0X laser distance sensor (the "eye") |
| **THINK** | Decide what to do | ESP32 microcontroller (the "brain") |
| **ACT** | Do something about it | DRV8833 driver + gear motors (the "muscles" and "wheels") |

Keep this in mind all day. Whenever you're confused, ask yourself: is this part sensing, thinking, or acting?

---

## Step 1 — Check Your Kit

Open your bag and find each part. Check them off as you go:

- [ ] **ESP32 DevKit V1** — the brain. Runs your code, talks to the sensor, drives the motors.
- [ ] **DRV8833 motor driver** — the muscles. Turns weak signals from the brain into strong power for the motors, and lets them run forward *or* backward.
- [ ] **2× yellow TT gear motors with wheels** — the legs. Two independent wheels let the robot go straight, curve, or spin in place.
- [ ] **VL53L0X distance sensor** — the eye. A tiny laser tape measure that works up to about 1.2 m.
- [ ] **4×AA battery holder** — the heart. Powers the motors and the ESP32. No power switch — the batteries only go in when it's time to drive.
- [ ] **4× AA batteries**
- [ ] **Chassis** (the flat base everything mounts to)
- [ ] **Caster wheel or marble** — the third point of contact behind the two wheels.
- [ ] **~12 jumper wires** (male-female and female-female)
- [ ] **WAGO splice connectors** — for sharing battery power and ground between boards. Lift the lever, push the wire in, snap the lever down.
- [ ] **Micro-USB cable** (a *data* cable, not charge-only)
- [ ] **Screw kit:**
  - 2× small M3 hex socket head cap screws — for the lidar (VL53L0X)
  - 2× longer M3 hex socket head cap screws — for the motors
  - 4× M3 nuts — pair with the M3 screws above
  - 4× #2-28 pan head Phillips thread rolling screws — for the ESP32 mount
  - 2× M3.5 flat head Phillips screws + 2× M3.5 nuts — for the battery pack
  - 2× #6 pan head Phillips thread rolling screws — for the caster
- [ ] **Small Phillips screwdriver**
- [ ] **2.5mm Allen wrench**

Missing something? Ask a helper now, not at step 4.

### Why can't the ESP32 spin the motors by itself?

Two reasons. First, an ESP32 pin can supply about 20–40 mA of current, but a gear motor wants **200+ mA** just to start moving. Second, to make a motor spin *backward*, the current has to flow through it in the *opposite direction* — the polarity gets reversed, like putting a battery in backwards. The DRV8833 contains an "H-bridge," a circuit that can do both: deliver the big current and flip its direction. Brain → muscles → wheels.

---

## Step 2 — Build the Body

1. **Mount the motors** to the chassis with the two longer M3 socket head cap screws and M3 nuts (2.5mm Allen wrench). **Leave the wheels off for now** — they get in the way while you work.
2. **Bolt the VL53L0X sensor to the front** of the chassis with the two small M3 socket head cap screws and M3 nuts. It's the robot's eye — it needs a clear view straight ahead, so make sure nothing blocks it.
3. **Insert the DRV8833 motor controller into its slot** on the chassis.
4. **Screw the ESP32 into position** with the four #2-28 pan head Phillips thread rolling screws.
5. **Wire the motor leads** to the DRV8833 outputs: left motor → AOUT1/AOUT2, right motor → BOUT1/BOUT2.
   Don't worry about which wire goes where — if a wheel spins the wrong way later, we fix it in code (or just swap the two wires).
6. **Bolt the battery pack** on top with the two M3.5 flat head Phillips screws and M3.5 nuts.

The caster and wheels come *after* the wiring in Step 3 — a robot without wheels can't roll off your table, and wheels get in the way of your hands.

---

## Step 3 — Wire the Brain

⚡ **Keep the batteries OUT of the holder for this whole step.** There's no power switch — the batteries *are* the switch.

Battery + and ground are *shared* connections — several wires need to join the same point. That's what the **WAGO splices** are for: one splice collects Battery + (feeding DRV8833 VM and ESP32 VIN), another collects all the grounds (Battery −, DRV8833 GND, ESP32 GND). Lift the lever, push the wire in all the way, snap the lever down, then tug gently to make sure it's caught.

Connect everything else with jumper wires:

| From | To (ESP32 pin) | Notes |
|---|---|---|
| DRV8833 AIN1 | GPIO 14 | Left motor direction A |
| DRV8833 AIN2 | GPIO 27 | Left motor direction B |
| DRV8833 BIN1 | GPIO 26 | Right motor direction A |
| DRV8833 BIN2 | GPIO 25 | Right motor direction B |
| DRV8833 AOUT1/2 | Left motor | Polarity doesn't matter — swap later if it spins backward |
| DRV8833 BOUT1/2 | Right motor | Same — swap if needed |
| DRV8833 VM | Battery + | Motor power, ≈6 V from 4×AA — shared through the battery + WAGO splice |
| DRV8833 GND | ESP32 GND **and** Battery − | All grounds meet in the ground WAGO splice — common ground is essential |
| VL53L0X VCC | 3V3 | **NOT 5V** — this sensor is 3.3 V only |
| VL53L0X GND | GND | |
| VL53L0X SDA | GPIO 21 | I²C data |
| VL53L0X SCL | GPIO 22 | I²C clock |
| ESP32 VIN | Battery + (optional) | From the same battery + WAGO splice — lets the battery pack power the ESP32, only after wiring is checked |

**Check the labels:** on this microcontroller — an ESP32 DevKit V1 — the D numbers printed on the board match the GPIO numbers (D14 = GPIO 14). But this is **not always the case** on other boards, so it's important to check the pinouts and datasheets of all your components.

**Cool fact:** the sensor talks to the brain over just two wires (SDA and SCL). That's called **I²C**, and it's a shared bus — you could add more sensors later on the *same* two wires.

### Finish the Body

Once the sensor is wired:

1. **Attach the caster** behind the motors with the two #6 pan head Phillips thread rolling screws.
2. **Push the wheels** onto the motor shafts — they go on last because they get in the way of everything else.

### ✅ Safety Checks Before Powering On

Go through all four. Then have a helper eyeball your wiring **before** any batteries go in.

1. **Common ground?** ESP32 GND, DRV8833 GND, and Battery − must all connect — they should all meet in the ground WAGO splice. Tug each wire to check it's gripped.
2. **VL53L0X on 3.3 V, not 5 V.**
3. **No bare motor wires** touching each other or the chassis.
4. **Batteries stay out of the holder** until a helper has checked your work.

---

## Step 4 — Code: Make It Move

### Get the Arduino IDE Ready

The Arduino IDE is the program that sends code to your robot's brain. Set it up once:

1. Open the **Arduino IDE** on your laptop.
2. **File → Open** and pick the starter sketch (a helper can point you to it — you don't need to type it).
3. Plug the micro-USB cable into the ESP32 and the laptop.
4. **Tell the IDE which board you have:** Tools → Board → esp32 → **DOIT ESP32 DEVKIT V1**. (Or click the dropdown at the top of the window, choose "Select other board and port…", and search for it.)
5. **Tell it which port the robot is on:** Tools → Port. Not sure which entry is your ESP32? Unplug the USB cable, look at the list, plug it back in — the port that appears is the one (Windows: `COM3`-style, Mac: `/dev/cu.usbserial-…`).

You only do this once — the IDE remembers your board and port until you switch laptops.

### The Starter Sketch

```cpp
#include <Wire.h>
#include <Adafruit_VL53L0X.h>

// Motor pins (DRV8833)
const int AIN1 = 14, AIN2 = 27;   // left motor
const int BIN1 = 26, BIN2 = 25;   // right motor

Adafruit_VL53L0X lox;

void drive(int left, int right) {
  // values from -255 to +255
  analogWrite(AIN1, left  > 0 ?  left : 0);
  analogWrite(AIN2, left  < 0 ? -left : 0);
  analogWrite(BIN1, right > 0 ?  right : 0);
  analogWrite(BIN2, right < 0 ? -right : 0);
}

void setup() {
  Serial.begin(115200);
  pinMode(AIN1, OUTPUT); pinMode(AIN2, OUTPUT);
  pinMode(BIN1, OUTPUT); pinMode(BIN2, OUTPUT);
  if (!lox.begin()) { Serial.println("Sensor not found"); while (1); }
}

void loop() {
  VL53L0X_RangingMeasurementData_t m;
  lox.rangingTest(&m, false);
  int d = (m.RangeStatus != 4) ? m.RangeMilliMeter : 9999;
  Serial.println(d);

  if (d > 200) {
    drive(200, 200);          // forward
  } else {
    drive(180, -180);         // spin to find a clear path
    delay(400);
  }
}
```

**What the pieces do:**

- `drive(left, right)` — sets each wheel's speed, from −255 (full reverse) to +255 (full forward). `analogWrite()` sends a PWM signal — a fast on/off pulse that acts like a volume knob for the motor.
- `setup()` — runs once. Sets the pins up and starts the sensor. If the sensor isn't found, it prints a message and stops.
- `loop()` — runs forever. Reads the distance, then decides: path clear → drive forward; obstacle close → spin and look for a way out.

### Upload and Drive

1. Click the **✓ Verify** button (top-left) to compile the code. No red errors in the console at the bottom means the code is good.
2. Click the **→ Upload** button (right next to it). Watch the console: you'll see "Connecting...", then a rising percentage, and finally **"Done uploading."**
3. **"Failed to connect"?** Hold down the **BOOT** button on the ESP32 while the console says "Connecting...", and release it when the upload starts. (Some boards need this nudge every time.)
4. Unplug USB, put the robot on the floor, and pop the batteries into the holder.
5. **One wheel spinning backward?** Totally normal. Swap that motor's two wires at the DRV8833 — or swap its two GPIO numbers in the code. Either fix works.
6. It moves? 🎉 Celebrate. You built a robot.

**Try it:** make both wheels different speeds — `drive(200, 100)`. What happens? One forward, one reverse? One forward, one stopped? This two-wheels-plus-caster setup is called **differential drive**, and it's how you steer with no steering wheel.

---

## Step 5 — Code: Make It See

This is the magic moment — the robot reacts to the world.

The sensor measures distance in **millimeters**. That trips up almost everyone: a threshold of `20` isn't 20 cm — it's 2 cm, so the robot thinks the wall is always far away!

1. Open the **Serial Monitor** — click the magnifying-glass icon in the top-right corner of the IDE (or Tools → Serial Monitor) and set the baud dropdown to **115200**, or the text will be gibberish. With USB plugged in, watch the live distance readings scroll by. Wave your hand in front of the sensor.
2. Find the line `if (d > 200)`. That's the rule: *if the nearest thing is more than 200 mm (20 cm) away, keep driving; otherwise spin.*
3. Upload, put it in front of a wall, and watch it stop and turn.
4. **Tune it.** Try 100 mm. Try 400 mm. What feels right?

---

## When Something Goes Wrong (It Will — That's the Fun Part)

**Golden rule: it's almost never the code.** About 80% of robot problems are wiring or power — a loose jumper, swapped pins, tired batteries. Check power and connections *first*, code *second*.

| Symptom | Likely cause & fix |
|---|---|
| Robot does nothing | Batteries in the holder and the right way around? DRV8833 VM and GND wired? Did the upload actually finish ("Done uploading")? |
| One wheel spins backward | Swap that motor's two leads at the DRV8833 — or flip its two GPIOs in code. |
| Both wheels backward | Reverse the driver output wires (swap each motor's two leads at the DRV8833 outputs) — or reverse the output in the code. |
| Robot drifts left or right | The motors aren't perfectly matched. Lower the PWM on the faster side by 10–20 (e.g., 200 vs 220). |
| Distance reads 8190 or always 0 | Sensor not detected. Check SDA/SCL aren't swapped and the sensor is on 3.3 V (not 5 V). |
| ESP32 keeps rebooting | Brownout — low batteries, or it's powered by USB while motors draw current. Fresh AAs; unplug USB for driving tests. |
| Upload fails: "Failed to connect" | Hold the **BOOT** button while clicking Upload; release when "Connecting..." appears. Make sure the cable is a data cable. |
| Sensor works but robot doesn't stop | Units! The sensor reports **mm**. 200 mm = 20 cm. Print the distance to Serial and check your threshold. |

---

## Finished Early? Level Up

### Easy (5–10 min)
- Tune the threshold and turn time. How close can the robot get before it stops?
- Change the spin direction — does your robot prefer left or right?
- Make it faster or slower by changing the PWM values.

### Medium (10–20 min)
- Add a piezo buzzer on a spare GPIO — beep when an obstacle is close.
- Add an LED that gets brighter as the robot nears a wall (map distance → `analogWrite`).
- Make the spin time proportional to distance — far obstacle, short turn; close obstacle, long turn.

### Advanced (20+ min, or at home)
- Add a **second VL53L0X**. Both share the I²C bus but need different addresses — use the XSHUT pin to start them one at a time.
- Add **Wi-Fi control**: drive the robot from a web page served by the ESP32 itself.
- Log distance readings to Serial and plot them in the Arduino IDE **Serial Plotter** — you've just built a turtle that draws a map.

---

## Safety Notes

- **Eyes:** don't stare directly into the VL53L0X. It's a Class 1 laser (eye-safe under normal conditions), but treating every laser with respect is a good habit for bigger projects later.
- **Batteries:** AAs are very safe, but a short circuit still heats up wires. That's why a helper checks your wiring before power-on.
- **Small parts:** screws and jumpers roll away. Keep them in the parts tray.
- **Driving area:** keep the floor space clear of bags and cables. Robots will bump into each other — that's fine.

---

## Before You Go — Can You Answer These?

1. Point to the part of your robot that **senses**. Now the part that **thinks**. Now the part that **acts**.
2. Why can't the ESP32 spin the motors directly?
3. Your robot is driving backward. What's the fastest fix — code or wiring? (Either is fine!)
4. If you had one more sensor, what would it be — and what would your robot do with it?

**Want to keep coding at home?** Install the free Arduino IDE from arduino.cc, then add ESP32 support: open **Boards Manager** (left sidebar), search "esp32", and install **esp32 by Espressif Systems**. Then open **Library Manager**, search "Adafruit VL53L0X", and click Install (say yes to dependencies). That's the exact setup the workshop laptops use.

If you can answer those, you didn't just build a robot. You *understand* it. Take it home, take the extension sheet, and keep going. 🤖
