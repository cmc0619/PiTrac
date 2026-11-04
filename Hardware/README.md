# V3 hardware: upstream-based corrections

This revision starts from PiTracLM/PiTrac `78f1de60fd21398f34d348f19f55e73029bc95f9` and makes three physical corrections. It retains the original architecture and populated parts except for R5's resistance value.

| Change | Reason and effect |
| --- | --- |
| R5: 2.49 kOhm to 6.8 kOhm, `MFR-25FTE52-6K8` | The green supply indicator's original 0.25 W resistor dissipates approximately 0.450 W at the nominal 35.57 V rail. The replacement dissipates approximately 0.165 W and reduces rail load by about 0.303 W. The green indicator is dimmer; the IR LED current-setting network is unchanged. |
| J2/J5 USB4085 power-contact copper | The original front/inner copper diameter was 0.10 mm around a 0.40 mm plated drill. The corrected 0.68 mm land gives a nominal 0.14 mm annular ring. Bottom solder lands are extended to surround the same drills; the local thermal gap is 0.20 mm. |
| D6-D15 center anode connections | The VSMA1085400 center contact is an anode. It is now connected to each LED's existing anode net with a 2.8 x 0.4 mm copper bridge. The bridge stays under solder mask; existing paste and mask openings are unchanged. |

Every existing component position and orientation, mounting hole, drilled pad, via, outline, and 3D model placement is preserved. Existing cases and printed-part geometry are unchanged. Software, the DAC setting/scale, boost current-limit resistor, LED current-setting network and pulse timing are unchanged. No new fuse, TVS, pulse limiter, or USB power-delivery controller is added.

Project-local library tables and project-specific transistor/LED symbols preserve the actual upstream pin definitions when the project is opened on another computer. Intentional unconnected pins, power sources and breakaway holes are identified explicitly. The connector-only board now has its own project file with the same fabrication rules as the combined board.

## Files to use

- `KiCad Source Files/`: native KiCad 9 schematics, project files, boards and local libraries. KiCad 10 can also read/check these files.
- `Fabrication Files/`: regenerated Gerber and separate plated/nonplated drill ZIPs for all three board variants. Existing archive names containing `IRLED2` are retained.
- `Assembly Files/`: regenerated BOMs and centroids. Reference sets and placement coordinates are unchanged; R5 is the only changed ordered part.
- `Schematic/V3 Connector + IRLED2 Schematic.pdf`: regenerated complete schematic.
- `Validation/upstream-iteration/`: verification summary and focused resistor calculation/SPICE evidence.

U4 and C13 remain **do not populate**, as in upstream. They are omitted from the assembly files. With U4 absent, R11 already pulls the MCP4801 shutdown pin high; no DAC modification is needed.

## Verification

KiCad 9.0.8 checks all three boards with zero DRC errors and zero unconnected copper items. The full schematic has zero active ERC violations and two documented, narrowly scoped exceptions: the MC33063 switch emitter tied to ground and the MCP1407's internally connected output pins tied together. No global rule severities were relaxed.

KiCad 10.0.6 independently checks the combined board and full schematic with the same zero-error result. Residual warnings are reported explicitly:

- Each connector board has 16 footprint-library mismatch warnings: 14 standard-footprint differences and two USB instance/library differences. The LED-only board has none.
- Combined-board schematic parity reports 18 unused USB signal pins without copper pads. These dedicated power-only footprints retain those unused contacts as unplated holes, as upstream.
- KiCad 10 additionally reports 32 legacy empty-versus-`~` Datasheet-field differences. These do not change components or connections.

Independent checks compare the original and final netlists, native pad assignments, tracks/vias, holes, outlines, component placements, DNP flags and assembly data. Original pin connectivity is unchanged apart from the ten intended center-anode additions. A separate reviewer found no material introduced defect.

The focused ngspice calculation models only the indicator branch, using a nominal 2.1 V LED drop. Its results agree with resistor-power arithmetic. A conservative 38 V/zero-LED-drop bound including resistor tolerance and temperature coefficient gives approximately 0.215 W for the replacement, below its 0.25 W rating through 70 C local ambient. Higher-temperature operation requires the manufacturer's derating curve and actual conditions.

## Boundaries

These checks establish CAD consistency and the stated electrical calculations. They do not establish measured light output, enclosure temperature, solder yield or physical fit of a populated assembly.

The existing MC33063 charging-current limit remains a separate issue: its 0.15 Ohm sense resistor implies a nominal 2 A threshold, above the controller's 1.5 A switch rating. A lower limit changes reservoir recharge; this revision does not claim to resolve it while preserving the complete strobe/calibration operating envelope. The actual calibration pulse includes software/SPI overhead, so its duration cannot be inferred solely from the SPI wire time.

J2/J5 remain dedicated Pi5 power outputs without USB PD negotiation. Adding ordinary Type-C source resistors would not establish 5 A PD capability. The existing Pi5 supply configuration and upstream current requirements still apply.

## Manufacturer references

- [Yageo MFR resistor ratings, dimensions and ordering](https://yageogroup.com/content/Resource%20Library/Datasheet/YAGEO-MFR_DATASHEET.pdf)
- [Vishay VSMA1085400 package and recommended copper, page 6](https://www.vishay.com/docs/80294/vsma1085400.pdf)
- [GCT USB4085 connector](https://gct.co/connector/usb4085)
- [TI MC33063A controller](https://www.ti.com/lit/ds/symlink/mc33063a.pdf)
- [Microchip MCP1406/1407 output-pin description](https://ww1.microchip.com/downloads/en/DeviceDoc/20002019C.pdf)
- [Raspberry Pi 5 USB Power Delivery](https://pip-assets.raspberrypi.com/categories/685-app-notes-guides-whitepapers/documents/RP-009856-WP-1-USB%20Power%20delivery%20on%20Raspberry%20Pi%205.pdf)
