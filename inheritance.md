# Issues with inheritance

I'm sometimes asked what's so bad about inheritance. Here I'll explain it in the
context of java as that's the OO language I'm most familiar with.

Inheritance tries to solve 2 distinct issues so we will tackle them separately.

## 1. Code reuse

Similarly to functions and modules, inheritance allows for code reuse. Unlike
those hovewer, it fails in some pretty spectacular ways.

### 1.1. Program design

Ideally, we want to organize our code so that understanding any part of it
requires remembering as little things as possible.

Inheritance does not help with that at all. Instead it promotes structuring
our code purely based on shared parts of an implementation or even \*shudders\*
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

public class ListImpl<E> implements List<E> { /* ... */ }

public class ListWithSum extends ListImpl<Integer> {
  // Does this class keep track of total correctly? It depends on implementation
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
account when designing a class, to remember about them when modifying it (this
issue is called fragile base class) and understand them to understand what a
subclass is doing.

## 2. Extensibility

Use of inheritance for extensibility is also questionable. Everything being
extensible by default makes many of these issues worse.

### 2.1. State

Unlike with interfaces, inheritance forces you to inherit state regardless of
whether you need it or not. What if you want to compute a field instead? Or add
laziness? Or make an adapter?

```java
public class Foo<T> {
  private final T foo;

  public Foo(T foo) { this.foo = Objects.requireNonNull(foo); }

  public T getFoo() {
    return this.foo;
  }
}

public class LazyFoo<T> extends Foo<T> {
  public LazyFoo(Supplier<T> supplier) {
    super(/* What do we even put here? */)
  }
}
```

### 2.2. Program design again

Accounting for infinite extensibility isn't always free. While it can be like
that sometimes, it can also make the implementation more complicated if we
are lucky or our api if we are not. One example where it's especially painful
is (de)serialization (assuming we want round trip to always result in object
identical to the one we started with).

Not making things extensible or only allowing small set of variants known
upfront can substantially simplify the design.

### 2.3. Language design

I've seen some people struggle with understanding variance. While it's certainly
possible they lacked good resources / teacher, the point still stands that they
did not find it intuitive despite having experience with OOP.

Language being complicated can be a tradeoff of course, but in my opinion the
benefits here are questionable at best.

### 2.4. Ease of reasoning

We can probably agree that overly generic / abstract code can be harder to
understand and reason about. With inheritance, for every object we either have
to reason about it (and its methods) abstractly, or rely on global reasoning
to trace actual types.

## Bonus: Another way to think about inheritance

Inheritance is a bit like version control. You start with a base class, then
you apply patches onto it. Except you can't view the current state of the code,
just the individual commits. While this comparison obviously exaggerates the
difficulty of working with inheritance, I do think it's useful to demonstrate
some of the issues inherent to it.
