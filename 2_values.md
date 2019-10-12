# Values

A `Value` is a basic type. You can think of it as a literal value, but you generally don't use `Value` directly. Instead, you use subtypes such as `Const` or `Signal`.

## Const

A `Const` is a literal value, with a given number of bits, that does not change. 

## Shapes: widths and signs

But how does nMigen know how many bits the value has? If you simply construct a `Const` out of an integer, like `a = Const(10)` then you'll get exactly as many bits as needed to fit the integer. In this example, four bits. However, you can specify the number of bits you want by adding its shape: `a = Const(10, shape=16)` will give you a 16-bit value.

You can also specify that the value is signed, which affects things like magnitude comparisons. `a = Const(-10)` will automatically create a signed value. But `a = Const(10, shape=(16, True))` explicitly creates a 16-bit signed value.

Note that signed values are 2's complement values. `Const(-10)` is a 4-bit signed value, but it takes up 5 bits. You can call the extra bit the sign bit, which is somewhat misleading because that would imply 1's complement values (where `-15` is `11111`). But the bit width in either case turns out the same. Adding signedness to a value increases its effective width by one.

You can also retrieve the shape of a `Const`:

```python
>>> from nmigen import *
>>> a = Const(10)
>>> a.shape()
(4, False)
>>> a.width
4
>>> a.signed
False
```

```python
>>> from nmigen import *
>>> a = Const(-10)
>>> a.shape()
(4, True)
>>> a.width
4
>>> a.signed
True
```

Why doesn't `width` show 5 bits? Because the width in a shape is meant to be taken along with the signedness.

## Signal

A `Signal` is a value that does change. Signals may be implemented as flipflops or wires. The backend will figure it out based on how they are used. It's the equivalent of `wire`, `reg`, or `logic` in Verilog.

Just like a `Const`, a `Signal` has a shape: a width and a signedness. You can create a 16-bit unsigned `Signal`, for example, by `a = Signal(16)` or `a = Signal(shape=16)`. You can specify a signed `Signal` via `a = Signal((16, True))`. Note that the argument to `Signal` is a tuple. If you forget that, you'll run into an error.

If you don't specify a shape at all: `a = Signal()`, the shape of such a `Signal` is one bit, unsigned.

Again, like with `Const`, you can retrieve the shape of a `Signal`:

```python
>>> from nmigen import *
>>> a = Signal((16, True))
>>> a.shape()
(16, True)
>>> a.width
16
>>> a.signed
True
>>>
```
