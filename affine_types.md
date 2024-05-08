# How affine types make programming easier

Imagine that you want to write a java function that takes 2 lists and appends
all elements from one to the other. Implementing it is trivial.

```java
public <T> void addAll(List<T> target, List<T> source) {
  for (var item : source) {
    target.add(item);
  }
}
```

Except that it's wrong. We could do it slightly differently.

```java
public <T> void addAll(List<T> target, List<T> source) {
  for (int i = 0; i < source.size(); i++) {
    target.add(source.get(i));
  }
}
```

And this one's bugged too. What's more, while both functions are wrong for the
same reason, they behave differently. Can you figure out what's going on?

It's simple. Just pass the same non-empty list as both arguments.
For standard library types at least, the first implementation throws
`ConcurrentModificationException`, while the other one keeps duplicating
elements in the list until it runs out of memory.

This happens because in presence of mutability functions behave differently
based on how mutated things alias.

We can solve it in one of two ways. Either by avoiding mutation, which is what
functional languages generally do:

```ocaml
let rec addAll target source =
  match source with
  | [] -> target
  (* We don't modify anything. Instead, we construct a new list. *)
  | head :: tail -> head :: (addAll target tail);;
```

Or by restricting aliasing:

```rust
// If we tried to pass the same `Vec` as both target and source borrow checker
// would stop us.
fn addAll(target: &mut Vec<T>, source: Vec<T>) {
  for item in source {
    target.push(item);
  }
}
```

Both of these approaches free us to focus on things that are actually important.
