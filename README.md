# Pole Climbing Robot

A Bluetooth-controlled pole climbing robot designed for inspection of single-phase electrical distribution lines and pole-mounted equipment.

The robot can climb vertically along a utility pole while carrying an inspection arm and electrical sensing system. A remote operator can manually control the robot through Bluetooth while monitoring electrical parameters from a safe distance.

This project was developed as a proof-of-concept inspection robot for electrical utility applications where manual climbing can be dangerous, time-consuming, and expensive.

---

# Project Demonstration

![Pole Climbing Robot](images/Pole_Climbing_Robot.gif)

The robot can:

- Climb vertically on a pole
- Move upward and downward
- Rotate an inspection arm
- Rotate the sensing head
- Measure live AC voltage
- Send voltage data wirelessly
- Be controlled through a Bluetooth interface

---

# Why This Project?

Inspection of utility poles often requires technicians to:

- Climb poles manually
- Work at dangerous heights
- Operate near live electrical lines
- Perform visual inspections
- Verify line voltage

These activities involve significant safety risks.

The goal of this project was to create a simple robotic platform capable of:

- Climbing a utility pole
- Carrying sensors
- Measuring electrical parameters
- Allowing remote operation from the ground

---

# Features

## Vertical Pole Climbing

The robot can climb vertically on cylindrical poles using a motorized traction system.

---

## Bluetooth Remote Control

The entire robot can be controlled wirelessly using Bluetooth.

The ESP32 creates a Bluetooth device named:

```text
PoleRobo
```

allowing direct connection from a smartphone or custom controller.

---

## AC Voltage Monitoring

The robot includes a ZMPT101B voltage sensing module capable of measuring AC voltage levels.

Measured voltage values are transmitted wirelessly back to the operator.

---

## Pan Mechanism

The inspection head can rotate independently using a servo motor.

This allows the operator to inspect different areas without repositioning the robot.

---

## Inspection Arm

A second servo-controlled arm provides adjustable sensor positioning.

The arm can be moved remotely to better align sensors with electrical equipment.

---

## Wireless Telemetry

Voltage measurements are automatically transmitted every few seconds.

This allows the operator to monitor line conditions in real time.

---

# Hardware Used

| Component | Purpose |
|------------|----------|
| ESP32 | Main Controller |
| ZMPT101B | AC Voltage Measurement |
| DC Gear Motors | Pole Climbing Mechanism |
| Motor Driver | Motor Control |
| Servo Motor | Sensor Head Rotation |
| Servo Motor | Inspection Arm |
| Bluetooth Module (ESP32 Internal) | Wireless Control |
| Battery Pack | Power Source |
| Mechanical Climbing Frame | Pole Attachment |

---

# System Architecture

```text
                 Smartphone Controller
                           │
                     Bluetooth
                           │
                           ▼

                      ESP32 MCU
                           │
     ┌─────────────────────┼─────────────────────┐
     │                     │                     │
     ▼                     ▼                     ▼

 Motor Driver        Servo System      Voltage Sensor

     │                     │                     │

 Climbing Motors     Inspection Arm    AC Monitoring

     │                     │                     │

     └─────────────────────┴─────────────────────┘
                           │

                    Telemetry Output
```

---

# Working Principle

The operator connects to the robot using Bluetooth.

Control commands are transmitted wirelessly to the ESP32.

The ESP32 then:

1. Decodes the incoming command
2. Controls the climbing motors
3. Moves the inspection servos
4. Reads AC voltage values
5. Sends sensor data back to the controller

---

# Bluetooth Control System

From the firmware:

```cpp
BluetoothSerial ESP_BT;
```

The Bluetooth interface is initialized as:

```cpp
ESP_BT.begin("PoleRobo");
```

Once paired, the robot listens continuously for control commands.

---

# Motion Control Algorithm

The robot uses two motor channels.

Motor control is performed through:

```cpp
motor(a,b);
```

where:

```text
1  = Forward
0  = Stop
-1 = Reverse
```

---

## Forward Climb

Bluetooth Command:

```text
F
```

Firmware:

```cpp
motor(1,1);
```

Result:

```text
Both motors rotate forward
Robot climbs upward
```

---

## Reverse Movement

Bluetooth Command:

```text
B
```

Firmware:

```cpp
motor(-1,-1);
```

Result:

```text
Both motors rotate reverse
Robot climbs downward
```

---

## Stop

Bluetooth Command:

```text
S
```

Firmware:

```cpp
motor(0,0);
```

Result:

```text
Robot stops immediately
```

---

# Motor Driver Logic

From the code:

```cpp
void motor(int a, int b)
{
    digitalWrite(IN1, a > 0);
    digitalWrite(IN2, a < 0);

    digitalWrite(IN3, b > 0);
    digitalWrite(IN4, b < 0);
}
```

The motor direction is determined by setting the driver input pins appropriately.

---

# Variable Speed Control

The robot supports adjustable climbing speed.

Bluetooth commands:

```text
0
1
2
3
4
5
6
7
8
9
```

are interpreted as speed levels.

Firmware:

```cpp
updateSpeed(
map((x-'0'),0,9,50,255)
);
```

---

## Speed Mapping

| Command | PWM Output |
|----------|-----------|
| 0 | 50 |
| 1 | 72 |
| 2 | 94 |
| 3 | 116 |
| 4 | 138 |
| 5 | 160 |
| 6 | 182 |
| 7 | 204 |
| 8 | 226 |
| 9 | 255 |

This allows smooth control of climbing speed.

---

# Inspection Head Control

The robot includes a servo-based rotating inspection head.

From the code:

```cpp
Servo base;
```

The servo is attached on:

```cpp
GPIO 27
```

---

## Rotation Commands

The firmware accepts:

```text
k
l
m
n
o
p
q
r
s
t
```

Each command corresponds to a different servo position.

Firmware:

```cpp
base.write(
map((x-'k'),
0,
9,
10,
180)
);
```

---

## Rotation Range

```text
10°
to
180°
```

This provides a wide inspection field.

---

# Inspection Arm Control

The robot includes a second servo-controlled arm.

From the code:

```cpp
Servo arm;
```

attached on:

```cpp
GPIO 25
```

---

## Arm Up

Command:

```text
D
```

Result:

```cpp
armMove = 1;
```

The arm gradually moves upward.

---

## Arm Down

Command:

```text
T
```

Result:

```cpp
armMove = -1;
```

The arm gradually moves downward.

---

## Stop Arm

Command:

```text
N
```

Result:

```cpp
armMove = 0;
```

The arm stops moving.

---

# Arm Motion Algorithm

Instead of jumping directly between positions, the arm moves incrementally.

Firmware:

```cpp
armPos++;
```

or

```cpp
armPos--;
```

with:

```cpp
delay(10);
```

between updates.

This creates smooth mechanical movement and reduces stress on the servo.

---

## Arm Limits

The firmware restricts motion to:

```text
30°
to
150°
```

preventing mechanical damage.

---

# AC Voltage Monitoring System

One of the most important features of the robot is live voltage measurement.

The project uses:

```text
ZMPT101B
```

voltage sensor module.

---

# Sensor Initialization

From the code:

```cpp
ZMPT101B voltageSensor(33,50.0);
```

Input pin:

```text
GPIO 33
```

Grid frequency:

```text
50 Hz
```

---

# Sensor Calibration

Firmware:

```cpp
#define SENSITIVITY 480.0f
```

and:

```cpp
voltageSensor.setSensitivity(
SENSITIVITY
);
```

This sensitivity value was selected for proper voltage conversion accuracy.

---

# Automatic Voltage Reporting

The robot automatically sends voltage readings every:

```text
5000 ms
```

Firmware:

```cpp
if(millis()-timer>5000)
```

---

## Measurement Process

```text
Read RMS Voltage
       │
       ▼

Convert To Integer
       │
       ▼

Create String
       │
       ▼

Transmit Via Bluetooth
```

---

# Wireless Telemetry Protocol

Voltage values are transmitted using:

```cpp
sendString(String(voltage));
```

---

The transmission function:

```cpp
void sendString(String data)
{
    for(...)
        ESP_BT.write(...);

    ESP_BT.write(';');
}
```

produces packets such as:

```text
220;
221;
219;
223;
```

The semicolon acts as an end-of-packet marker.

---

# Applications

- Electrical Pole Inspection
- Distribution Line Monitoring
- Educational Robotics
- Remote Voltage Monitoring
- Utility Maintenance Research
- Infrastructure Inspection
- Climbing Robot Research
- Smart Utility Systems

---

# Repository Structure

```text
Pole_Climbing_Robot/

├── Pole_Climbing_bot.ino
├── motor.ino
├── data.ino
│
├── images/
│   └── Pole_Climbing_Robot.gif
│
└── README.md
```

---

# Future Improvements

Potential future upgrades include:

- Autonomous Pole Climbing
- Computer Vision Inspection
- Thermal Camera Integration
- Live Video Streaming
- Fault Detection Algorithms
- GPS Logging
- LTE Connectivity
- Cloud Monitoring Dashboard
- AI-Based Defect Recognition
- Multi-Sensor Inspection Suite

---

# Developer

**Developed by:**  
Fazle Elahi Tonmoy

Robotics Engineer  
Founder, Robodemy

Specializing in:

- Robotics Education
- Embedded Systems
- IoT Development
- Autonomous Systems
- STEM Innovation

---

# License

This project is released for educational and research purposes.

You are welcome to study, modify, and adapt the project for learning, teaching, and non-commercial applications.

Attribution is appreciated when using this project as a reference for educational or research work.