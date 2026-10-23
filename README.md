# TEC Temperature Controller for an Optical-Feedback-Stabilized ECDL

Implementation of a TEC (thermoelectric cooler) temperature controller, with heating and cooling capability, for an optically feedback-stabilized external cavity diode laser (ECDL).

Developed as part of a guided course under **Dr. Pablo Solano**, at the **Laboratory of Atomic and Molecular Physics (LAMP)**.

## Motivation

The wavelength and mode of a laser diode drift with temperature. For the external cavity and the optical feedback to keep the laser frequency stable, the diode temperature must be actively regulated. A TEC (Peltier element) can both cool and heat the diode by reversing the current direction, which makes it a good actuator for a closed-loop temperature controller.

## System overview

The design is split into four stages (see `schematic.png`):

!\[Schematic](schematic.png)

|#|Stage|Description|
|-|-|-|
|1|**Regulated ±15 V power supply**|Dual-rail ±15 V supply (+VCC / −VCC) with LED on/off indicators (D1, D2 with 5 kΩ series resistors).|
|2|**Error signal generator**|Wheatstone bridge with a 10 kΩ NTC thermistor, a 10 kΩ setpoint resistor (SP) and a 6.5 V bridge supply (LM399 reference). The bridge output is amplified by an LT1920 instrumentation amplifier, producing the error signal `P\_out`. A sinusoidal current source (`perturbation`, 100 µA at 15 kHz, in parallel with C5 = 1 µF) is injected at the thermistor node to test the loop response.|
|3|**PID system**|Proportional-integral network built around an OP27 op-amp (R9, R11, C2), with an additional R10/C1 network for the derivative part.|
|4|**Cooling and heating stage**|AD711 op-amp driving a complementary push-pull source-follower output stage (IRF530 NMOS / IRF9Z24S\_L PMOS) that delivers bidirectional current to the TEC, enabling both cooling and heating. The output is fed back to the AD711. The TEC is modeled as a resistor in simulation.|

## Repository structure

```
.
├── DATASHEET/         Datasheets of the components used
├── Schematics PCB/    Schematics and PCB design files
├── simulations/       LTspice simulations
├── schematic.png      Overview of the full circuit
├── LICENSE
└── README.md
```

## Simulation

The circuits were simulated in LTspice. Open the files in `simulations/` and run them directly. For example, the power supply stage uses:

```
.tran 0 0.3 0 100u uic
```

### Simulation results

* **Error signal:** the LT1920 gain is G = 1 + 49.4 kΩ / RG. With RG = 10 kΩ, G ≈ 5.94. A 0.4 kΩ imbalance in the thermistor gives an error of \~0.38 V at `P\_out`, in agreement with the hand calculation. With the bridge balanced (all resistors at 10 kΩ), `P\_out` goes to 0. Leaving RG open gives G = 1.
* **PI stage:** proportional gain R11/R9 = 10, integral gain 1/(R9·C2) = 1000 s⁻¹ (corner frequency ≈ 16 Hz).
* **Loop sign:** the NTC is on the in− side of the bridge. If the thermistor resistance rises (colder), `P\_out` becomes negative, the inverting PI output goes positive and the TEC heats. If the temperature rises, the opposite happens and the TEC cools. The TEC wiring polarity must be checked on hardware.
* **Output stage:** with the default LTspice MOSFET models, the output saturated at about ±1.7 V because the generic models cannot deliver enough current. With power MOSFETs (IRF530 and IRF9Z24S\_L) the TEC voltage follows the PI output (V\_TEC ≈ 0.909 · V\_PI, set by the R8/R2 divider).
* **Perturbation:** the ripple observed in V(pi) and V(tec) was measured at 14.97 kHz, matching the injected 15 kHz source, so it is not an oscillation of the output stage.

### Known limitations and pending items

* In simulation the TEC and the thermistor are fixed resistors, so the thermal loop is not closed. With a constant error the integrator ramps to the rail, and the PI output can remain wound up after the startup transient. A thermal plant model is needed to validate the PID tuning in simulation, and an anti-windup or soft start may be needed on hardware.
* The TEC load (a high-value resistor in the simulations) is not realistic. A real TEC is a few ohms and draws amperes, so the current and power dissipation of the MOSFETs must be checked with a realistic load, and heatsinks may be required.
* The MOSFET pair used in the PCB design must be confirmed, and its SPICE models used in the simulation.

## Main components

* LT1920: instrumentation amplifier (error signal)
* LM399: voltage reference (bridge supply)
* NTC 10 kΩ: temperature sensor
* OP27: PID stage
* AD711: output driver
* IRF530 (NMOS) / IRF9Z24S\_L (PMOS): bidirectional TEC drive (simulation models)

## Status

Work in progress. Schematics and simulations are available.

### Development plan

1. **Simulation (done, with pending items above):** the error signal, PI and output stages were simulated in LTspice.
2. **Breadboard testing (current stage):** the circuit is being built on a breadboard and tested with an oscilloscope, verifying each stage (power supply, error signal generator, PID and TEC drive) and tuning the PID.
3. **Final prototype:** once the circuit is validated, it will be soldered onto a board, mounted in an enclosure and wired to the TEC, the thermistor and the power input.

## Report

A report will be written documenting the whole process: the simulation, the breadboard tests and the final prototype, including the design decisions, measurements and results of each stage.

## Author

Agustín Ivan Cáceres Reyes, guided course under Dr. Pablo Solano, LAMP.

PCB design: Rodrigo Andrés Jofré Cerda, Department of Physics, Universidad de Concepción.

## Acknowledgments

This project was carried out under the guidance of **Rodrigo Andrés Jofré Cerda**, scientist at the Department of Physics, Universidad de Concepción, who designed the PCB and taught me the design, building and testing process, and whose instructions I follow throughout the work. Many thanks for his time and mentorship.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

