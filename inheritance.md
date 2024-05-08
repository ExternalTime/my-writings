# Issues with inheritance

I'm sometimes asked what's so bad about inheritance. Here I'll explain it in the
context of java as that's the OO language I'm most familiar with.

Inheritance tries to solve 2 distinct issues so we will tackle them separately.

## 1. Code reuse

TODO: add something about functions, modules and composition.

### 1.1. Design

Ideally, we want to organize our code so that understanding any part of it
requires remembering as little things as possible.

Inheritance does not help with that at all. Instead it promotes structuring
our code purely based on shared parts of an implementation or even *shudders*
ontological relationships.

### 1.2. Leaking implementation details

Inheritance leaks information about whether and when methods in a superclass
call each other:

```java
// This is ofc simplified
public interface List<E> {
  public void add(E v);
  public void addAll(List<E> other);
}

public class ListImpl<E> implements List<E> { /* ... */}

public class ListWithSum extends ListImpl<Integer> {
  // Does this class keep track of total correctly? Depends on implementation
  // of `ListImpl`.
  private int total = 0;

  public void add(Integer n) {
    super.add(n);
    this.total += n;
  }
}
```

While this might seem inocous at first, it's the biggest issue with inheritance
and it's only made worse by it being ignored in education.

These details being part of the public api means that we need to take them into
account when designing a class, to remember about them when modifying a class
(this issue is called fragile base class) and understand them to understand what
a subclass is doing.

## 2. Extensibility

TODO: Add something about interfaces, ADTs and existential types.

### 2.1. Not everything has to be extensible

Accounting for infinite extensibility isn't always free. While it can be like
that sometimes, it's also possible for it to require changing implementation or
even the design. One example where it's especially painful is (de)serialization.

Not making things extensible or only allowing small set of variants known
upfront can substantially simplify the design.

### 2.2. Inheriting state

Unlike with interfaces, inheritance forces you to inherit state regardless of
whether you need it or not. What if you want to compute a field instead? Or add
laziness? Or make an adapter?

```java
public class Foo<T> {
  private final T foo;

  public Foo(T foo) { this.foo = foo; }

  public T getFoo() {
    return this.foo;
  }
}

public class LazyFoo<T> extends Foo<T> {
  // ...
  public LazyFoo(Supplier<T> supplier) {
    super(null);
    // ...
  }

  @Override public T getFoo() { /* ... */ }
}
```

While in this case the workaround is easy, it could become more complicated if
the superclass enforced the value to not be null.

### 2.3. Makes langauge more complicated

I've seen some people struggle with understanding variance. While it's certainly
possible they lacked good resources / teacher, the point still stands that they
did not find it intuitive despite having experience with OOP.

Language being complicated can be a tradeoff of course, but in my opinion the
benefits here are questionable at best.
