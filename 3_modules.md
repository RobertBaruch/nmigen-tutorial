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

    block = ThingBlock()

    m = Module()
    m.domains += sync
    m.submodules += block

    main(m, ports=[sync.clk, sync.rst])
```

* `main(module, ports=[<ports>], platform="<platform>")` translates the given module, including any submodules recursively, in either Verilog or RTLIL. This is called _elaboration_. All `elaborate()` methods will have its `platform` argument set to the given `platform`. Elaboratables might create different logic for different platforms.

```
python3 thing.py generate -t [v|il]
```

## Domains

A _domain_, in its basic definition, is a grouping of logic elements. If we consider a module as a black box with inputs and outputs, then any given output is generated within one and only one domain. If you attempt to set an output in more than one domain, you'll get an error during elaboration that the signal has more than one driver.

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

You can access this domain's clock via `m.d.other_stuff.clk` and you can access its reset signal via `m.d.other_stuff.rst`.

You can also get the clock and reset signals without access to the domain:

* `ClockSignal(domain="<domain>")` gives you the clock signal for the given domain.
* `ResetSignal(domain="<domain>")` gives you the reset signal for the given domain.

### Tip: clock domains with the same clock but different edges

This can be done simply by creating one `ClockDomain` for the positive edge using and another `ClockDomain` with the same domain name but with `clk_edge="neg"`.

## Ports

The equivalent of ports in a module is public attributes. In the following example, `a` and `data` are publically available to other modules, while `b` is not, just as `a` and `data` are publically available to other Python classes, and `b` is not.

```python
class ThingBlock(Elaboratable):
    def __init__(self):
        self.a = Signal()
        self.data = Signal(8)

    def elaborate(self, platform: str):
        m = Module()

        b = Signal()

        return m
```

## Reset/default values for signals

If a signal is set in the _combinatorial_ domain, then you can specify the default value of the signal if it is not set. By default, this is zero, but for a non-zero value, you can specify the default value for a signal when constructing the signal by setting the `reset` named parameter in the constructor. For example, this creates a 16-bit unsigned signal, `self.x`, which defaults to `0x1000` if not set:

```python
self.x = Signal(unsigned(16), reset=0x1000) # Yes, reset.
```

Likewise, if a signal is set in a _synchronous_ domain, then you can specify its reset value using the `reset` named parameter in the constructor. By default the reset value is zero.

### Explicitly not resetting

For synchronous signals (that is, signals set in a synchronous domain), you can specify that it is not reset on the reset signal, instead only getting an _initial value_ on power-up. This is done by setting the `reset_less` named parameter in the constructor to `True`:

```python
self.x = Signal(unsigned(16), reset=0x1000, reset_less=True)
```

This would create a 16-bit unsigned signal that is initially set to `0x1000`, but is not reset to that value when the domain's reset signal is activated.

This is especially useful during simulation or formal verification where you want to activate the reset, but keep some signals "outside" the reset. For example, a cycle counter that maintains its count across resets.
