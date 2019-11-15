## What's New in Scala 3

and why you should care

A brief guided tour of some of the highlights of Scala 3, and what they mean for everyday functional programming

--

1.  Enums! they're good again
2. Union and intersection types
3. New keyword considered deprecated
4. Extension methods - no more `implicit class`
5. Given instances, given clauses and more
6. Match types
7. `@alpha` and `@infix` annotations
8. That's not all....

💜

---

## Enums: they're good again!

Finally, a nice enum implementation.

```scala

enum Colour {
  case Red, Green, Blue
}
```

Note: This will define a new `sealed` class Colour, with three values, Red, Green and Blue. The colour values are
members of Colour's companion object. So straight away we can see that we avoid one of the main gripes with enums
in Scala 2, the lack of exhaustiveness checking for match statements.

--

They can be parameterised...

```scala

enum Colour(val rgb: Int) {
  case Red extends Colour(0xFF0000)
  case Green extends Colour(0x00FF00)
  case Blue extends Colour(0x0000FF)

  // ... have user-defined members ...
  def hexCode: String = f"#$rgb%x"
}
```

--

... and have an `ordinal` method defined, associating each enum value with a unique integer.

```scala
scala> val red = Colour.Red
val red: Colour = Red
scala> red.ordinal
val res0: Int = 0
```

--

They can even play nicely with Java:

```scala
enum Colour extends java.lang.Enum[Colour] {
  case Red, Green, Blue
}
```

No more relying on third-party libraries 🎉

Note: The type parameter comes from the Java enum definition, and should be the same as the type of the enum.


---

## Intersection types

The type `A & B` includes all values that are of both type `A` and type `B`.

```scala
trait Reader[T] {
  def read: T
}
trait Writer[T] {
  def write(t: T): Unit
}

def doSomething(x: Reader[String] & Writer[String]): Unit = {
  val name = x.read
  x.write(s"Hello, $name")
}

```

Note: When used on types, the `&` operator creates an intersection type. In this example, `x` here is required to be **both** a Reader and a Writer.

--

If a member appears in both `A` and `B`, its type in `A & B` is the intersection of its types in `A` and `B`.
For example:

```scala
trait A {
  def children: Seq[A]
}

trait B {
  def children: Seq[B]
}

class C extends A with B {
  def children: Seq[C] = ???
}

val x: A & B = new C
val ys: Seq[A] & Seq[B] = x.children
```

Note: We can further simplify the type of `Seq[A] & Seq[B]` to `Seq[A & B]` because Seq is covariant.

---

## Union types

The type `A | B` includes all values of type `A` and all values of type `B`.

```scala
case class Username(name: String)
case class PasswordHash(hash: Hash)

def help(id: Username | PasswordHash) = {
  val user = id match {
    case Username(name) => findUser(name)
    case PasswordHash(hash) => lookUpHash(hash)
  }
  // ...
}
```

Note: In situations where you would previously use an `Either[A, B]`, now you don't have to incur the overhead
of wrapping your function parameter in an object if all you want to do is match on it and do one of two things.
Maybe not quite as useful as an intersection - still cool though

