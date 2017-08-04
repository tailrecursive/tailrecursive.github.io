---
title: "Introducing: mockito-sweetener for ScalaTest"
categories: scala
author: Johan Östling
---
After having used [ScalaTest](http://www.scalatest.org/) and [Mockito](http://mockito.org/)
together for the last year, I finally got fed up with using Mockito's Java API for almost everything:

```scala
class TheTest extends FunSpec with MockitoSugar {
  import org.mockito.Mockito.{when, verify}
  // alias eq because of conflict with Scala's eq
  import org.mockito.Matchers.{isA, eq => meq} 

  test("something") {
    val mock = mock[TheThing] // This part, from MockitoSugar is nice enough

    when(mock.someMethod(meq("foo"))).thenReturn("bar")

    verify(mock).anotherMethod(isA(classOf[String]))    
  }  
}
```

It's just not Scalaish enough, not to mention the annoying quirks:

1. need to alias `Matchers.eq` when imported, or type out `Matchers.eq("foo")` in code
2. `any(classOf[String])` will actually _not_ match on the argument type, which pretty much
  nobody knows about

> As of Mockito 2.0 the second point is no longer valid!

So after reading the Specs2 documentation, and seeing how nice their Mockito helper was, I
felt I had to do the same for our ScalaTests.

Hence was born the humble [mockito-sweetener](https://github.com/jostly/mockito-sweetener):

```scala
val m = mock[List[Int]]
m.length returns 17

assert(m.length === 17)

m.contains(any[Int]) returns true

assert(m.contains(5) === true)
```

Set up an expectation (`when`) by calling the method and adding `returns(whatever)`.
Use `any[<type>]` as it makes sense to do, with matching on argument type.

```scala
val m = mock[List[Int]]
m.length
m.contains(4)
m.contains(6)

there was one(m).length
there was zero(m).head
there was one(m).contains(lessThan(5))
there were two(m).contains(*)
```

That's right, `*` matches any argument, regardless of type. Why make it so? Because it was possible.

### Conclusion

The library is field-tested, and works. Right now it has very little documentation,
but on the other hand, the source is pretty short, so it should be easy to figure out
until I get around to writing up the documentation.

Do [try it out](https://github.com/tailrecursive/mockito-sweetener), and don't be shy about
opening issues on the project regarding features / bugs / whatever.
