# Basic operations

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

## Statements

nMigen doesn't convert Python to hardware. In essence, what you are writing using nMigen is a _generator_ of logic, not the logic itself. So if you want one `Value` to take the value of another, you don't write `a = b`, but instead you call the method of `a` that generates the equality: `a.eq(b)`. This is known as a _statement_.

However, many math operators are overridable in Python, since these translate to calls to Python functions. So for example, you can write `a.eq(b+1)` instead of something like `a.eq(b.plus(1))` because Python addition can be overridden to a function call, and nMigen's `Signal` class does that for all such operators.

### List of directly translatable Python operators

| Operator | Operation                | Notes                            |
| -------- | ------------------------ | -------------------------------- |
| `~`      | inversion                |
| `-`      | arithmetic negation      |
| `+`      | addition                 |
| `-`      | subtraction              |
| `*`      | multiplication           |
| `%`      | modulus                  |
| `//`     | division                 | Integer division, rounding down. |
| `<<`     | shift right              |
| `>>`     | shift left               | Logical, not arithmetic          |
| `&`      | bitwise and              |
| `|`      | bitwise or               |
| `^`      | bitwise xor              |
| `==`     | equality                 |
| `>`      | greater than             |
| `>=`     | greater than or equal to |
| `<`      | less than                |
| `<=`     | less than or equal to    |

Note that there are no translatable Python logical operators (`and`, `or`). Attempts to use an nMigen value as a boolean will result in an error saying `Attempted to convert nMigen value to boolean`.

### Effects of operations on result width

Python's integers are of potentially infinite bit width. In keeping with this philosophy, operations on signals create results with enough bits to hold the result, given the widths of the operands, and whether they are signed (represented in 2's complement) or not. Thus, adding two unsigned 4-bit signals must result in a 5-bit signal. The range of each operand is 0 to 15, so the range of the result is 0 to 30. This requires 5 bits.

```python
>>> from nmigen import *
>>> s1 = Signal(4)
>>> s2 = Signal(4)
>>> v = s1 + s2
>>> v.shape()
(5, False)
>>>
```

The same happens when adding two signed 4-bit signals. The range of each operand is -15 to +15, so the range of the result is -30 to 30.

However, adding a signed 16-bit signal to an unsigned 16-bit signal will result in an _18-bit_ signal:

```python
>>> s1 = Signal((16, True))
>>> s2 = Signal(16)
>>> (s1+s2).shape()
(18, True)
>>>
```

This is, perhaps, surprising. Consider a 4-bit signal. If unsigned, its representation is simply four bits. However, if signed, then it should be able to hold any integer from -15 to +15. This takes 5 bits in 2's complement: `10000` to `00000` to `01111`. Adding an unsigned 4-bit number adds as little as 0 to the range, and as much as 15. Thus the range of the result is -15 to +30. This is as good as -30 to +30, which is as good as -31 to +31, which in 2's complement is `100000` to `000000` to `011111`, or 6 bits.

The rules of result width are:
* A signed operand has its effective length increased by one (due negative numbers being represented as 2's complement).
* A result is as long as it needs to be for the largest possible positive or negative result.

## Placing statements in domains

Statements are written in the combinatorial domain of a module, or in a sequential domain (clock domain) of a module. The equivalent in Verilog is continuous assignment and clocked assignment.

So if you have a module `m`, you can add a statement that `x` gets the value of `y+1` all the time like:

```python
m.d.comb += x.eq(y+1)
```

On the other hand, if you have a clock domain `sync` that is clocked on the positive edge, then you can add a statement that `x` gets the value of `y+1` on the positive edge of the clock of `sync` like:

```python
m.d.sync += x.eq(y+1)
```

## Adding multiple statements

The `+=` operator for a domain can take one statement, or an array of statements, which can be convenient:

```python
m.d.comb += [
    x.eq(y+1),
    z.eq(w+2),
]
```

## Statements that "override"

If a statement sets the same signal that a previous statement set, then the second set takes precedence. So for example:

```python
m.d.comb += x.eq(y+1)
m.d.comb += x.eq(y+2)
```

In this case, `x` will get `y+2`.
