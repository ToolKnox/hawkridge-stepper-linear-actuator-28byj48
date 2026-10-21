# Build Guide: 28BYJ-48 Stepper Linear Actuator with Printed Ball Screw

Remix of the Linear Ball Bearing Stepper Slide v2 by 3DPRINTINGWORLD (Thingiverse thing 2813896, CC BY 4.0). Written from scratch for an Arduino Nano + ULN2003 build with endstop homing. Read the whole guide once before printing; the bearing packing step is the only fiddly part.

## 1. Software first

1. Install the Arduino IDE (2.x). Board: **Arduino Nano**, processor **ATmega328P (Old Bootloader)** for CH340 clones.
2. Install **AccelStepper** from the Library Manager.
3. The 28BYJ-48 is a unipolar 5 V stepper with a 1:64 gearbox; in half-step mode it needs 4096 steps per output-shaft revolution (2048 in full-step). The distance per revolution is the lead of the printed screw you chose: upstream states a 1/2 in (12.7 mm) pitch for the twin-lead screw and 1 in (25.4 mm) for the quad-lead screw. Measure your print and set `MM_PER_REV` accordingly.
4. Example sketch (MIT, Hawkridge). ULN2003 IN1-IN4 on D8-D11, endstop between D2 and GND. It homes against the endstop, then moves 20 mm out and back.

```cpp
#include <AccelStepper.h>
const float MM_PER_REV = 12.7;          // twin-lead screw; 25.4 for the quad-lead screw
const float STEPS_PER_REV = 4096;       // 28BYJ-48 half-step
const int ENDSTOP = 2;
AccelStepper m(AccelStepper::HALF4WIRE, 8, 10, 9, 11);   // IN1, IN3, IN2, IN4
long mm(float x) { return (long)(x * STEPS_PER_REV / MM_PER_REV); }
void setup() {
  pinMode(ENDSTOP, INPUT_PULLUP);
  m.setMaxSpeed(900); m.setAcceleration(400);     // 28BYJ-48 stalls above ~1000 half-steps/s
  m.setSpeed(-400);
  while (digitalRead(ENDSTOP) == HIGH) m.runSpeed();   // home towards the motor end
  m.setCurrentPosition(0);
  m.moveTo(mm(20));
}
void loop() {
  if (m.distanceToGo() == 0) m.moveTo(m.currentPosition() == 0 ? mm(20) : 0);
  m.run();
}
```

5. Upload with the driver unpowered, then power the driver and watch the carriage home. If it drives away from the endstop, swap the sign of `setSpeed` or reverse the IN2/IN3 order.

## 2. Printing the parts

Print files (11, one of each unless noted):

- `BS-V2-RAIL.stl`: the rail with the Gothic-arch bearing grooves. Print flat on its base.
- `BS-V2-CARRIAGE.stl`: the solid bearing carriage.
- `BS-V2-CARRIAGE_CAP.stl` and `BS-V2-CARRIAGE_CAP-MIRROR.stl`: the two end caps that return the BBs.
- `BS-V1-SCREW-1_2-DBL.stl`: twin-lead screw, 1/2 in pitch (strong, slow: about 7.5 in/min at 15 rpm).
- `BS-V1-SCREW-1-QUAD.stl`: quad-lead screw, 1 in pitch (fast: about 15 in/min). Print only the screw you want.
- `BS-V1-NUT-1_2.stl` / `BS-V1-NUT-1.stl`: the matching drive nut for each screw.
- `DT-V3-BEARING_FLANGE.stl`, `DT-V3-BEARING_CAP.stl`, `DT-V3-END_CAP.stl`: thrust-bearing flange, its cap and the far end cap.

Settings: PLA, 0.2 mm layers (0.12 mm for the screw and nut helps), 4 walls, 30 % infill, no supports. Print the screws standing up. The bearing grooves and the screw threads are the precision surfaces, so slow the outer wall down to 25 to 30 mm/s and calibrate your extrusion width first. The package contains the current (v2) carriage and end caps; the older v1 carriage and caps are not included.

## 3. Assembly (after the upstream instructions, in our words)

1. Fit the two M3 x 20 preload screws with nuts into the carriage; leave them loose.
2. Bolt one end cap to the carriage with M3 x 10 screws and slide the carriage onto the rail.
3. Fill the open bearing track with BBs: it takes about 34. Put 6 more BBs into the second end cap, move the carriage to the end of the rail and tip the cap onto the carriage without spilling. Bolt it on. If the BBs will not recirculate, remove one BB on each side.
4. Set the preload with the two long screws until the carriage glides with no play and no binding. No lubricant on the linear bearing.
5. Build the thrust bearing: 8 BBs between the flange and the cap, then grease. Press the drive nut into the carriage.
6. Mount the 28BYJ-48 to the flange with M3 x 10 screws, slide the printed screw onto the motor shaft (D-flat) and through the thrust bearing, then fit the far end cap. Grease the screw and nut.
7. Mount the endstop at the motor end so the carriage lever hits it a few millimetres before the mechanical stop.

## 4. Wiring

- ULN2003 board: `IN1`-`IN4` to D8, D9, D10, D11; `-` to Nano `GND`; `+` to the 5 V supply (barrel adapter), not to the Nano 5 V pin.
- Motor: the 5-pin JST plug into the driver board.
- Endstop: common to D2, normally-open to `GND`.
- Share ground between the supply and the Nano.

## 5. Calibration and first run

1. Home, then command 20 mm and measure the actual travel with calipers; correct `MM_PER_REV` by the ratio.
2. Find the speed limit: raise `setMaxSpeed` until the motor stalls, then back off 20 %. The 28BYJ-48 is a 15 rpm motor; it will not go faster, and more torque comes from the twin-lead screw, not from speed.
3. Load test: the author moved 25 lb with the twin screw; expect less until the bearing is well packed and greased.

## 6. Troubleshooting

- Grinding or clicking: BBs jamming at the end cap, remove one per side; or print the caps at a lower speed.
- Carriage rocks: increase preload with the two M3 x 20 screws.
- Motor steps but the carriage does not move: nut not seated in the carriage, or screw slipping on the D-shaft; add a drop of CA glue on the flat.
- Random stalls: supply sag; use the 5 V 2 A adapter, not USB power.

## 7. Safety and limits

Low force, low voltage: nothing hazardous. The printed screw is not a precision ball screw; expect a few tenths of a millimetre of backlash, fine for pushers and sliders, not for CNC.

## Credits

Original design: Linear Ball Bearing Stepper Slide v2 by 3DPRINTINGWORLD (John Mulac), https://www.thingiverse.com/thing:2813896, CC BY 4.0. The build path, endstop homing, guide and Bill of Material are Hawkridge additions; the example sketch is MIT.
