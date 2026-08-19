# Drone_PDB_Aethrone

---

Candidate : Anurag Patankar

Personal Email : patankaranurag23@gmail.com

---

### Problem Statement :
Drone Power Distribution Board (PDB)

---
### Objective : 
Design a Power Distribution Board (PDB) for a medium-sized multirotor UAV capable of safely distributing power from a 6S LiPo battery to the avionics and propulsion systems.

## System Specifications

Input source - 6S LiPo battery, 18 V – 25.2 V (3.0–4.2 V/cell)
Continuous current - 80 A
Peak current - 120 A
Regulated outputs -	5 V @ 3 A, 12 V @ 2 A
Unregulated output  -	Battery voltage direct to ESC1–ESC4
PCB	2-layer, 3 oz copper (both outer layers)
Target thermal design margin - 20 °C rise above ambient
Input Connectors - XT60 (6S LiPo datasheet suggests XT90 for 80A continuous current, but the problem statement mentioned this, so I have used XT60 instead)
Output Connectors (Non-ESC) - JST
Output Connectors (ESC) - Solder holes

Note: Copper sizing is assumed to be at 3oz as it was not mentioned which size is to be used in the problem statement.

---

### Input Protection

1. Reverse Polarity Protection — U1 (LM74700) + Q1 (IPT015N10N5)

Rather than a series diode (which would waste power as heat at 80 A) , meaning 24–40 W burned continuously), the board uses an ideal diode controller driving an N-channel MOSFET, which behaves like a near-lossless diode.

U1 – TI LM74700-Q1: an ideal-diode controller rated for a 3.2 V–65 V supply range, covering the 18–25.2 V input. It drives an external NMOS and regulates its forward voltage drop to ~20 mV, versus the 300–500 mV of a real diode. It also blocks reverse current if the battery is momentarily disconnected/reconnected backward or during a fault, and responds in under 1 µs.
Q1 – Infineon IPT015N10N5: a 100 V, 300 A-rated N-channel MOSFET with a maximum Rds(on) of 1.5 mΩ. At 80 A continuous, conduction loss is I²R = 80² × 0.0015 ≈ 9.6 W; at 120 A peak, ≈21.6 W. The 300 A / 100 V rating gives sufficient margin over the operating envelope of 80 A/25.2 V. This MOSFET choice is what makes a lossless reverse polarity protection stage feasible at this current level, since a lower-current MOSFET would need paralleling.

---
2. TVS Diode

A unidirectional TVS Diode is placed across the protected rail to protect against voltage spikes. When the MOSFET turns off, it will create high voltage spikes (V = L. di/dt). To suppress this,  we use a bulk capacitor(s). E = $\frac{1}{2} \cdot L \cdot I$ and since the energy needs to be stored in the capacitor as well, $E = \frac{1}{2} \cdot C \cdot (V_{max}^2$ - V_{nominal}^2$. After some calculations, we get about C = 1666uF. To minimize the ESR, we place these in parallel instead of 1 big capacitor. Hence 680u * 3 = 2040uF.  

---
3. Fuse

Since we have to handle 120A (peak), the fuse needs to be a MIDI/MEGA style fuse. Since KiCAD doesn't have these footprints readily available, I created it myself with through-holes so that it can be soldered later on.

---
4. TPS5430, TPS5420 buck converters

We needed 5V@3A and 12V@2A, thus these buck converters were chosen after going through the datasheet. TPS5430 can provide a constant current of about 3A, while the TPS5420 can provide a constant current of 2A. 

Feedback Resistors calculation (TPS5430):
Vout = Vref × (1 + R2/R3)
5 V = 1.221 V × (1 + 10 kΩ / R3)
R3 = 10 kΩ / (5/1.221 − 1) ≈ 3.24 kΩ

(TPS5420):
12 V = 1.221 V × (1 + 10 kΩ / R5)
R5 = 10 kΩ / (12/1.221 − 1) ≈ 1.13 kΩ

---
5. Trace Width Calculation:

120 A peak (peak) at 3 oz for	20 °C rise (assuming) is about ~7917 mil²	≈ 48.6 mm which is massive. That is why I have used copper pour instead to dissipate heat evenly.
3A constant at 5V => 1 mm width. This is deliberately kept higher to reduce IR drop, and it also keeps output regulation tighter under load. 
2A constant at 12V => 1 mm width. Same reasoning as above. 

The standard used to calculate the trace widths is **IPC-2221**.

---
6. Via sizing
Via sizing basis: 0.8 mm drill / 1.2 mm pad, ≈4.8 A per via at 20 °C rise (1 oz barrel plating assumed). Thus for each ESC output, TVS diode GND, Bulk capacitor GND, 5 vias are placed and about 20 for Battery input ground pin. Default via sizes are used for all other components.

