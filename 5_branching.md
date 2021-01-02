# Branching

Just as in the standard HDLs, it is possible to use conditional logic.

## If-Elif-Else

You cannot use the standard Python `if-elif-else` statements to create statements. Instead, we use the following `with` constructs:

```python
with m.If(condition1):
    m.d.comb += statements1
with m.Elif(condition2):
    m.d.comb += statements2
with m.Else():
    m.d.comb += statements3
```

If you use regular Python `if-elif-else`, then those will be evaluated during _generation_ of the logic, not within the logic itself. This can be useful if you want a flag to cause different logic to be generated, and this is a good use of the `platform` string passed to `elaborate()`. So for example:

```python
if (platform == "this"):
    m.d.comb += statement1
else:
    m.d.comb += statement2
```

If `platform` is `"this"` then only `statement1` will appear in the generated hardware, otherwise only `statement2` will appear.

### Conditions

The conditions in `If-Elif-Else` are comparisons, for example `a == 1` or `(a >= b) & (a <= c)`. Note that in this latter example, we used parentheses around each term. Each comparison, in essence, becomes a one-bit signal, and `&` is a _bit-wise operator_, not the logical `and`. When in doubt, just use parentheses.

If you have a signal with more than one bit and use it as the condition, as in `with m.If(a):`, then the condition will be true if any bit in `a` is 1.

## Switch-Case-Default

You can use `Switch-Case-Default` just as in standard HDLs using the following `with` constructs:

```python
with m.Switch(expression):
    with m.Case(value1):
        statements1
    with m.Case(value2):
        statements2
    with m.Default():
        statements3
```

A `Case` can also have multiple values:

```python
with m.Switch(expression):
    with m.Case(value1, value2):
        statements1
    with m.Case(value3, value4, value5, value6):
        statements2
```

You can leave out the `Default` case if you don't want to change any signals in the default case:

```python
m.d.comb += x.eq(1)
with m.Switch(y):
    with m.Case(0, 1, 2):
        m.d.comb += x.eq(2)
```

In the above example, if `y` is 0, 1, or 2, then `x` is assigned 2. Otherwise `x` retains its value of 1. Recall the section on overriding statements.

## Bit patterns

The way to specify matching a pattern in a `Case` is with a Python string of binary digits. For example, `"0011101011"`. A don't-care bit is specified using a dash, so for example `"001---101"`. The number of bits in the string must exactly match the number of bits in the expression it is being compared to.

A `Value` has a method `matches` which acts like a `Switch-Case`. For example:

```python
with m.If(a.matches("11---", 3, b)):
    statement1
```

is the same as:

```python
with m.Switch(a):
    with m.Case("11---", 3, b):
        statement1
```
