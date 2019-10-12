# Modules

## Basic structure

```python
from nmigen import *

class ThingBlock(Elaboratable):
    def __init__(self):
        pass

    def elaborate(self, platform: str):
        m = Module()
        return m
```

## Elaborating a module

```python
from nmigen.cli import main

if __name__ == "__main__":
    sync = ClockDomain()
    clk = ClockSignal()
    rst = ResetSignal()

    block = ThingBlock()

    m = Module()
    m.domains += sync
    m.submodules += block

    main(m, ports=[clk, rst])
```

* `main(module, ports=[<ports>], platform="<platform>")` translates the given module, including any submodules recursively, in either Verilog or RTLIL. All `elaborate()` methods will have its `platform` argument set to the given `platform`. Elaboratables might create different logic for different platforms.

```
python3 thing.py generate -t [v|il]
```

## Domains

A _domain_, in its basic definition, is a grouping of logic elements. If we consider a module as a black box with inputs and outputs, then any given output is generated within one and only one domain. If you attempt to set an output in more than one domain, you'll get an error.

`Modules` come with two domains built in: a combinatorial domain and a synchronous domain.

The domains in a `Module` can be accessed through its `d` attribute.

### Combinatorial

Logic that contains no clocked elements is called _combinatorial_: it just combines logic elements together. This is one of the domains that a `Module` contains. It is always named `comb`, and it can be accessed via `m.d.comb`.

### Synchronous

Synchronous logic is called synchronous because all of the flipflops (FFs) within a particular clock domain all change, in synchrony, according to the clock domain's clock. Each clock domain also has a reset signal which can reset all FFs to a given state. Finally, the domain specifies the edge on which all the FFs change: positive or negative.

FFs that are not clocked using the edge of a given clock domain cannot be in that clock domain. By definition, they have a different clock and reset, and so belong in a different clock domain.

Some hardware supports only one clock domain. Many FPGAs support at least two clock domains.

Unless otherwise specified, there is one synchronous domain in a `Module` called `sync`. It can be accessed via `m.d.sync`.

### Creating more domains

There is no reason to create combinatorial domains. As mentioned above, modules already contain one combinatorial domain, `comb`.

You can create a synchronous clock domain using `ClockDomain(domain="<domain-name>", clk_edge="<pos|neg>")`. This gives you both the clock and the reset signal for the domain. By default, the domain name is `sync` and the clock edge is `pos`.

You can access a domain via its name. So a domain created via `ClockDomain(domain="other_stuff")` is accessed via `m.d.other_stuff`.

* `ClockSignal(domain="<domain>")` gives you the clock signal for the given domain.
* `ResetSignal(domain="<domain>")` gives you the reset signal for the given domain.

### Tip: clock domains with the same clock but different edges

This can be done simply by creating one `ClockDomain` for the positive edge using and another `ClockDomain` with the same domain name but with `clk_edge="neg"`.
