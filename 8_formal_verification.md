# Coming soon

## Bounded Model Checking

## Coverage

## Assert, Assume, and Cover for fun and profit.

```python
from nmigen.asserts import Assert, Assume, Cover
from nmigen.cli import main_parser, main_runner
from somewhere import Adder

if __name__ == "__main__":
    parser = main_parser()
    args = parser.parse_args()

    m = Module()
    m.submodules.adder = adder = Adder()

    m.d.comb += Assert(adder.out == (adder.x + adder.y)[:8])

    with m.If(adder.x == (adder.y << 1)):
        m.d.comb += Cover((adder.out > 0x00) & (adder.out < 0x40))

    main_runner(parser, args, m, ports=[] + adder.ports())
```

## Past, Rose, Fell, Stable

```python
from nmigen.asserts import Assert, Assume, Cover
from nmigen.asserts import Past, Rose, Fell, Stable
```

## The sby file

```
[tasks]
cover
bmc

[options]
bmc: mode bmc
cover: mode cover
depth 40
multiclock off

[engines]
smtbmc boolector

[script]
read_ilang toplevel.il
prep -top top

[files]
toplevel.il
```

## Running formal verification

Generate the output, then run Symbiyosys:

```
$ python3 adder_test.py generate -t il > toplevel.il
$ sby -f <file>.sby
```