Realized recently that the way java added sum types also allows us to write
GADTs. As expected there is nothing stopping us from defining GADTs and
predictably pattern matching on anything non-trivial doesn't work. Oh well.

```java
// Type of type equality. Non-null object of `TypeEq<T1, T2>` indicates that `T1` == `T2`.
public sealed interface TypeEq<T1, T2> {
  public final class Witness<T> implements TypeEq<T, T> {}

  // Equality is transitive
  // `forall a b c. (a = b /\ b = c) -> a = c`
  default <T3> TypeEq<T1, T3> trans(TypeEq<T2, T3> eq2) {
    return switch (this) {
      case Witness<T2> w1 -> switch (eq2) {
      //   ^^^^^^^^^^^^^^ TypeEq<T1,T2> cannot be safely cast to Witness<T2>
        case Witness<T2> w2 -> new Witness<>();
        //                                ^^ incompatible types: bad type in switch expression
        //   ^^^^^^^^^^^^^^ TypeEq<T2,T3> cannot be safely cast to Witness<T2>
      };
    };
  }
}
```
