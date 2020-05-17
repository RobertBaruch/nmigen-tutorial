# Coming soon

# Supported devices

See the [vendor directory](https://github.com/m-labs/nmigen/tree/master/nmigen/vendor) for supported devices and toolchain details.

Devices supported as of 20 Dec 2019:

| Device            | Platform classname        | Toolchain required |
| ----------------- | ------------------------- | ------------------ |
| Intel             | IntelPlatform             | Quartus            |
| Lattice ECP5      | LatticeECP5Platform       | Trellis            |
| Lattice ICE40     | LatticeICE40Platform      | IceStorm/iCECube2  |
| Lattice MachXO2   | LatticeMachXO2Platform    | Diamond            |
| Xilinx 7 series   | Xilinx7SeriesPlatform     | Vivado             |
| Xilinx Spartan 3A | XilinxSpartan3Or6Platform | ISE                |
| Xilinx Spartan 6  | XilinxSpartan3Or6Platform | ISE                |
| Xilinx UltraScale | XilinxUltraScalePlatform  | Vivado             |


# Defining your board

Many boards are defined for you at [nmigen_boards](https://github.com/m-labs/nmigen-boards/tree/master/nmigen_boards).

You can copy one from there and modify it to suit your needs, or create a new class subclassed from one of the above supported device platform classes.

## Class properties

* `device`: a string. See the base platform class for which one to choose. This affects options passed to the toolchain so that it compiles for the correct chip.
* `package`: a string. See the base platform class for which one to choose. This affects options passed to the toolchain so that it compiles for the correct package of the chip.
* `resources`: a list of `Resource`. This names the pins you want to use, and configuration options for each such pin.
* `default_clk`: the name of the resource that is the clock for the default clock domain.
* `default_rst`: the name of the resource that is the reset for the default clock domain.
* `connectors`: optional, a list of `Connector`. It isn't obvious what purpose this serves. It may have something to do with certain toolchains.

## Resources

A `Resource` is a structure that contains a name, a number, and one or more configuration items for the resource. Adding a `Resource` to a board does two things:
* configures pins on the device
* allows you to request a resource's `Pin` by name from the platform in your `elaborate` function. Such a `Pin` has several `Signal` associated with it, such as `i` for input and `o` for output, which you can then use in your module.

For example, by including this `Resource` into the platform's `resources` list:

```python
    Resource("abc", 0, Pins("J3", dir="i"))
```

then pin `J3` on the device will be configured as an input, and you can request the `abc` resource's input `Signal` like this:

```python
    platform.request("abc").i
```

### Resource configuration items

* `Pins`: specifies the space-separated pin names associated with the resource, their direction type, and whether the signal should be automatically inverted when crossing the pin (for, e.g., active low signals). Direction types are:
  * `i`: input only. Signal for the pin is `.i`
  * `o`: output only. Signal for the pin is `.o`
  * `io`: bidirectional. Signals for the pin are `.i` for the input, `.o` for the output, and `.oe` is the direction for the pin: 0 for input, 1 for output
  * `oe`: tristate. Signals for the pin are `.o` for the output, and `.oe` to enable output: 0 for disable, 1 for enable

* `PinsN`: shorthand for `Pins`, where all pins are active low.

* `DiffPairs`: specifies the space-separated pin names for one or more differential pairs (positive and negative pins)

* `Clock`: specifies that the resource is a clock with the given frequency in Hz.

* `Attrs`: platform-specific attributes such as voltage standard to select.

A full resource specification for a clock pin used on a Lattice ICE40 board could be as follows:

```python
    Resource("clk", 0, Pins("J3", dir="i"), Clock(12e6), Attrs(GLOBAL=True, IO_STANDARD="SB_LVCMOS"))
```

This says that the `clk` resource is at pin `J3` on the FPGA, has a frequency of 12MHz, is a "global" signal, and uses the LVCMOS voltage standard. Without knowing about the toolchain for the platform, you will not know what attributes are required.

## Example

Example for the Lattice ICE40-HX8K breakout board.

```python
from nmigen.build import Platform, Resource, Pins, Clock, Attrs, Connector
from nmigen.build.run import LocalBuildProducts
from nmigen.cli import main_parser, main_runner
from nmigen.vendor.lattice_ice40 import LatticeICE40Platform

class ICE40HX8KBEVNPlatform(LatticeICE40Platform):
    device = "iCE40HX8K"
    package = "CT256"

    resources = [
        Resource("clk1", 0, Pins("J3", dir="i"), Clock(12e6),
                 Attrs(GLOBAL=True, IO_STANDARD="SB_LVCMOS")),  # GBIN6
        Resource("rst", 0, Pins("R9", dir="i"),
                 Attrs(GLOBAL=True, IO_STANDARD="SB_LVCMOS")),  # GBIN5
        Resource("led", 0, Pins("C3", dir="o"),
                 Attrs(IO_STANDARD="SB_LVCMOS")),  # LED2
    ]

    default_clk = "clk1"  # you must have a default clock resource
    default_rst = "rst"   # you must have a default reset resource

    connectors = [
        Connector(
            "j",
            1,  # J1
            "A16 -   A15 B15 B13 B14 -   -   B12 B11 "
            "A11 B10 A10 C9  -   -   A9  B9  B8  A7  "
            "B7  C7  -   -   A6  C6  B6  C5  A5  C4  "
            "-   -   B5  C3  B4  B3  A2  A1  -   -   "),
        Connector(
            "j",
            2,  # J2
            "-   -   -   R15 P16 P15 -   -   N16 M15 "
            "M16 L16 K15 K16 -   -   K14 J14 G14 F14 "
            "J15 H14 -   -   H16 G15 G16 F15 F16 E14 "
            "-   -   E16 D15 D16 D14 C16 B16 -   -   "),
        Connector(
            "j",
            3,  # J3
            "R16 -   T15 T16 T13 T14 -   -   N12 P13 "
            "N10 M11 T11 P10 -   -   T10 R10 P8  P9  "
            "T9  R9  -   -   T7  T8  T6  R6  T5  R5  "
            "-   -   R3  R4  R2  T3  T1  T2  -   -   "),
        Connector(
            "j",
            4,  # J4
            "-   -   -   R1  P1  P2  -   -   N3  N2  "
            "M2  M1  L3  L1  -   -   K3  K1  J2  J1  "
            "H2  J3  -   -   G2  H1  F2  G1  E2  F1  "
            "-   -   D1  D2  C1  C2  B1  B2  -   -   "),
    ]

    def toolchain_program(self, products: LocalBuildProducts, name: str):
        iceprog = os.environ.get("ICEPROG", "iceprog")
        with products.extract("{}.bin".format(name)) as bitstream_filename:
            subprocess.check_call([iceprog, "-S", bitstream_filename])

# Important! For WSL, install
# https://github.com/FPGAwars/toolchain-icestorm/releases/download/v1.11.1/toolchain-icestorm-windows_x86-1.11.1.tar.gz
#
# See also the somewhat outdated https://github.com/FPGAwars/toolchain-icestorm/wiki#testing-iceprog

if __name__ == "__main__":
    ICE40HX8KBEVNPlatform().build(Blinker(), do_program=True)  # Set to False on WSL!
```

# Building

```
$ python3 file.py
```

This will result in a directory, `build`, containing the output files:

* `top.il`: The ilang output for yosys.
* `top.bin`: The bitstream to send to the device (e.g. via iceprog)
* `top.rpt`: Statistics from nextpnr. The most useful is the cell and LUT count at the end.
* `top.tim`: Timing analysis. Shows how fast you can go. Probably.

# Appendix: Known Lattice ICE40 attributes

* `GLOBAL`: bool. If True, the pin is global. Global pins are designated `GBIN` in the datasheet pin listing.
* `IO_STANDARD`: string:
  * `SB_LVCMOS`: for all single-ended pins and differential output pins
  * `SB_LVDS_INPUT`: for differential input pins

Other attributes may be supported, but are not documented.
