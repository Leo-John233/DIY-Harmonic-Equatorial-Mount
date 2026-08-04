<div align="center">
  <a href="./Assembly_CN.md"><img src="https://img.shields.io/badge/简体中文-4285F4.svg?style=flat&logo=googletranslate&logoColor=white" alt="简体中文"></a>
  <a href="./Assembly_EN.md"><img src="https://img.shields.io/badge/English-4285F4.svg?style=flat&logo=googletranslate&logoColor=white" alt="English"></a>
</div>

# Dual 25-100 Harmonic Equatorial Mount Assembly Instructions

Combining real-life footage from the `images/Assembly process` directory, this guide explains the complete assembly process, from the mechanical structure to the electrical control system, step-by-step.

## 🛠️ Preparation and Materials List (BOM)

Before starting assembly, please ensure you have the following core components and tools ready (refer to the BOM for specific hardware requirements):

* **Transmission Core**: 25-100 type harmonic reducer (1:100 reduction ratio)
* **Idler Gears**: 2GT 10mm wide 80-tooth idler gear and 2GT 10mm wide 19mm high 20-tooth idler gear
* **Motor and Protection**: Stepper motor, power-off brake (electromagnetic damping/holding brake) module
* **Main Control and Drive**: ESP32 + ESP07S combination, TMC5160 stepper motor drive module
* **Circuit Board**: Custom-designed main control PCB (supports 12V/24V power supply)
* **Wires and Accessories:** Power switch leads, various types of Allen screws, lubricant, and screw glue.

*Note: For the idler gear, you need some machining skills to modify the finished idler gear according to the specifications in the drawings. If you cannot modify it, please modify the model to fit standard parts available on the market.*

---

## 🖨️ Phase One: 3D Printing and Machining Preparation

Before formal assembly, it is recommended to complete the following preparations:

1. **3D Printing:** Use PETG-GF/CF material to print all necessary parts, ensuring print quality and dimensional accuracy, especially for key components such as the reducer protective cover, drive shaft, and sensor mounting bracket.

      **Printed Parts List:**

    | Item No. | Part Location | Corresponding Source File |
    | :---: | :--- | :--- |
    | 1 | Main Housing | `Housing.SLDPRT` |
    | 2 | Dovetail Slot Housing | `Dovetail Slot Housing.SLDPRT` |
    | 3 | Declination / Right Ascension Axis | `DEC Axis.SLDPRT` , `RA Axis.SLDPRT` |
    | 4 | Dovetail Slot Housing | `Dovetail Slot Housing.SLDPRT` |
    | 5 | Control Board Cover | `Control Board Cover - Insert.SLDPRT` |
    | 6 | Photoelectric Sensor Mount | `Photoelectric Sensor Mount A.SLDPRT`, `Photoelectric Sensor Mount B.SLDPRT`, `Photoelectric Sensor Mount B - Heightening.SLDPRT` |

> 📷 *Reference Image:*
> ![3D Printed Parts](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(20).jpg "3D Printed Parts")
> ![3D Printed Parts](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(21).jpg "3D Printed Parts")
> ![3D Printed Parts](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(22).jpg "3D Printed Parts")

2. **Fuse Hot-Melt Nuts into Printed Parts:** Before assembly, fuse the prepared hot-melt nuts into all 3D printed parts that require screws. Refer to the markings in the image below for the locations where nuts will be installed. Ensure each nut is securely fixed in the printed part and accurately positioned for easy screw installation during subsequent assembly.

> 📷 *Reference Images:*
> ![Installing the Hot Melt Nut](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(23).jpg "Installing the Hot Melt Nut")
> ![Installing the Hot Melt Nut](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(24).jpg "Installing the Hot Melt Nut")
> ![Installing the Hot Melt Nut](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(25).jpg "Installing the Hot Melt Nut") ![Installing the thermoplastic nut](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(26).jpg "Installing the thermoplastic nut")

3. **Machining**: Modify the idler wheel as necessary to ensure its dimensions and mounting hole positions meet design requirements. Simultaneously, drill and tap holes in the aluminum plate, preparing the necessary screws and copper posts for installation.

---

## ⚙️ Phase Two: Mechanical Structure Assembly

### 1. Harmonic Reducer Installation (RA/DEC Axis)
In this step, we will install the core components of the equatorial mount. Considering the stress requirements of the radial slow-speed rotating load, please ensure that the flange contact surfaces are flat and in close contact during installation.

1. Precisely align the reducer flange face with the screw holes on the aluminum plate, making the center through hole of the aluminum plate as coaxial as possible with the hollow shaft of the reducer. Use hex socket head cap screws and tighten them evenly in a diagonal sequence to ensure uniform stress on the reducer housing and avoid deformation caused by excessive tightness or vibration caused by excessive looseness.
2. Two screw holes require longer screws, and lock nuts are not needed for subsequent motor mount installation.
3. Four screw holes do not require screws for subsequent installation of the 3D-printed reducer protective cover.
4. Tighten the excess parts of the remaining core screw components (excluding those from step 2) into lock nuts to ensure a secure connection between the reducer and the aluminum plate that will not loosen due to vibration.

> 📷 *Reference Images:*
> ![Installing Harmonic Reducer](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(1).JPG "Installing Harmonic Reducer")
> ![Installing Harmonic Reducer](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(2).JPG "Installing Harmonic Reducer")
> ![Installing Harmonic Reducer](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(3).JPG "Installing Harmonic Reducer")

### 2. Transmission Assembly

1. Install the machined 80-gear 2GT idler wheel onto the input shaft of the reducer, ensuring the idler wheel is coaxial with the input shaft. Use Allen screws and tighten them evenly in a diagonal sequence to ensure a secure connection that will not loosen due to vibration.
2. Install the 20-gear 2GT idler wheel onto the output shaft of the stepper motor.
3. Mount the stepper motor onto the motor mount using copper pillars.
4. **Power-off Brake Installation**: Replace the original motor screws at the tail end of the dual-output-axis stepper motor used for the RA axis and install an electromagnetic brake module. This effectively prevents damage to the telescope's "head" due to counterweight imbalance during unexpected power outages.
5. Place a 2GT 10mm wide synchronous belt onto the motor idler wheel beforehand for easier subsequent installation.

> 📷 *Reference Image:*
> ![RA Axis Motor and Brake Module Installation](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(15).jpg "RA Axis Motor and Brake Module Installation")
> ![RA Axis Motor and Brake Module Installation](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(16).jpg "RA Axis Motor and Brake Module Installation")
> ![DEC Axis Motor Installation](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(17).jpg "DEC Axis Motor Installation")

### 3. Assembly of RA Axis Latitude Seat Flange
1. 1. Connect the RA shaft flange to connecting shafts A and B using hex socket screws.
2. During step 1, simultaneously assemble the push rod and push block structure, ensuring the push rod can slide smoothly within the push block, and that connecting shafts A and B can achieve relative movement via the push rod.
3. Install the assembled structure from steps 1 and 2 onto the flange of the harmonic reducer using hex socket screws. Simultaneously install the 3D-printed reducer protective cover during the installation process.

> 📷 *Reference Image:*
> ![Assembly of Latitude Mount](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(7).JPG "Assembly of Latitude Mount")
> ![Assembly of Latitude Mount](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(8).JPG "Assembly of Latitude Mount")
> ![Assembly of Latitude Mount](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(10).JPG "Assembly of Latitude Mount")

### 4. Wiring Sequence of Photoelectric Sensor and Motor

1. Confirm that the purchased photoelectric sensor is a slot-type photoelectric switch, four-wire NPN type, and ensure that its output signal is compatible with the input pins of the ESP32 main control board.
2. Configure the wiring of the zero-position sensor (HOME sensor) as follows: brown wire to +5V power supply, blue wire to GND, black wire (i.e., NPN NO).
3. Configure the wiring of the limit sensor (LIMIT sensor) as follows: brown wire to +5V power supply, blue wire to GND, white wire (i.e., NPN NC).
4. Configure the wiring sequence of the motors corresponding to the DEC and RA axes as follows: A-, A+, B+, B-, ensuring the correct phase sequence of the motors.

### 5. Connection of Motor, Synchronous Belt and Reducer, and Installation of Sensor and Flange

1. Install the motor onto the longer screws pre-drilled on the aluminum plate in step 1 through the pre-drilled holes on the motor mount. Tighten the synchronous belt and tighten the anti-loosening nuts on the screws to ensure that the synchronous belt does not slip or become excessively loose during transmission.
2. Adjust the height of the idler pulley on the motor output shaft appropriately so that it is on the same horizontal plane as the idler pulley on the reducer input shaft. This ensures even tension distribution of the synchronous belt and avoids excessive wear or reduced transmission efficiency due to non-parallelism.
3. After completing the above steps, manually rotate the large idler pulley to check if the synchronous belt runs smoothly and ensure there are no abnormal friction sounds or vibrations. This will help confirm the correct installation and adjustment of the transmission system.
4. After installation, it is recommended to spray an appropriate amount of silicone lubricant on the synchronous belt to reduce friction and abnormal noise and extend the service life of the synchronous belt.
5. Install the photoelectric sensor limit switches and origin switches on the RA/DEC shafts to ensure that their positions can accurately detect the extreme positions and origin positions of the shaft for safety protection and precise positioning.
6. Install the 3D-printed reducer protective cover on the outside of the reducer, using the pre-drilled screw holes and hex socket screws for fixing. This ensures that the protective cover can effectively prevent dust and debris from entering the reducer without interfering with its normal operation.
7. Install the flange of the RA/DEC shaft, ensuring it is coaxial with the shaft, and tighten it evenly with Allen screws to ensure the flange does not shift or loosen during operation.

> 📷 *Reference Image:*
> ![RA shaft motor and brake module, sensor, flange installation](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(5).JPG "Installation of RA axis motor, brake module, sensor, and flange")
> ![DEC axis motor, timing belt](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(9).JPG "DEC axis motor, timing belt")
> ![DEC axis flange installation](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(6).JPG "DEC axis flange installation")
> ![Sensor Installation](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(27).jpg "Sensor Installation")
> ![Sensor Installation](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(28).jpg "Sensor Installation")
> ![Sensor Installation](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(29).jpg "Sensor Installation")

### 6. Housing Assembly
1. Connect the assembled RA and DEC axes mechanical structures using aluminum blocks. Utilize a marble-like horizontal surface to ensure all aluminum plates are installed perpendicularly and orthogonally without interference.
2. Apply an appropriate amount of screw glue to the screw connections to ensure the stability and reliability of the mechanical connections during equipment operation.

> 📷 *Reference Image:*
> ![Shell Assembly](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(11).JPG "Shell Assembly")

---

## ⚡ Phase Three: Electrical Control Assembly and Wiring

### 1. PCB Base Plate and Driver Installation

1. Fix the assembled PCB board inside the electrical control compartment using copper pillars. During installation, carefully check the 12V/24V high-current power supply plane on the back to ensure sufficient insulation distance between it and the metal casing. 2. Connect the TMC5160 driver module and the ESP32 main control board to the corresponding female connectors in sequence, carefully checking the pin orientation to ensure they are correctly connected.

> 📷 *Reference Images:*
> ![PCB and Driver Installation](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(18).jpg "PCB and Driver Installation")
> ![PCB and Driver Installation](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC(19).jpg "PCB and Driver Installation")

### 2. Standard Wiring and Cable Management To ensure the absolute stability of the equipment in outdoor environments and under high current driving conditions, the wiring in this project has strict requirements:

1. The main power supply line must use standard **power switch leads** to connect to the PCB. **The use of any arbitrary "flying wires"** as a substitute for the power path is strictly prohibited.
2. Connect the control cables for the RA and DEC axis stepper motors to the corresponding terminals on the TMC5160.
3. Connect the control cables for the power-off protection system and tidy up all wiring harnesses to ensure that the cables do not become tangled or excessively stretched during the equatorial mount's extreme rotation.

> 📷 *Reference Image:*
> ![Standard Power Leads and Internal Routing](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(12).JPG "Standard Power Leads and Internal Routing")

---

## 💻 Phase Four: Firmware Configuration and Initial Debugging

The equatorial mount's hardware logic is designed specifically for the standard OnStep environment.

1. **Firmware Selection**: The system runs **standard OnStep firmware** (⚠️Please note: This project **does not apply** OnStepX; do not flash the wrong firmware version).
2. **Communication Confirmation**: Ensure the ESP32 pin mapping matches your modified OnStep firmware, and confirm the ESP07S's network communication connection is normal. 3. **Power-on Self-Test:** Connect to power and test whether the electromagnetic brake can engage/release normally. Monitor the operating status and microstepping settings of the TMC5160 via the host computer.

> 📷 *Reference Images:*
> ![Electronic Control System Lighting and Online Testing](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(13).JPG "Electronic Control System Lighting and Online Testing")
> ![Electronic Control System Lighting and Online Testing](https://github.com/Leo-John233/DIY-Harmonic-Equatorial-Mount/blob/main/images/Assembly%20process/DSC%20(14).JPG "Electronic Control System Lighting and Online Testing")

---

## 🤝 Open Source License and Contributions

If you find any optimizable mechanical tolerances or better wiring solutions during the assembly process, please submit an Issue or Pull Request. This project uses `GPLv3 + CERN-OHL-S` Dual open-source license, thank you for your contributions to the open-source astronomical hardware community!
