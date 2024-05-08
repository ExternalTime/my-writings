Compiletime zig is dependently typed. While dynamic checking prevents us from
doing some of the more interesting things, it shows that dependent types can do
perfectly fine in "mainsteram" languages.

Dependent types give a lot of flexibility without sacrificing type safety.
They allow, for example, to implement arrays and tuples as a library in a
straightforward way:

```zig
fn MyArray(comptime n: usize, comptime T: type) type {
    if (0 < n) {
        return struct {
            head: T,
            tail: MyArray(n - 1, T),
        };
    } else {
        return struct {};
    }
}

// Could make this parametrized by normal list instead.
fn MyTuple(comptime n: usize, comptime ts: MyArray(n, type)) type {
    if (0 < n) {
        return struct {
            head: ts.head,
            tail: MyTuple(n - 1, ts.tail),
        };
    } else {
        return struct {};
    }
}
```
