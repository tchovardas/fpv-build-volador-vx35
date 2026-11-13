# 3.5" FPV Quad Build – FlyFishRC Volador VX3.5

My first self-built FPV quadcopter: a 3.5" analog freestyle build on 4S, assembled and soldered from individual parts.

![Finished build](images/28-final-hero.jpg)

## Specs

| | |
|---|---|
| Frame | FlyFishRC Volador VX3.5 (3.5", deadcat-style) |
| Battery | 4S 750 mAh LiPo, XT30 |
| Video | Analog (Caddx Ratel 2 → SpeedyBee TX800 → Skyzone Cobra X V4) |
| Radio link | ExpressLRS 2.4 GHz (RadioMaster Pocket → RP1) |
| Firmware | Betaflight, DShot300, bidirectional DShot |
| Dry weight | _TODO_ g |
| All-up weight (with battery) | _TODO_ g |

## Parts List

### Quad
| Part | Component | Price |
|---|---|---|
| Frame | FlyFishRC Volador VX3.5 | €36.00 |
| Motors | iFlight XING2 1404 3800KV (×4) | €72.00 |
| FC/ESC Stack | SpeedyBee F405 Mini | €65.00 |
| Camera | Caddx Ratel 2 | €33.95 |
| VTX | SpeedyBee TX800 | €36.95 |
| Receiver | RadioMaster RP1 ELRS Nano | €21.95 |
| Props | Gemfan 3520 (4 sets) | €10.00 |
| **Subtotal** | | **€275.85** |

### Ground Gear
| Part | Component | Price |
|---|---|---|
| Goggles | Skyzone Cobra X V4 | €270.00 |
| Antennas | Foxeer Lollipop V4 LHCP, 2-pack (one on the quad, one on the goggles) | €19.95 |
| Patch antenna | Foxeer Echo 2 Patch LHCP | €19.95 |
| Radio | RadioMaster Pocket (ELRS) | €75.00 |
| Charger | HOTA D6 Pro | €100.00 |
| 18650 cell | 3400 mAh | €4.95 |
| **Subtotal** | | **€489.85** |

### Batteries & Safety
| Part | Component | Price |
|---|---|---|
| Batteries | Tattu R-Line 4S 750 mAh 95C XT30 (×4) | €75.80 |
| LiPo bag | Emax LiPo Safe Bag | €9.95 |
| Smoke stopper | JHEMCU, 30 V / 0.5 A | ~€5.00 |
| **Subtotal** | | **€90.75** |

**Total: €856.45**

## Build Log

### 1. Frame
The kit comes with carbon plates, 3D-printed TPU parts, CNC camera mounts, standoffs and labeled screw bags.

| | |
|---|---|
| ![](images/02-frame-kit-layout.jpg) | ![](images/03-frame-assembled.jpg) |

### 2. Motors
XING2 1404 3800KV motors mounted to the arms, with the motor wires routed along the arms and secured with zip ties.

| | |
|---|---|
| ![](images/04-motor-xing2-1404.jpg) | ![](images/05-motors-mounted.jpg) |

### 3. Receiver
RadioMaster RP1 (ELRS) soldered to a UART on the FC (5V, GND, TX, RX; TX and RX crossed over).

| | |
|---|---|
| ![](images/07-rx-rp1-before.jpg) | ![](images/08-rx-rp1-soldered.jpg) |

### 4. Stack, VTX and Camera
SpeedyBee F405 Mini stack with the motor wires soldered directly to the ESC pads. A low-ESR capacitor sits on the battery leads to filter voltage spikes. The TX800 VTX is powered from the 9 V pad, video goes in on the VTX pad and SmartAudio on a UART TX.

| | |
|---|---|
| ![](images/11-esc-motor-pads-a.jpg) | ![](images/12-esc-motor-pads-b.jpg) |
| ![](images/13-stack-wiring-top.jpg) | ![](images/18-open-frame-c.jpg) |

Wiring reference used:

![Wiring diagram](images/wiring-diagram.png)

_Diagram from the community, not my own drawing._

### 5. Finished

| | |
|---|---|
| ![](images/26-final-no-props.jpg) | ![](images/27-final-top.jpg) |
| ![](images/20-side-profile.jpg) | ![](images/25-goggles-antennas.jpg) |

## What Went Wrong: Smoke Stopper Triggering

Before the first battery plug-in I used a smoke stopper. It lit up red, which normally means a short.

What I did:
1. Inspected every joint, and resoldered the VTX connections to be sure.
2. Checked for shorts between battery +/− and between the 5 V / 9 V rails and GND with a multimeter. No short.
3. Noticed that my smoke stopper trips at only **0.5 A**.

![Smoke stopper](images/21-smoke-stopper.jpg)

Why it most likely false-triggered: when the battery is plugged in, the input capacitors of the 4-in-1 ESC and the extra low-ESR capacitor charge up almost instantly. That inrush current is

$$i(t) = C \, \frac{\mathrm{d}u}{\mathrm{d}t}$$

and with several hundred µF charged to ~16 V within microseconds, the peak current is far above 0.5 A. On top of that, the FC, camera and VTX draw a steady current at idle. A 0.5 A trip threshold is simply too low for a modern 4S build.

Since the multimeter showed no short, I plugged in the battery directly. It powered up without any problems.

**Lesson learned:** a smoke stopper is only useful if its trip current fits the build. Next time I'll use one with a higher or adjustable threshold (or a current-limited bench supply) and always do the multimeter short check first.

![Pad close-up](images/14-stack-pads-closeup.jpg)

## Setup & Configuration

### Stack mounted backwards → board alignment
To make all components fit correctly in the frame, I had to mount the FC/ESC stack rotated by 180° (arrow pointing backwards). The flight controller then sees every movement mirrored, so I compensated in Betaflight under **Configuration → Board and Sensor Alignment** by setting the **yaw alignment to 180°**. After that, the 3D model in the Setup tab moved correctly with the quad.

### Receiver binding
The RadioMaster RP1 and the RadioMaster Pocket are both ExpressLRS, so I bound them with a shared **binding phrase** instead of the button-press bind procedure. Both devices derive the same link ID from the phrase and connect automatically after power-up.

### VTX: the hardest part
Getting the SpeedyBee TX800 to work properly took the most time:
1. **Unlocking the VTX** so that all channels and power levels become available.
2. **Finding a VTX table** for the TX800 online. Betaflight needs it to know which bands, channels and power levels the VTX supports.
3. **Loading the table in the Betaflight GUI didn't work.** The table wouldn't save.
4. **Solution: the CLI.** I pasted the `vtxtable` commands directly into the Betaflight CLI and finished with `save`. After the reboot, the table was stored and I could control band, channel and power from the goggles/OSD.

### Betaflight settings

- Mixer: Quad X, motor direction not reversed (Betaflight default)
- Board alignment: yaw 180°
- ESC protocol: DShot300, bidirectional DShot enabled
- Motor poles: 14
- Motor idle: 5.5 %

![Betaflight motors tab](images/23-betaflight-motors-tab.jpg)

The full configuration is in [`config/betaflight-diff-all.txt`](config/betaflight-diff-all.txt).

## Tools Used

- Soldering iron, Stannol Flux X32-10i flux pen
- Multimeter
- Smoke stopper
- HOTA D6 Pro charger (4S LiPo, charged at 1.0 A ≈ 1.3C)
- Needle-nose pliers, hex drivers, zip ties

## Next Steps

- [ ] Fill in the weights
- [ ] Add flight footage
- [ ] Blackbox logging and PID / filter tuning analysis
