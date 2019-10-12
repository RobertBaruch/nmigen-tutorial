# Bits and pieces of signals

## Slicing signals

We can use the array indexing operator to extract bits from a signal. For example, given a 16-bit signal `s`, we can get the least significant bit via `s[0]` or the most significant bit via `s[15]`. Just like Python arrays, the bits in a signal are always ordered in one and only one way. This can cause a bit of confusion for those familiar with indexing in HDL. While `s[7:0]` might seem to extract the eight least significant bits of a signal, this is not the way Python works, and _it is Python that we are programming in_. The correct way to extract the eight least significant bits of a signal would be `s[0:8]` or `s[:8]`, and this is the same way we would extract the first eight elements of a Python array. In essence, the first N elements of a signal, when treated like an array of bits, are the N least significant bits of that signal.

