# UPS 2kW Line-Interactive Pure Sinewave
## Project Roadmap & Execution Plan

**Project:** Line-Interactive Pure Sinewave UPS, 2kVA / 2kW  
**Topology:** Full-Bridge Inverter, 48V Battery Bank (4x 12V SLA), 220Vac Output  
**MCU Platform:** STM32F303RE (Nucleo-F303RE)  
**Design Tools:** Altium Designer 26.3 (PCB), Cadence OrCAD Capture CIS 25.1 (Schematic/Simulation)  
**Target Market:** Iran Domestic (component availability constraint)  
**Document Version:** 1.0  
**Date:** September 2026

---

## Philosophy

This project follows a **project-based learning (PBL)** approach. Every design decision is documented, every mistake is logged, and every milestone is celebrated. The senior engineer (mentor) guides; the junior engineer (you) executes, learns, and grows.

> "The best way to learn hardware design is to build something that could actually catch fire — and then make sure it doesn't."

---

## Phase Overview

| Phase | Name | Duration | Deliverables |
|-------|------|----------|--------------|
| 0 | Foundation & Learning | 2 weeks | Tool proficiency, component survey |
| 1 | System Architecture & FSM | 3 weeks | Complete FSM, block diagram, specs |
| 2 | Firmware Core (STM32) | 4 weeks | SPWM, ADC, control loops on Nucleo |
| 3 | Power Stage Design | 5 weeks | Schematic, simulation, BOM, PCB layout |
| 4 | Control & BMS Board | 3 weeks | Schematic, PCB, sensor interfaces |
| 5 | User Panel Board | 2 weeks | LCD, buttons, buzzer, enclosure plan |
| 6 | Integration & Testing | 4 weeks | Full system bring-up, debug, tuning |
| 7 | Documentation & Handover | 2 weeks | Final handbook, test reports, user manual |

**Total Estimated Duration:** ~25 weeks (6 months)  
**Buffer:** +4 weeks for unexpected issues  
**Realistic Timeline:** 7 months

---

## Phase 0: Foundation & Learning (Weeks 1–2)

### Goal
Become comfortable with the design tools and understand the component landscape in Iran.

### Week 1: Tool Onboarding

#### OrCAD Capture CIS 25.1
- [ ] Install and configure OrCAD (license setup)
- [ ] Complete "Getting Started" tutorial (build a simple op-amp circuit)
- [ ] Learn to create custom symbols and footprints
- [ ] Practice: Draw the schematic of a simple buck converter (input 12V, output 5V)
- [ ] Run a transient simulation in PSpice for the buck converter
- [ ] Learn to generate BOM from schematic

#### Altium Designer 26.3
- [ ] Install and configure Altium
- [ ] Complete "PCB Design Getting Started" tutorial
- [ ] Learn: schematic capture → PCB layout → Gerber export
- [ ] Practice: Layout the same buck converter from OrCAD (or redraw in Altium)
- [ ] Learn design rules: clearance, trace width, via sizing, copper pour
- [ ] Learn to create component libraries (schematic symbol + PCB footprint)

#### STM32CubeMX + STM32CubeIDE
- [ ] Install STM32CubeMX and STM32CubeIDE
- [ ] Create a project for Nucleo-F303RE
- [ ] Configure: GPIO, TIM (PWM), ADC (DMA), UART, EXTI
- [ ] Generate code and blink an LED
- [ ] Read a potentiometer via ADC and print value over UART

### Week 2: Component Survey & Market Research

#### Power Semiconductors (Available in Iran)
- [ ] Survey MOSFETs: IRFP260N, IRFP4468, IRFP4227, STW45NM60, SPW47N60C3
- [ ] Survey IGBTs: IRG4PC50U, HGTG30N60A4D
- [ ] Gate Drivers: IR2110, IR2113, UCC21520, HCPL-3120 (optocoupler)
- [ ] Document: Vds, Rds(on), Qg, Tj(max), price, availability

#### Passive Components
- [ ] Capacitors: Electrolytic (Nichicon, Rubycon), Film (MKP), Ceramic (X7R)
- [ ] Inductors: Toroidal cores (Micrometals, Amidon), ferrite E-cores
- [ ] Transformers: Custom winding vs. off-the-shelf AVR transformers

#### Sensors & ICs
- [ ] Current sensors: ACS712, ACS758, Shunt + INA219
- [ ] Voltage dividers + Op-Amps: LM358, TL072, LM339 (comparator)
- [ ] Temperature: NTC 10k, LM35, TMP36
- [ ] Relays: 30A/250Vac (Songle, Omron clone)

#### Deliverables
- [ ] **Component Database:** Excel/CSV file with part numbers, specs, prices, suppliers
- [ ] **Design Decision Log:** Why each component family was chosen
- [ ] **Risk Assessment:** Which components are hard to source?

---

## Phase 1: System Architecture & FSM (Weeks 3–5)

### Goal
Define WHAT the system does, WHEN it does it, and HOW the states transition.

### Week 3: Requirements Specification

#### Functional Requirements
- [ ] Output: 220Vac ± 5%, 50Hz ± 0.5%, Pure Sinewave (THD < 5%)
- [ ] Power: 2000W continuous, 3000W surge (for 10 seconds)
- [ ] Transfer Time: < 10ms (Line → Battery)
- [ ] Battery: 48Vdc (4x 12V 7.2Ah SLA), runtime at full load: ~10 minutes
- [ ] Charging: 3-stage (CC → CV → Float), max charge current: 0.1C = 0.72A
- [ ] Protection: Overload, Short Circuit, Overtemp, Battery undervoltage/overvoltage
- [ ] Display: LCD 16x2 or 20x4 (I2C), showing V_in, V_out, I_out, battery %, status
- [ ] Indicators: LEDs for Line, Battery, Fault, Charging
- [ ] Audible: Buzzer for fault/warning/low battery

#### Non-Functional Requirements
- [ ] Efficiency: > 85% at full load
- [ ] Operating Temperature: 0°C to 40°C ambient
- [ ] Cooling: Smart PWM fan (temperature-controlled)
- [ ] Enclosure: IP20, desktop/tower form factor
- [ ] Safety: Isolation between high-voltage and low-voltage sections

#### Interface Requirements
- [ ] AC Input: IEC C14 inlet, 220Vac, 50Hz
- [ ] AC Output: IEC C13 outlet(s), 220Vac
- [ ] Battery: Anderson Powerpole or terminal block, 48V
- [ ] User Interface: LCD, 4 buttons, LEDs, buzzer
- [ ] Debug: UART header (for PC connection)

### Week 4: Finite State Machine (FSM) Design

#### States
1. **INIT** — Power-on self-test
2. **STANDBY** — Idle, monitoring
3. **NORMAL (LINE)** — Mains powering load, charger active
4. **BATTERY (BACKUP)** — Inverter powering load
5. **FAULT** — Error condition, safe state
6. **SHUTDOWN** — Controlled power-off

#### Sub-State Machines
- [ ] **Charging State Machine:** CC → CV → Float (inside NORMAL)
- [ ] **Inverter State Machine:** SOFT_START → STEADY_STATE → SYNC (inside BATTERY)
- [ ] **Fan State Machine:** OFF → LOW → MEDIUM → HIGH (orthogonal, runs in parallel)

#### Transition Table
Document ALL transitions with:
- Source State
- Destination State
- Guard Condition (trigger)
- Action (what happens during transition)
- Priority (which transition wins if multiple guards are true)

#### Deliverables
- [ ] **FSM Diagram** (drawn in draw.io or Visio)
- [ ] **State Transition Table** (Excel or Markdown table)
- [ ] **State Handler Pseudo-Code** for each state

### Week 5: Block Diagram & Interface Definition

#### System Block Diagram
- [ ] Draw complete block diagram with all boards and interfaces
- [ ] Define power flow in each state
- [ ] Define signal flow (analog sensors, digital control, communication)

#### Interface Definition Document
For every inter-board connection, define:
- [ ] Signal name
- [ ] Direction (input/output)
- [ ] Voltage level (3.3V, 5V, 12V, 48V, 220V)
- [ ] Connector type (pin header, terminal block, etc.)
- [ ] Pin assignment

Example:
```
Power Board → Control Board (10-pin header)
Pin 1: VCC_3V3 (out)
Pin 2: GND
Pin 3: V_BUS_SENSE (analog, 0-3.3V scaled from 56V)
Pin 4: I_INV_SENSE (analog, 0-3.3V from ACS712)
Pin 5: ZCD_MAINS (digital, 3.3V pulse)
Pin 6: ZCD_INV (digital, 3.3V pulse)
Pin 7: PWM_H1 (digital, 3.3V, TIM1_CH1)
Pin 8: PWM_L1 (digital, 3.3V, TIM1_CH1N)
Pin 9: PWM_H2 (digital, 3.3V, TIM1_CH2)
Pin 10: PWM_L2 (digital, 3.3V, TIM1_CH2N)
```

#### Deliverables
- [ ] **System Block Diagram** (high resolution, printable)
- [ ] **Interface Definition Document** (all inter-board connections)
- [ ] **Power Budget Table** (current consumption per rail, per board)

---

## Phase 2: Firmware Core — STM32 (Weeks 6–9)

### Goal
Make the Nucleo-F303RE generate SPWM, read sensors, and implement the FSM.

### Week 6: SPWM Generation

#### Theory & Simulation
- [ ] Understand SPWM: carrier (triangle) vs. reference (sine)
- [ ] Calculate: switching frequency (f_sw = 16–20 kHz), modulation index (m)
- [ ] Simulate in Python/MATLAB: generate sine table, plot SPWM waveform
- [ ] Calculate dead-time: 1–2 µs (based on MOSFET turn-on/turn-off delays)

#### STM32 Implementation
- [ ] Configure TIM1 (Advanced Timer) for center-aligned PWM
- [ ] Set ARR (Auto-Reload Register) for desired resolution (e.g., 1023 for 10-bit)
- [ ] Configure TIM1_CH1, CH1N, CH2, CH2N for complementary PWM
- [ ] Enable dead-time insertion (DTG register)
- [ ] Generate sine lookup table (256 or 512 points, 16-bit)
- [ ] Use DMA to update CCR registers from sine table
- [ ] Verify with oscilloscope: PWM waveform, dead-time, frequency

#### Deliverables
- [ ] Python/MATLAB simulation script
- [ ] STM32 project: SPWM generation with variable frequency
- [ ] Oscilloscope screenshots of PWM outputs

### Week 7: ADC & Sensor Reading

#### Voltage Sensing
- [ ] Design voltage divider: 56V → 3.3V (e.g., 100k / 6.8k)
- [ ] Calculate scaling factor and resolution
- [ ] Read DC bus voltage via ADC (DMA, continuous conversion)
- [ ] Read AC output voltage via ADC (sampling at multiple points per cycle)

#### Current Sensing
- [ ] Interface ACS712 (Hall effect) or shunt resistor + op-amp
- [ ] Read inverter output current
- [ ] Read battery charge/discharge current

#### Temperature Sensing
- [ ] Interface NTC 10k thermistor
- [ ] Convert ADC reading to temperature (Steinhart-Hart or lookup table)

#### Zero-Cross Detection
- [ ] Design ZCD circuit: optocoupler (PC817) + comparator
- [ ] Detect zero-cross of mains input
- [ ] Detect zero-cross of inverter output
- [ ] Measure phase difference between mains and inverter

#### Deliverables
- [ ] Sensor interface circuits (schematic snippets)
- [ ] STM32 project: ADC + DMA reading all sensors
- [ ] UART output showing real-time sensor values

### Week 8: Control Loops

#### Open-Loop Test
- [ ] Drive H-Bridge (on breadboard or eval board) with SPWM
- [ ] Measure output with low-voltage DC input (e.g., 12V)
- [ ] Verify sine wave shape with oscilloscope
- [ ] Measure RMS voltage vs. modulation index

#### Closed-Loop Voltage Control
- [ ] Implement PI controller for output voltage
- [ ] Kp, Ki tuning: start conservative, increase gradually
- [ ] Test with resistive load (light bulb)
- [ ] Measure THD with oscilloscope FFT

#### Current Limiting
- [ ] Implement overcurrent protection (hardware + software)
- [ ] Hardware: comparator shuts down PWM immediately
- [ ] Software: reduce modulation index if current exceeds threshold

#### Deliverables
- [ ] PI controller code (with anti-windup)
- [ ] Test results: voltage regulation, THD, load step response
- [ ] Oscilloscope screenshots

### Week 9: FSM Implementation

#### State Machine Code
- [ ] Implement FSM using function-pointer table or switch-case
- [ ] Each state has: entry action, run action, exit action
- [ ] Transition table with guard conditions

#### State Testing
- [ ] Simulate state transitions manually (force inputs)
- [ ] Test: INIT → STANDBY → NORMAL → BATTERY → NORMAL → SHUTDOWN
- [ ] Test fault injection: force overcurrent, overtemperature
- [ ] Verify safe state entry (all PWM off, relays open)

#### Deliverables
- [ ] Complete FSM source code
- [ ] State transition test log (table of tests and results)
- [ ] Debug UART log showing state transitions

---

## Phase 3: Power Stage Design (Weeks 10–14)

### Goal
Design, simulate, and layout the Power Board (Inverter + Charger + AUX DC-DC).

### Week 10: Inverter H-Bridge Design

#### Topology Selection
- [ ] Full-Bridge (H-Bridge) confirmed for 2kW
- [ ] MOSFET selection: IRFP4468 (Vds=100V, Rds=2.0mΩ, Qg=215nC) or similar
- [ ] Calculate conduction losses: P = I² × Rds(on) × 2 (two MOSFETs on at any time)
- [ ] Calculate switching losses: P_sw = 0.5 × V × I × (tr + tf) × f_sw
- [ ] Total loss < 10% of 2kW = 200W → heatsink + fan required

#### Gate Driver Design
- [ ] Select gate driver: IR2110 (bootstrap) or UCC21520 (isolated)
- [ ] Design bootstrap circuit: diode (UF4007) + capacitor
- [ ] Calculate bootstrap capacitor: C_boot = 2 × (2Qg + Qls + Iqbs/f) / (VCC - Vf - Vls)
- [ ] Layout: gate resistor (10–22Ω), gate-source pulldown (10kΩ), gate snubber

#### Output Filter Design
- [ ] LC filter: L = 1–2 mH, C = 2–4 µF (MKP film capacitor)
- [ ] Cutoff frequency: fc = 1 / (2π√(LC)) ≈ 3–5 kHz
- [ ] Attenuation at 16 kHz: > 40 dB
- [ ] Inductor design: toroidal core, calculate turns, wire gauge, saturation check

#### Deliverables
- [ ] Inverter schematic (OrCAD)
- [ ] Loss calculation spreadsheet
- [ ] Inductor design calculations
- [ ] Simulation: PSpice/LTspice of H-Bridge + filter

### Week 11: Charger (PFC + Battery Charger)

#### Topology
- [ ] Input rectifier: bridge rectifier + bulk capacitor
- [ ] PFC stage: Boost PFC (optional for first prototype — can skip for simplicity)
- [ ] Charger: Buck converter 220V → 56V (or Flyback for isolation)
- [ ] CC/CV control: current sense + voltage feedback

#### Component Selection
- [ ] Rectifier bridge: KBPC5010 (50A, 1000V)
- [ ] Bulk capacitor: 2× 470µF / 400V (electrolytic, 105°C)
- [ ] Charger MOSFET: based on topology
- [ ] Charger controller: TL494 (analog) or STM32 (digital)

#### Deliverables
- [ ] Charger schematic
- [ ] Charging profile: CC → CV → Float (voltage/current vs. time)
- [ ] Simulation of charging behavior

### Week 12: AUX DC-DC Power Supplies

#### Rails Required
| Rail | Voltage | Current | Purpose |
|------|---------|---------|---------|
| VCC_GD | 12–15V | 1A | Gate drivers |
| VCC_FAN | 12V | 0.5A | Cooling fan |
| VCC_RELAY | 12V | 0.5A | Relays |
| VCC_5V | 5V | 0.5A | Sensors, LCD |
| VCC_3V3 | 3.3V | 0.3A | STM32, logic |

#### Topology
- [ ] Flyback converter: 56V → 12V/15V (primary rail)
- [ ] Buck converters: 12V → 5V, 12V → 3.3V
- [ ] Or: Multiple flyback secondaries

#### IC Selection
- [ ] Flyback controller: UC3844, TOPSwitch, or VIPer
- [ ] Buck converters: LM2596, LM7805 (linear for low current), AMS1117-3.3

#### Deliverables
- [ ] AUX power schematic
- [ ] Power budget verification (all rails within spec)
- [ ] Transformer design (if flyback)

### Week 13: Power Board Schematic Integration & Simulation

#### Integration
- [ ] Combine: Inverter + Charger + AUX DC-DC into one schematic
- [ ] Add protection: fuses, TVS diodes, varistors, snubbers
- [ ] Add connectors: AC input, AC output, battery, control header
- [ ] Add test points (TP) for debugging

#### Simulation
- [ ] Simulate inverter startup (soft-start behavior)
- [ ] Simulate load step (0 → 100% → 50%)
- [ ] Simulate short-circuit response
- [ ] Verify no shoot-through in H-Bridge

#### Design Review
- [ ] Check: trace width for high current (use online calculator)
- [ ] Check: clearance for high voltage (≥ 3mm for 220Vac)
- [ ] Check: thermal design (heatsink + fan adequacy)
- [ ] Check: safety isolation distances

#### Deliverables
- [ ] Complete Power Board schematic (OrCAD)
- [ ] Simulation results (plots, screenshots)
- [ ] Design Review Checklist (signed off by mentor)

### Week 14: Power Board PCB Layout

#### Layout Strategy
- [ ] Separate: high-voltage section (top) from low-voltage section (bottom)
- [ ] Power path: short, wide traces for high current
- [ ] Gate drive: short traces, minimize loop area
- [ ] Grounding: single-point connection between power GND and signal GND
- [ ] Thermal: heatsink mounting holes, thermal vias under MOSFETs

#### Layer Stack (4-layer recommended)
| Layer | Purpose |
|-------|---------|
| Top | Signals + Components |
| Inner 1 | Power planes (+V, -V) |
| Inner 2 | Ground plane |
| Bottom | Signals + Components |

#### Placement
- [ ] Place MOSFETs near heatsink
- [ ] Place bulk capacitors close to MOSFETs
- [ ] Place gate drivers close to MOSFET gates
- [ ] Place connectors on board edge

#### Routing
- [ ] Route high-current paths first (inverter output, battery)
- [ ] Route gate drive signals (short, matched length)
- [ ] Route sensor signals (differential, guarded)
- [ ] Add copper pours for heat dissipation

#### DRC & Output
- [ ] Run Design Rule Check (DRC)
- [ ] Generate Gerber files
- [ ] Generate drill files
- [ ] Generate pick-and-place file
- [ ] Generate assembly drawing

#### Deliverables
- [ ] Power Board PCB files (Altium)
- [ ] Gerber files (ready for fabrication)
- [ ] Assembly drawing
- [ ] 3D render of PCB (for enclosure design)

---

## Phase 4: Control & BMS Board (Weeks 15–17)

### Goal
Design the board that hosts the Nucleo-F303RE, sensor interfaces, relay drivers, and BMS.

### Week 15: Control Board Schematic

#### MCU Section
- [ ] Nucleo-F303RE header pinout (all used pins defined)
- [ ] Decoupling capacitors (100nF + 10µF per power pin)
- [ ] Reset circuit + boot mode selection
- [ ] Debug header: SWD (ST-Link)

#### Sensor Interfaces
- [ ] Voltage dividers for: V_bus, V_battery, V_ac_out
- [ ] Current sense: ACS712 or shunt + op-amp
- [ ] Temperature: NTC 10k with pull-up
- [ ] Zero-cross detectors: optocoupler + comparator

#### Relay Drivers
- [ ] Bypass relay driver: transistor + flyback diode
- [ ] Inverter relay driver: transistor + flyback diode
- [ ] Charger relay/contactor driver (if needed)

#### BMS Circuit
- [ ] Cell voltage monitoring: 4x voltage dividers + op-amp buffers
- [ ] Cell balancing: 4x bypass resistors (47Ω, 2W) with MOSFET switches
- [ ] Overcharge detection: LM339 comparator per cell
- [ ] Overdischarge detection: LM339 comparator per cell
- [ ] Charge/discharge MOSFETs: high-side + low-side switches

#### Communication
- [ ] UART header (for debug)
- [ ] I2C header (for future expansion)
- [ ] SPI header (for future expansion)

#### Deliverables
- [ ] Control Board schematic (OrCAD)
- [ ] Pin assignment table (STM32 pins → functions)
- [ ] BMS circuit calculations

### Week 16: Control Board PCB Layout

#### Layout Considerations
- [ ] Analog section (sensors) far from digital section (MCU)
- [ ] Ground separation: analog GND, digital GND, power GND
- [ ] Single-point ground connection
- [ ] Sensor traces: short, away from switching noise
- [ ] Relay drivers: near connectors, away from sensitive analog

#### Layer Stack (2-layer acceptable for control board)
| Layer | Purpose |
|-------|---------|
| Top | Signals + Components |
| Bottom | Ground plane + Power traces |

#### Deliverables
- [ ] Control Board PCB files (Altium)
- [ ] Gerber files

### Week 17: BMS Firmware

#### Cell Voltage Reading
- [ ] Read 4 cell voltages via ADC (multiplexer or separate channels)
- [ ] Calculate total battery voltage
- [ ] Detect imbalance (ΔV > 0.2V between cells)

#### Balancing Algorithm
- [ ] Passive balancing: turn on bypass resistor for highest cell
- [ ] Balancing during charging (when cell > 13.5V)
- [ ] Stop balancing when ΔV < 0.05V

#### Protection Logic
- [ ] Overcharge: if any cell > 14.8V → stop charging
- [ ] Overdischarge: if any cell < 10.5V → stop discharging
- [ ] Overcurrent: if I > 50A → disconnect battery
- [ ] Overtemp: if T > 50°C → reduce charge current or stop

#### Deliverables
- [ ] BMS firmware module
- [ ] Test results: balancing behavior, protection triggers

---

## Phase 5: User Panel Board (Weeks 18–19)

### Goal
Design the human interface: LCD, buttons, LEDs, buzzer.

### Week 18: User Panel Schematic & PCB

#### LCD Interface
- [ ] LCD 16x2 or 20x4 with I2C backpack (PCF8574)
- [ ] Or parallel interface (4-bit mode) if I2C too slow
- [ ] Contrast potentiometer
- [ ] Backlight control (PWM for brightness)

#### Buttons
- [ ] 4 tactile buttons: UP, DOWN, SELECT, BACK/ESC
- [ ] Debounce: hardware (RC) + software (10ms delay)
- [ ] Pull-up resistors (internal or external)

#### LEDs
- [ ] Line LED (green)
- [ ] Battery LED (yellow/blue)
- [ ] Fault LED (red, blinking)
- [ ] Charging LED (green, blinking when charging)

#### Buzzer
- [ ] Piezo buzzer (active or passive)
- [ ] Drive: transistor + PWM for tone control

#### PCB
- [ ] Small PCB (e.g., 80x60mm)
- [ ] Mounting holes for enclosure
- [ ] Flexible cable (FPC or ribbon) to Control Board

#### Deliverables
- [ ] User Panel schematic & PCB
- [ ] Menu structure document (LCD screens and navigation)

### Week 19: User Interface Firmware

#### Menu Structure
```
Main Screen:
  Input: 220V | Output: 220V
  Load: 45%  | Batt: 85%
  [NORMAL]

Menu:
  ├── Status
  │     ├── Input Voltage
  │     ├── Output Voltage
  │     ├── Output Current
  │     ├── Battery Voltage
  │     ├── Battery Current
  │     ├── Temperature
  │     └── Runtime Estimate
  ├── Settings
  │     ├── LCD Brightness
  │     ├── Buzzer On/Off
  │     ├── Fan Mode (Auto/Manual)
  │     └── Set Clock (if RTC added)
  ├── Logs
  │     ├── Last Fault
  │     ├── Switch Count
  │     └── Total Runtime
  └── Test
        ├── Self Test
        ├── Manual Switch
        └── Buzzer Test
```

#### Button Handling
- [ ] State machine for button presses
- [ ] Short press vs. long press
- [ ] Menu navigation logic

#### Display Update
- [ ] Update main screen every 500ms
- [ ] Update menu on button press
- [ ] Show popup messages (faults, warnings)

#### Deliverables
- [ ] UI firmware module
- [ ] Menu flow diagram
- [ ] Demo video (optional)

---

## Phase 6: Integration & Testing (Weeks 20–23)

### Goal
Bring all boards together, test as a system, debug, and tune.

### Week 20: Board Bring-Up (Individual)

#### Power Board Bring-Up
- [ ] Visual inspection (solder bridges, wrong components)
- [ ] Power-on with current-limited supply (no MOSFETs first)
- [ ] Test AUX DC-DC: verify all rails (3.3V, 5V, 12V, 15V)
- [ ] Install MOSFETs, test gate drive signals (no HV)
- [ ] Apply low DC voltage (12V), verify SPWM output

#### Control Board Bring-Up
- [ ] Power-on, verify 3.3V rail
- [ ] Program STM32 via ST-Link
- [ ] Test UART communication
- [ ] Test sensor readings (apply known voltages)
- [ ] Test relay drivers (click test)

#### User Panel Bring-Up
- [ ] Test LCD (display test pattern)
- [ ] Test buttons (read via STM32)
- [ ] Test LEDs and buzzer

#### Deliverables
- [ ] Bring-up checklist (completed for each board)
- [ ] Issue log (any problems found and fixes)

### Week 21: Subsystem Integration

#### Inverter Test (Low Power)
- [ ] Connect inverter output to resistive load (100W bulb)
- [ ] Apply 12V DC input, verify sine wave output
- [ ] Measure: Vrms, frequency, THD
- [ ] Test overload: increase load to 200W (if possible with 12V)

#### Charger Test
- [ ] Connect to AC input, verify DC bus voltage
- [ ] Connect to battery, verify charging profile
- [ ] Measure charge current in CC, CV, Float modes

#### BMS Test
- [ ] Connect to battery pack
- [ ] Verify cell voltage readings
- [ ] Trigger overcharge (use power supply), verify protection
- [ ] Trigger overdischarge, verify protection

#### Relay Switching Test
- [ ] Test bypass relay: on/off with load
- [ ] Test inverter relay: on/off
- [ ] Measure switch time with oscilloscope

#### Deliverables
- [ ] Subsystem test reports (voltage, current, waveform screenshots)
- [ ] Calibration data (ADC scaling factors)

### Week 22: Full System Integration

#### Connection
- [ ] Connect all boards together
- [ ] Connect battery pack
- [ ] Connect AC input and output
- [ ] Double-check all connections (polarity, voltage levels)

#### Power-On Sequence
- [ ] Power on with no load
- [ ] Verify INIT → STANDBY → NORMAL transition
- [ ] Verify LCD shows correct status
- [ ] Verify charger starts (battery charging)

#### Load Test
- [ ] Apply 500W load (resistive)
- [ ] Measure: efficiency, output voltage stability, temperature
- [ ] Apply 1000W load
- [ ] Apply 1500W load
- [ ] Apply 2000W load (full power)

#### Transfer Test
- [ ] Disconnect AC input (simulate blackout)
- [ ] Measure transfer time (oscilloscope on output)
- [ ] Verify seamless transition (or < 10ms gap)
- [ ] Reconnect AC input
- [ ] Verify return to NORMAL mode

#### Fault Test
- [ ] Overload: apply > 2200W, verify shutdown
- [ ] Short circuit: momentary short on output, verify protection
- [ ] Overtemp: block fan, verify temp rise and shutdown
- [ ] Battery undervoltage: discharge battery, verify low-battery warning and shutdown

#### Deliverables
- [ ] Full system test report
- [ ] Oscilloscope screenshots (transfer time, waveforms)
- [ ] Thermal images (if available)

### Week 23: Tuning & Optimization

#### Control Loop Tuning
- [ ] Fine-tune PI controller (Kp, Ki) for best response
- [ ] Test load step response (0 → 100%, 100% → 0)
- [ ] Minimize overshoot and settling time

#### THD Optimization
- [ ] Adjust SPWM parameters (carrier frequency, modulation index)
- [ ] Test different loads: resistive, inductive (motor), capacitive
- [ ] Target: THD < 5% at full load

#### Efficiency Optimization
- [ ] Measure efficiency at 25%, 50%, 75%, 100% load
- [ ] Identify loss sources (conduction, switching, magnetic)
- [ ] Optimize if possible (better MOSFETs, lower Rds(on))

#### Thermal Optimization
- [ ] Measure temperatures at full load
- [ ] Adjust fan curve if needed
- [ ] Verify heatsink adequacy

#### Deliverables
- [ ] Tuning log (parameter changes and results)
- [ ] Final performance summary (efficiency, THD, transfer time)

---

## Phase 7: Documentation & Handover (Weeks 24–25)

### Goal
Create a professional engineering document package.

### Week 24: Engineering Design Handbook

#### Contents
1. **Executive Summary** — What, why, how, results
2. **System Architecture** — Block diagram, FSM, requirements
3. **Power Stage Design** — Inverter, charger, AUX DC-DC (theory, calculations, schematics)
4. **Control System Design** — MCU, sensors, BMS, firmware architecture
5. **User Interface Design** — LCD menu, buttons, indicators
6. **PCB Design** — Layout considerations, layer stack, DRC
7. **Firmware Documentation** — Code structure, modules, APIs
8. **Test Results** — All test data, graphs, screenshots
9. **Bill of Materials** — Complete BOM with part numbers, suppliers, costs
10. **Assembly Instructions** — Step-by-step build guide
11. **Troubleshooting Guide** — Common problems and solutions
12. **Future Improvements** — What could be better next time

#### Deliverables
- [ ] Complete Engineering Design Handbook (PDF, printable)
- [ ] All source files (OrCAD, Altium, STM32CubeIDE)
- [ ] Gerber files and fabrication notes

### Week 25: Final Reports & Presentation

#### Final Test Report
- [ ] Summary of all tests passed/failed
- [ ] Performance metrics vs. requirements
- [ ] Certification readiness (if applicable)

#### User Manual
- [ ] Quick start guide
- [ ] Safety warnings
- [ ] Operating instructions
- [ ] Maintenance schedule

#### Presentation
- [ ] Slide deck: problem → solution → design → results → demo
- [ ] 10-minute presentation + 5-minute Q&A
- [ ] Demo video (optional but impressive)

#### Deliverables
- [ ] Final Test Report
- [ ] User Manual
- [ ] Presentation slides
- [ ] Demo video (optional)

---

## Weekly Rituals

### Every Monday: Planning Meeting (30 min)
- Review last week's progress
- Define this week's goals
- Identify blockers
- Update Gantt chart / roadmap

### Every Friday: Review & Log (30 min)
- What was accomplished?
- What problems were encountered?
- What was learned?
- Update the Engineering Design Handbook
- Update the Progress Report

### Continuous: Issue Log
- Every bug, every question, every "huh?" moment gets logged
- Format: Date | Issue | Severity | Root Cause | Fix | Verified

---

## Risk Register

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Key component unavailable in Iran | High | High | Maintain 2nd and 3rd source for every critical part |
| PCB fabrication delay | Medium | Medium | Order early; have backup fab house |
| MOSFET failure during test | Medium | High | Current-limited power supply; fuse protection; start at low voltage |
| STM32 code bugs causing shoot-through | Medium | Critical | Hardware dead-time + software interlocks + scope monitoring |
| Thermal runaway | Low | Critical | Temperature monitoring + smart fan + thermal shutdown |
| Scope creep (adding features) | High | Medium | Strict change control; new features go to "Phase 8" |
| Personal burnout | Medium | High | Take weekends off; celebrate milestones; ask for help |

---

## Milestone Celebrations

| Milestone | Reward |
|-----------|--------|
| First SPWM on oscilloscope | Coffee + pastry |
| First sine wave from inverter | Dinner out |
| First successful transfer (Line → Battery) | Movie night |
| Full power 2kW test passed | Weekend off |
| Project complete | Big celebration + start next project! |

---

## Appendix A: Learning Resources

### OrCAD / PSpice
- "Complete PCB Design with OrCAD" — YouTube playlist
- Cadence official documentation

### Altium Designer
- Altium Academy (free online courses)
- "Getting Started with Altium Designer" — official tutorial

### STM32
- STM32CubeMX + HAL library documentation
- "Mastering STM32" book by Carmine Noviello
- YouTube: Controllerstech, STM32duino

### Power Electronics
- "Power Electronics" by Daniel W. Hart
- "Fundamentals of Power Electronics" by Erickson & Dragan
- YouTube: EEVblog, W2AEW, Power Electronics

### Control Systems
- "Feedback Systems" by Astrom & Murray (free online)
- YouTube: Brian Douglas (Control Systems)

---

## Appendix B: Tools & Software Checklist

| Tool | Purpose | Status |
|------|---------|--------|
| OrCAD Capture CIS 25.1 | Schematic capture | ☐ Installed |
| PSpice | Circuit simulation | ☐ Installed |
| Altium Designer 26.3 | PCB layout | ☐ Installed |
| STM32CubeMX | MCU configuration | ☐ Installed |
| STM32CubeIDE | Firmware development | ☐ Installed |
| Python + NumPy/Matplotlib | Simulation & analysis | ☐ Installed |
| Git | Version control | ☐ Installed |
| Oscilloscope (≥ 100 MHz) | Signal measurement | ☐ Available |
| Multimeter | Basic measurements | ☐ Available |
| DC Power Supply (0–60V, ≥ 10A) | Testing | ☐ Available |
| Variable AC Source / Variac | AC testing | ☐ Available |
| Electronic Load (or resistive loads) | Load testing | ☐ Available |

---

*Document maintained by: [Your Name]*  
*Mentor: Senior Circuit Designer*  
*Last Updated: September 2026*
