## What's New in Scala 3

and why you should care

A brief guided tour of some of the highlights of Scala 3, and what they mean for everyday functional programming

Note: Hi everyone, my name's Frankie. I'm going to talk to you today about some of my favourite new features
in Scala 3, and how they will hopefully improve daily life.

--

1.  Enums! they're good again
2. New keyword considered deprecated
3. Union and intersection types
4. Given instances, given clauses and more
5. Extension methods - no more `implicit class`
6. `@alpha` and `@infix` annotations
7. That's not all....

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

### New keyword optional

The `new` keyword can be omitted from almost all instance creations - it's only needed to disambiguate
an apply method.


```scala
class Foo(bar: Int) {
  def this() = this(0)
}

Foo(123) // same as new Foo(123)
Foo()    // same as new Foo()

```

Note: So now you won't need to define a case class just to get nice constructor calls - something I've definitely
been guilty of

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
Maybe not quite as useful as an intersection? - still cool though

---

## Extension methods

Replaces implicit classes


```scala
object Extensions {
  def (s: String) times (n: Int): String = {
    if (n <= 0) "" else s.times(n - 1) ++ s
  }
}

import Extensions._

"hello".times(3) // "hellohellohello"

```

Note: A replacement for implicit classes, this new syntax for extension methods uses this kind of infix notation
that we see here. Makes it easier to monkey-patch new functionality onto an existing class, without extending
it - as well as being less cryptic, also means less enforcement of object-oriented approaches

---

## Given instances

* New keyword `given`
* Expresses intent instead of mechanism
* Gives a nice syntax for typeclasses

```scala
trait Ord[T] {
  def (x: T) compareTo (y: T): Int
  def (x: T) < (y: T): Boolean = x.compareTo(y) < 0
}

given IntOrd: Ord[Int] {
  def (x: Int) compareTo (y: Int): Int = {
    if (x < y) -1 else if (x > y) 1 else 0
  }
}

def maximum[T](xs: List[T])(given ord: Ord[T]): T = {
  xs.reduce((x, y) => if (x < y) y else x)
}
```

Note: Given instances define "canonical" values of certain types that serve for synthesizing arguements to given
clauses. Although in this example we've given a name to our given instance, IntOrd, this is optional and
it's perfectly fine to declare an anonymous given instance.

--

## Typeclasses using given instances

```scala
trait Semigroup[T] {
  def (x: T) combine (y: T): T
}
trait Monoid[T] extends Semigroup[T] {
  def unit: T
}

given Monoid[String] {
  def (x: String) combine (y: String): String = x.concat(y)
  def unit: String = ""
}

def sum[T](xs: List[T])(given Monoid[T]): T = {
  xs.foldLeft(summon[Monoid[T]].unit)(_.combine(_))
}

```

Note: Note the use of `summon[Monoid[T]]` in the first argument to foldLeft. Similar to `implicitly` in current
usage, all that does is simply returns the given instance for that particular type, in whatever scope it's
invoked from.

--

## Implicit conversions

Implicit conversions have been criticised for being too easy to implement compared to how dangerous they are.
`implicit` as a modifier will go away completely, to be replaced by `given` instances of `scala.Conversion`.

```scala
// in package `scala`
abstract class Conversion[-T, +U] extends (T => U)

// usage
given Conversion[String, Token] {
  def apply(str: String): Token = new Keyword(str)
}

// or, more concisely:
given Conversion[String, Token] = new Keyword(_)

```

--

## Why given?

Implicits are one of Scala's most powerful features, but also quite controversial.

Given instances and given clauses are a simpler, safer alternative:

- emphasize **intent** over mechanism
- conceptually more accessible
- discourage abuse

Note: The new approach represents many improvements over current implicits. Names of given instances don't
matter, and indeed they can be completely anonymous. Because nesting of declarations is significant, it's
easier to achieve local coherence, and avoid problems with shadowing. Restricted implicit scope means fewer
surprises, and fewer accidental conversions; there's also been a bit of work on making the error messages nicer.

---

## Rules for operators

An `@alpha` annotation on a method defines an alternate name for the implementation of that method.

```scala
class Container[T](elems: List[T]) {

  @alpha("concat")
  def ++(other: Container[T]): Container[T] = {
    Container(elems ++ other.elems)
  }
}

```

An `@alpha` annotation will be mandatory for symbolic method names.

Note: Here, the `++` operation is implemented (in bytecode or native code) under the name `concat`, and can be
invoked from Java or other languages by that name. Any application of the method from Scala has to use `++`.

--

An `@infix` annotation on a method definition allows using the method as an infix operation.

```scala
trait MultiSet[T] {
  @infix
  def union(other: MultiSet[T]): MultiSet[T]

  def difference(other: MultiSet[T]): MultiSet[T]

  @alpha("intersection")
  def * (other: MultiSet[T]): MultiSet[T]
}

val s1, s2: MultiSet[Int]
```


```scala
s1 union s2  // OK
s1.union(s2) // also OK
```
<!-- .element: class="fragment" data-fragment-index="1" -->


```scala
s1.difference(s2)  // OK
s1 `difference` s2 // OK
s1 difference s2   // gives a deprecation warning
```
<!-- .element: class="fragment" data-fragment-index="2" -->

```scala
s1 * s2  // OK
s1.*(s2) // OK... but why would you want to
```
<!-- .element: class="fragment" data-fragment-index="3" -->

Note: The purpose of the @infix annotation is to achieve consistency across a code base in how a method
is applied. The idea is that the author of a method decides whether that method should be applied as
an infix operator or in a regular application. Use sites then implement that decision consistently.

---

## Conclusion

Scala 3: a land of contrasts

Many language changes, including feature removals and deprecations

New constructs aimed to improve user experience and learning

Scala 3.0 will keep most constructs of 2.13, alongside the new ones, although some will be deprecated and phased out

End result: a more compact and regular language, with fewer pitfalls and mysteries

💜

