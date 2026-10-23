# TEC Temperature Controller for an Optical-Feedback-Stabilized ECDL

Implementation of a TEC (thermoelectric cooler) temperature controller, with heating and cooling capability, for an optically feedback-stabilized external cavity diode laser (ECDL).

Developed as part of a guided course under **Dr. Pablo Solano**, at the **Laboratory of Atomic and Molecular Physics (LAMP)**.

## Motivation

The wavelength and mode of a laser diode drift with temperature. For the external cavity and the optical feedback to keep the laser frequency stable, the diode temperature must be actively regulated. A TEC (Peltier element) can both cool and heat the diode by reversing the current direction, which makes it a good actuator for a closed-loop temperature controller.

## System overview

The design is split into four stages (see the full schematic in [`schematic.pdf`](schematic.pdf)):

| # | Stage | Description |
|---|-------|-------------|
| 1 | **Regulated ±15 V power supply** | Dual-rail ±15 V supply (+VCC / −VCC) with LED on/off indicators (D1, D2 with 5 kΩ series resistors). |
| 2 | **Error signal generator** | Wheatstone bridge with a 10 kΩ NTC thermistor, a 10 kΩ setpoint resistor (SP) and a 6.5 V bridge supply (LM399 reference). The bridge output is amplified by an LT1920 instrumentation amplifier, producing the error signal `P_out`. A sinusoidal current source (`perturbation`, 100 µA at 15 kHz, in parallel with C5 = 1 µF) is injected at the thermistor node to test the loop response. |
| 3 | **PID system** | Proportional-integral network built around an OP27 op-amp (R9, R11, C2), with an additional R10/C1 network for the derivative part. |
| 4 | **Cooling and heating stage** | AD711 op-amp driving a complementary push-pull source-follower output stage (IRF530 NMOS and a PMOS: IRF9Z24S_L in simulation, IRF5305 in hardware) that delivers bidirectional current to the TEC, enabling both cooling and heating. The output is fed back to the AD711. The TEC is modeled as a resistor in simulation. |

## Repository structure

```
.
├── Schematics/        LTspice schematics
├── simulations/       LTspice simulations
├── schematic.pdf      Overview of the full circuit
├── LICENSE
└── README.md
```

## Simulation

The circuits were simulated in LTspice. Open the files in `simulations/` and run them directly. For example, the power supply stage uses:

```
.tran 0 0.3 0 100u uic
```

### Simulation results

- **Error signal:** the LT1920 gain is G = 1 + 49.4 kΩ / RG. With RG = 10 kΩ, G ≈ 5.94. A 0.4 kΩ imbalance in the thermistor gives an error of ~0.38 V at `P_out`, in agreement with the hand calculation. With the bridge balanced (all resistors at 10 kΩ), `P_out` goes to 0. Leaving RG open gives G = 1.
- **PI stage:** proportional gain R11/R9 = 10, integral gain 1/(R9·C2) = 1000 s⁻¹ (corner frequency ≈ 16 Hz).
- **Loop sign:** the NTC thermistor is on the in− side of the bridge. If its resistance rises (colder), `P_out` becomes negative, the inverting PI output goes positive and the TEC heats. If the temperature rises, the opposite happens and the TEC cools. The TEC wiring polarity must be checked on hardware.
- **Output stage:** with the default LTspice MOSFET models, the output saturated at about ±1.7 V because the generic models cannot deliver enough current. With power MOSFETs (IRF530 and IRF9Z24S_L) the TEC voltage follows the PI output (V_TEC ≈ 0.909 · V_PI, set by the R8/R2 divider).
- **Perturbation:** the ripple observed in V(pi) and V(tec) was measured at 14.97 kHz, matching the injected 15 kHz source, so it is not an oscillation of the output stage.

### Known limitations and pending items

- In simulation the TEC and the thermistor are fixed resistors, so the thermal loop is not closed. With a constant error the integrator ramps to the rail, and the PI output can remain wound up after the startup transient. A thermal plant model is needed to validate the PID tuning in simulation, and an anti-windup or soft start may be needed on hardware.
- The TEC load (a high-value resistor in the simulations) is not realistic. A real TEC is a few ohms and draws amperes, so the current and power dissipation of the MOSFETs must be checked with a realistic load, and heatsinks may be required.
- The simulation uses the IRF9Z24S_L as PMOS, while the hardware uses the IRF5305. The IRF5305 SPICE model should be added to repeat the simulation with the real pair.

## Main components

Datasheets are not included in this repository, to respect the manufacturers' copyright; the table links to the product pages instead. The table distinguishes what is used in the LTspice simulation from what is used in the real hardware (breadboard and PCB).

| Function | Simulation (LTspice) | Hardware (breadboard / PCB) | Product page |
|----------|----------------------|-----------------------------|-----------|
| +15 V supply | Ideal 15 V source (V3) | LM7815 regulator | n/a |
| −15 V supply | Ideal 15 V source (V1) | LM79xx negative regulator | n/a |
| Bridge voltage reference (6.5 V) | Ideal 6.5 V source (labelled LM399) | LM399 | [LM399](https://www.analog.com/en/products/lm399.html) |
| Temperature sensor | 10 kΩ resistor (modeled as an NTC) | NTCLE100E3 10 kΩ NTC thermistor (Vishay) | [NTCLE100E3](https://www.vishay.com/docs/29049/ntcle100.pdf) |
| Error signal amplifier | LT1920 | LT1920 | [LT1920](https://www.analog.com/en/products/lt1920.html) |
| PID stage | OP27 | OP27 | [OP27](https://www.analog.com/op27) |
| TEC driver op-amp | AD711 | To be confirmed | [AD711](https://www.analog.com/en/products/ad711.html) |
| NMOS output transistor | IRF530 | IRF530 | [IRF530](https://www.vishay.com/en/product/91019/) |
| PMOS output transistor | IRF9Z24S_L (substitute, available in the LTspice library) | IRF5305 | [IRF5305](https://www.infineon.com/part/IRF5305) |
| TEC | Fixed resistor | Real TEC (a few ohms, amperes) | n/a |
| Power indicators | LEDs with 5 kΩ resistors | LEDs with 5 kΩ resistors | n/a |

The IRF5305 is not in the default LTspice library. To simulate the hardware pair exactly, its SPICE model can be added to the project and assigned to M2.

## Status

Work in progress. Schematics and simulations are available.

### Development plan

1. **Simulation (done, with pending items above):** the error signal, PI and output stages were simulated in LTspice.
2. **Breadboard testing (current stage):** the circuit is being built on a breadboard and tested with an oscilloscope, verifying each stage (power supply, error signal generator, PID and TEC drive) and tuning the PID.
3. **Final prototype:** once the circuit is validated, it will be soldered onto a board, mounted in an enclosure and wired to the TEC, the thermistor and the power input.

## Report

A report will be written documenting the whole process: the simulation, the breadboard tests and the final prototype, including the design decisions, measurements and results of each stage.

## Author

Agustín Iván Cáceres Reyes, guided course under Dr. Pablo Solano, LAMP.

PCB design (not included in this repository): Rodrigo Andrés Jofré Cerda, Department of Physics, Universidad de Concepción.

## Acknowledgments

This project was carried out under the guidance of **Rodrigo Andrés Jofré Cerda**, scientist at the Department of Physics, Universidad de Concepción, who designed the PCB and taught me the design, building and testing process, and whose instructions I follow throughout the work. Many thanks for his time and mentorship.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

The PCB design used for the final prototype is not part of this repository.
