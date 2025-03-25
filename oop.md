# Issues with traditional OOP

Here is a non-exhaustive list of issues I find with object oriented programming
with focus on java.

## 1. Everything is an object

While abstracting over behaviour can be useful (especially for interacting with
environment), it's not something we should be using *everywhere*. There is no
need to hide the implementation of a type that doesn't even escape the module
its defined in. Getters and setters hide nothing while introducing noise into
the codebase. Premature abstraction doesn't help anyone.

There is also an issue with types for which we know all the possible variants
upfront. How do we define them in a way that will allow us to add functionality
without modifying the type definition? We can just ignore the issue and use
downcasting which will be fun when someone gets the bright idea to extend our
interface or use scott encoding, which is *very* unergonomic to define.

Luckily for us, if we are working with relatively modern version of java we can
just use sum types:

```java
public sealed interface JsonElement {
  public record Null() implements JsonElement {}
  public record Bool(boolean v) implements JsonElement {}
  public record Num(Number v) implements JsonElement {}
  public record Str(String v) implements JsonEleemnt {}
  public record Arr(List<JsonElement> v) implements JsonElement {}
  public record Obj(Map<String, JsonElement> v) implements JsonElement {}
}
```

## 2. Subtyping and dynamic dispatch

While subtyping can be useful, it increases complexity when interacting with
other language features. We need variance or similar to account for parametric
polymorphism. Reflection gets more features to account for class hierarchy.

It's often simply not what we want or need. With dynamic dispatch if we want
our abstraction to include ways of constructing a type, we need to define an
entirely separate object just for that. Dealing with objects of potentially
different types (where you don't know all the possible variants up-front)
also needlesly complicates basic operations on them like comparison or
deserialization.

This is less important but dynamic dispatch also makes static analysis harder
and inhibits optimization.

## 3. Inheritance

Inheritance is a code reuse mechanism. What it doesn't do is help with creating
abstractions. Allowing a class to be extended (which is of course the default)
inevitably leaks implementation details (which as we know with enough users
someone will depend on):

```java
// This is ofc simplified
public interface List<E> {
  public void add(E v);
  public void addAll(List<E> other);
}

public class ListImpl<E> implements List<E> { /* ... */ }

public class ListWithSum extends ListImpl<Integer> {
  private int total = 0;

  public void add(Integer n) {
    super.add(n);
    this.total += n;
  }
}
```

Is this `ListWithSum` implemented correctly? It depends entirely on the
implementation of `ListImpl`!
