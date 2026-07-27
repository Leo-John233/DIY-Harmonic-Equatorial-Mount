<div align="center">
  <a href="./README_CN.md"><img src="https://img.shields.io/badge/简体中文-4285F4.svg?style=flat&logo=googletranslate&logoColor=white" alt="简体中文"></a>
  <a href="./README_EN.md"><img src="https://img.shields.io/badge/English-4285F4.svg?style=flat&logo=googletranslate&logoColor=white" alt="English"></a>
</div>

## 📄 License / Open Source Agreement

This project employs a combination of open-source licenses, with separate licensing applied to the software and hardware components:

*   **Software** (e.g., firmware code): Licensed under the [GNU General Public License v3.0 (GPL-3.0)](LICENSE-GPL). For detailed information, please refer to the `LICENSE-GPL` file.
*   **Hardware** (e.g., PCB designs, schematics, mechanical 3D models): Licensed under the [CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S v2)](LICENSE-CERN). For detailed information, please refer to the `LICENSE-CERN` file.

---
## DIY Harmonic Equatorial Mount (Field Imaging and Guiding Verified)

Having grown weary of the weight of my EQ3D equatorial mount, I planned to replace it with a lighter and more portable harmonic mount. Commercial harmonic mounts are expensive, especially when compared with the specifications achievable through a DIY build. This project was therefore created to combine precision, portability, and cost-effectiveness without compromising performance.

This project aims to address several critical issues commonly encountered in amateur astronomy equipment:
- The high cost of commercially manufactured mounts.
- A lack of openness and flexibility in design.
- Excessive complexity or incompatibility with individual equipment setups.

The key advantages of this DIY harmonic equatorial mount include:
- 🧰 **Cost-Effective** — The total build cost is under 3,000 RMB, with the cost of custom-machined parts amounting to less than 1,500 RMB.
- 🪶 **Relatively Portable** — Weighs 7.5 kg excluding the latitude base.
- 🏗️ **Easy to Build** — Designed for makers with basic tools.
- 💪 **High Performance** — PHD2 guide satellite imaging test completed.
- ⚠️ **Note on Current Project Status** ⚠️
The project has completed power-on, basic GoTo, guiding, and real-world imaging verification with an actual imaging payload of approximately 13kg. The total RMS of PHD2 during the stable guiding period is typically around 0.4″ to 0.6″, although results may vary depending on the assembly, payload, and environment.

## ✨ Key Features
| **Feature** | **Description** |
| :----- | :----- |
| Mount Type | German Equatorial Mount (GEM) |
| Drive Type | Harmonic Drive + Belt Drive + 42 Stepper Motor |
| Payload Capacity | Design target of up to 30 kg with counterweights and 20–25 kg without counterweights; approximately 13 kg has been verified through field imaging |
| Harmonic Drive Model | Leaderdrive LHS-25-100-C-III |
| Control System | OnStep / OnStepx |
| Connectivity | USB-B; Wi-Fi; Bluetooth; Integrated DS3231SN Real-Time Clock Chip |
| Hardware Features | RA Axis Brake; RA/DEC Axis Photoelectric Limit Sensors; DEC/RA Axis Photoelectric Home Sensors;<br>Simultaneous compatibility with both Step/Dir and SPI driver modes (i.e., compatible with TMC2209 and TMC5160 drivers); 12V/24V Input |
| Estimated Build Cost | 2500–2900 RMB |
| Estimated Weight | 9.5 kg (including dovetail saddle and latitude base) |
| Measured Guiding Performance | PHD2 total RMS typically around 0.4″ to 0.6″ during stable guiding |

---

## Final Results

The core components of this DIY harmonic equatorial mount are used industrial Leaderdrive harmonic reducers. The design is also compatible with equivalent 25-100 harmonic reducers.
![image.png](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/3D-Views/1.png)
![image.jpg](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/3D-Views/3.jpg)
![image.jpg](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/3D-Views/5.JPG)
![image.jpg](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/3D-Views/4.jpg)
![image.jpg](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/3D-Views/2.jpg)

---

## Field Imaging and Guiding Test

The imaging target was **Herschel 36**

![Herschel 36](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/Test/Herschel-36.jpg)

| **Imaging Item** | **Configuration** |
| :----- | :----- |
| Main imaging system | Sky-Watcher BKP 150/750 with an MPCC coma corrector |
| Main camera | Nikon D7000 |
| Guide camera | ZWO ASI662MC |
| Actual imaging payload | Approximately 13 kg |
| Test conditions | Light breeze |

## Current Validation Status

- [√] Controller power-up and basic communication
- [√] OnStep ASCOM connection
- [√] Basic GoTo and sidereal tracking
- [√] PHD2 calibration and closed-loop guiding
- [√] Dither and settling
- [√] Continuous guiding record of approximately 52 min 45 s
- [√] Herschel 36 field imaging with an actual payload of approximately 13 kg
- [√] Multi-night repeat testing with different payloads and center-of-gravity positions
- [√] Systematic comparison between counterweighted and counterweight-free configurations
- [ ] Long unguided intrinsic periodic-error testing with guide corrections and dithering disabled
- [ ] Long-duration imaging validation at different focal lengths, exposure times, and environmental conditions

## PHD2 Guiding Test

### Test Configuration

| Item | Configuration |
| :--- | :--- |
| Test software | PHD2 Guiding 2.6.13 |
| Mount connection | OnStep ASCOM |
| Guide camera | ZWO ASI662MC |
| Guide focal length | 200 mm |
| Guide-camera pixel size | 2.9 μm |
| Guide image scale | Approximately 2.99″/px |
| Guide exposure | 0.2 s |
| Actual imaging payload | Approximately 13 kg |
| Test conditions | Light breeze |
| Complete log duration | 52 min 45 s |

### Results Summary

| Test interval | RA RMS | Dec RMS | Total RMS | Notes |
| :--- | ---: | ---: | ---: | :--- |
| Stable guiding interval | 0.43″ | 0.50″ | 0.66″ | PHD2 real-time statistics screenshot |
| Complete 52 min 45 s log | 0.45″ | 0.82″ | 0.94″ | Includes dither, settling, and a small number of outlier peaks |

During normal use, the total RMS during stable guiding was typically around 0.4″ to 0.6″. The representative stable screenshot shows a total RMS of 0.66″, while the complete log reached 0.94″ because it includes dither, settling, and a small number of outlier peaks

### Results Analysis

- **The two axes were well balanced during stable guiding**: RA RMS was 0.43″ and Dec RMS was 0.50″, with only a small difference between the axes and no clear indication of sustained single-axis oscillation. The total RMS of 0.66″ corresponds to approximately 0.22 guide-camera pixels at 2.99″/px, including approximately 0.14 px in RA and 0.17 px in Dec
- **The higher RMS in the complete log was driven mainly by transient Dec deviations**: The complete-log RA RMS was 0.45″, which is close to the stable interval and indicates broadly consistent long-duration RA behavior. Dec RMS increased to 0.82″, suggesting that dither, settling, the light breeze, or a small number of outlier peaks mainly affected the Dec statistics. It does not mean that Dec continuously tracked at an error of 0.82″ during stable operation
- **Stable guiding was maintained under an actual payload of approximately 13 kg**: With the fully modified Sky-Watcher BKP 150/750, Nikon D7000, MPCC, and guide system installed, the stable interval showed no obvious divergence or sustained mechanical oscillation. This indicates that the present structural rigidity, transmission control, and guiding response are sufficient for practical imaging with this tested setup
- **The 0.2 s guide exposure demonstrates fast correction response**: A short exposure can respond quickly to tracking deviations, but it is also more sensitive to seeing and brief disturbances. The observed 0.4″ to 0.6″ range should therefore be treated as a measured result for this specific equipment, parameter set, and environment rather than a fixed value for every operating condition
- **The frequency-domain result is not a direct measurement of intrinsic periodic error**: The selected low-frequency component had a period of approximately 2206.6 s and a displayed amplitude of approximately 0.1″. Because the data came from a closed-loop guiding log containing low-frequency drift, dither, and settling, the result can only be used to inspect frequency components and cannot be identified directly as the intrinsic periodic error of the harmonic reducer

<details>
<summary>View the PHD2 screenshots and frequency-domain analysis</summary>

![PHD2 stable guiding](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/Test/2026-07-03%20002555.png)

![PHD2 complete log](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/Test/2026-07-03%20133401.png)

![PHD2 drift-corrected analysis](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/Test/2026-07-03%20153632.png)

![PHD2 frequency analysis](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/Test/2026-07-03%20153353.png)

The curve with RA guide corrections removed still contains low-frequency drift, dither, and settling effects. Its full peak-to-peak range must not be treated as the intrinsic periodic error of the harmonic reducer

In the frequency analysis, the selected low-frequency component has a period of approximately 2206.6 s and a displayed amplitude of approximately 0.1″. This result is useful only for inspecting frequency components and does not replace a long unguided periodic-error test with guide corrections and dithering disabled

</details>

---

## 🔗 Bill of Materials (BOM)

Please refer to [BOM.md](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/tree/main/BOM)

**Estimated Total Cost: Approx. 2500 RMB (including latitude base)**

The main structural components of this project rely heavily on custom fabrication. Key components include two size 25-100 harmonic drives, which provide an exceptionally high gear ratio and torque output. The base interface can be adapted to fit most mainstream tripods, allowing for customization based on individual preference.

*Note: Prices are approximate and may vary depending on the motor supplier, CNC machining batch, and geographic location.*

---

## 🔌Electronics Overview

The control system features a completely custom design, built around a compact architecture, and includes:

*   **An ESP32-WROOM-32UE16N-based main control core**, running the powerful [OnStep/OnStepX](https://onstep.groups.io/g/main/wiki) system.
*   **Integrated ESP07S wireless connectivity**, providing efficient wireless and wired communication capabilities.
*   **TMC-series stepper motor drivers**, enabling precise, silent microstepping control and pointing for both Right Ascension (RA) and Declination (Dec) axes.
*   **An efficient power management solution**, featuring an onboard TPS5450 buck converter module and AMS1117 LDO voltage regulator chips to ensure clean and stable system voltage, even under heavy loads.
*   Integrated interfaces for **home sensors and limit switches**, ensuring device safety during operation.
*   Support for remote control via Wi-Fi, Bluetooth, or USB routing, allowing for seamless integration with mainstream astrophotography software ecosystems (N.I.N.A., INDI, ASCOM, etc.).
*   The control board's PCB layout is designed with future expandability in mind; using jumpers located next to the main board's driver slots, you can easily physically switch between the advanced **SPI Smart Control Mode** and the traditional **STEP/DIR Independent Mode** (firmware reflashing is required after switching modes).

*   **TMC5160 SPI Mode (Recommended Configuration) 🚀**
*   **Hardware Setup:** Use jumper caps to bridge the dedicated SPI pin headers, directly connecting the ESP32's SPI bus to the TMC5160 drivers.
*   **System Advantages:** Completely eliminates the need for manual potentiometer adjustment, allowing the OnStep firmware to directly read from and write to the driver registers. You can configure run/hold currents and precisely set microstepping resolutions in real-time—either via the web interface or within `Config.h`—and intelligently switch between StealthChop (ultra-silent operation) and SpreadCycle (high dynamic torque) modes to maximize the performance of the harmonic drive reducer.
*   **Firmware Integration:** Within `Config.h`... The RA/Dec drive model is defined as `TMC5160_SPI`.

* **Traditional STEP/DIR Mode (Backward Compatible) 🔌**
* **Hardware Setup:** Disconnect the SPI communication jumpers.
* **Use Case:** Serves as a reliable low-level fallback solution. If you need to temporarily substitute standard A4988 or DRV8825 drivers, or if you choose to utilize the "Standalone" mode of the TMC drivers, this design offers exceptional hardware fault tolerance and ease of maintenance.
* **Firmware Integration:** You must precisely enter the microstep values—as configured via jumper caps—into the `Config.h` file. Furthermore, you **must manually use a multimeter and a ceramic screwdriver** to adjust the VREF potentiometer on the driver board to set the motor current.

All circuit schematics and PCB layouts—which feature strictly controlled safety clearances, via dimensions, and high-current trace widths—were designed using open-source tools. You can order the PCBs directly from [JLC PCB](https://member.jlc.com/?spm=PCB.Homepage) or modify the designs to suit your specific requirements.

### PCB Manufacturing and Assembly

![PCB front](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/PCB/PCB-1.jpg)

![PCB reverse-side driver installation](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/PCB/PCB-2.jpg)

![PCB reverse-side TMC2209 installation](https://github.com/Leo-John233/DIY-HarmonicEquatorialMount/blob/main/images/PCB/PCB-3.jpg)

The following sequence is recommended for PCB manufacturing and assembly:

1. **Submit the PCB files**
   * Upload the Gerber archive for the corresponding board revision directly to the PCB manufacturer
   * Before ordering, verify the board outline, mounting holes, slots, and connector openings, and make sure the manufacturing platform has not automatically scaled or modified the design
   * If the board thickness, copper weight, surface finish, or component footprints are changed, recheck enclosure clearance, connector height, and high-current trace capacity

2. **Verify components and orientation**
   * Before soldering, check every component against the schematic, BOM, PCB silkscreen, and footprint orientation
   * Pay particular attention to diodes, LEDs, electrolytic capacitors, voltage regulators, the ESP32, the ESP-07S, and other polarized or orientation-sensitive components
   * Install the CR1220 battery according to the PCB marking. In the photographed assembly, the positive side faces outward
   * Keep metal plates, shielding, and enclosure walls away from the ESP32 antenna area to avoid degrading wireless performance

3. **Recommended soldering order**
   * Begin with low-profile SMD components such as resistors, capacitors, diodes, and small ICs
   * Continue with the power circuitry, pin headers, driver sockets, buttons, and LED connectors
   * Install the XH2.54 connectors, USB connector, network connector, DC jack, XT60 connector, and other tall components last
   * Install the TMC drivers, ESP modules, and coin-cell battery only after completing short-circuit checks and basic power-rail testing

4. **Driver installation**
   * The driver modules are installed on the reverse side of the PCB. Before insertion, verify the PCB silkscreen against the driver's `GND`, `VM`, `EN`, `STEP`, and `DIR` pins
   * TMC2209 and TMC5160 modules from different manufacturers may use different pin arrangements, pad layouts, or thermal structures. Do not determine orientation from appearance alone
   * For TMC5160 SPI mode, close the corresponding SPI jumpers and select `TMC5160_SPI` in the firmware
   * For TMC2209 or other STEP/DIR drivers, open the SPI jumpers and configure the microstepping and driver current to match the installed hardware
   * Reversed installation or hot-plugging can immediately damage the driver and the controller board

5. **First power-up**
   * Before applying power, use a multimeter to check for an obvious short circuit between the main power input and ground
   * A current-limited bench power supply is recommended for the first test, beginning with a low current limit
   * During the first power-up, leave the motors, sensors, and driver modules disconnected. Confirm that the power section does not overheat and verify the main voltage rails
   * After confirming controller operation, USB communication, and wireless connectivity, install the drivers one at a time and then connect the RA and Dec motors
   * Perform the first motor test at low current and low speed. Verify direction, home-sensor behavior, and limit logic before increasing the operating parameters

6. **Interface connections**
   * Connect `LIMIT-RA`, `LIMIT-DEC`, `HOME-RA`, and `HOME-DEC` to the corresponding axis limit and home sensors
   * Confirm the pin order in the schematic before connecting the `RA-XH2.54`, `DEC-XH2.54`, and `BRAKE` interfaces
   * Observe LED polarity when connecting the status, link, and power indicators
   * Do not use multiple main power inputs simultaneously unless their electrical relationship and intended use have been confirmed from the schematic

## 💻Firmware Configuration and Flashing

The main controller for the harmonic equatorial mount features an **ESP32-WROOM-32UE (16MB)** module. This high-capacity core not only effortlessly runs the complex [OnStep / OnStepX](https://onstep.groups.io/g/main/wiki) firmware but also provides ample storage space for built-in star catalogs, high-precision pointing models, and a feature-rich Smart Web UI.

Please refer to [Firmware_Setup.EN.md](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/software/Firmware_Setup.EN.md) for a complete list of `Config.h` parameters and detailed pin mappings.

---

## 🛠️Mechanical Assembly Guide

Please refer to [Assembly.EN.md](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/hardware/Assembly.EN.md)

Every component has been meticulously selected or designed to ensure high mechanical precision and ease of assembly—from the robust, CNC-machined aluminum body and the 25:100 harmonic drive reducer, to the 3D-printed protective enclosures for the stepper motors and controllers. Each stage has undergone rigorous testing and pointing model calibration to achieve sub-arcsecond tracking accuracy.