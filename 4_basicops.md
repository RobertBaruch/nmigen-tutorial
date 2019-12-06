# Basic operations

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
| `\|`     | bitwise or               |
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
(width=5, signed=False)
>>>
```

The same happens when adding two 5-bit 2's complement signals. The range of each operand is -16 to +15, so the range of the result is -32 to 30, which means that adding two 5-bit 2's complement signals yields a 6-bit 2's complement signal. This is an important point to keep in mind, since it will affect comparisons. We will see how to deal with this later.

## Multiplexing signals

`Mux` returns one signal if the condition is true, the other signal otherwise. It is the equivalent of the ternary operator than many languages have, but Python does not:

```python
y.eq(Mux(cond, x1, x2))
```

In this case, if `cond` is true then `y` is set to `x1`, otherwise `x2`. In other languages, this would be `cond ? x1 : x2`.

`Mux` cannot be used on the left-hand side of an assignment.

## Placing statements in domains

Statements are written in the combinatorial domain of a module, or in a sequential domain (clock domain) of a module. The equivalent in Verilog is continuous assignment and clocked assignment.

So if you have a module `m`, you can add a statement that `x` gets the value of `y+1` all the time like this:

```python
m.d.comb += x.eq(y+1)
```

On the other hand, if you have a clock domain `sync` that is clocked on the positive edge, then you can add a statement that `x` gets the value of `y+1` on the positive edge of the clock of `sync` like this:

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

Remember that a signal cannot be set in two different domains so this will result in a driver-driver conflict:

```python
m.d.comb += x.eq(y+1)
m.d.sync += x.eq(y+2)
```
