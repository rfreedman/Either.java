Either.java
===========

[![Build Status](https://travis-ci.org/spencerwi/Either.java.svg?branch=master)](https://travis-ci.org/spencerwi/Either.java)

A right-biased implementation of Haskell's `Either a b` for Java, using Java 8 for mapping/folding and type inference.

Wait, what?
------------

Yes, polymorphism is great in Java. `Animal.speak()`, `Dog.speak()`, `Cat.speak()`, and so on.

But sometimes you want to be REALLY EXPLICIT about the possible types of objects you're dealing with, either (ha ha)
 because they don't logically make sense in the same inheritance structure or because you want to limit your
 possible cases to exactly two classes.

For example, let's say I'm dealing with my code and some third-party library that still uses `java.util.Date`. Terrible,
 I know, but there are always *those people*. I don't want to further propagate their usage of a terrible deprecated
 class, so I use `java.time.LocalDateTime` in my code.

Except great, now I've gotta deal with grabbing a `List<LocalDateTime>` from *these* and a `List<Date>` from *those* and
then do a bunch of gnarly conversion everywhere.

I could instead just get a `List<Either<Date,LocalDatetime>>`. This tells me (and the compiler) that I'm dealing with a
bunch of things that are *either* a Date (yuck) or a LocalDateTime.

I can even nicely convert them all into LocalDateTimes:

```java

List<Either<Date, LocalDateTime>> birthdays = allThePeople.stream()
                                                          .map(person -> Either.<Date,LocalDateTime>either(this::getDeprecatedDOBFromPerson, this::getNiceNewDOBFromPersonIfAvailable)
                                                          .collect(Collectors.toList());

List<LocalDateTime> birthdays.stream()
                             .map(eitherDateOrLocalDateTime.fold(
                                (Date deprecatedDOB)   -> myFunctionToConvertDateToLocalDateTime(deprecatedDOB),
                                (LocalDatetime newDOB) -> newDOB
                             )).collect(Collectors.toList());
```

Boom. Now you have a list of LocalDateTimes, and it's explicit from the code that some of the people had deprecated
 Dates, while others had nice, shiny new LocalDateTimes. Even the compiler can tell!


Other, more common use-cases include handling errors using `Either<SomeKindOfException, SuccessfulResultClass>`.
The convention in the Haskell world (from which I totally "borrowed" the Either) is that an Either gives you
"either the Right answer or whatever's Left" -- that is, errors on the left, expected output on the right.

For this reason, this Either is right-biased; if you give it `Either.either(()->42, ()->"Hello, World!");`, you'll get a
`Right` containing `"Hello, World!"`, not a `Left` containing `42`. I swear, it's not a political thing; there just needs to be a predictable rule-of-thumb for how to handle it when the Either gets *both* a left value *and* a right value (after all, it's called an *Either*, not a *Both*).

Because this errors-to-the-left, results-to-the-right idiom is so common, this library also includes a `Result<T>` class, 
which instead of being `Left<L,R>` or `Right<L,R>` is `Err<T>` or `Ok<T>`. Instead of `Either.either(() -> "left",  () -> 42)`,
the constructor method you'll want is `Result.attempt(() -> someMethodThatMightThrowAnException())`. You can even chain
a series of possibly-failing functions using `map`:

```java
Result<C> = Result.attempt(() -> someOperationThatMightFailOrReturnA())
                  .map(a -> someOtherOperationThatMightFailOReturnB(a))
                  .map(b -> someThirdOperationThatMightFailOrReturnC(b));
```

So what else can it do?
-----------------------

Wanna see more? Check out the unit tests for a run-down of how Eithers behave. If those tests aren't descriptive enough,
 or you think they should behave differently, open a Github issue! I built this because it didn't look like anyone else
 had built it for Java yet, and I may have lost something in the translation. I'm totally open for feedback.


Cool! How do I get it for my project?
-------------------------------------

Simple! It's in Maven Central, so just add this to your pomfile (or the equivalent for gradle/sbt/lein/whatever):

```
<dependency>
    <groupId>com.spencerwi</groupId>
    <artifactId>Either.java</artifactId>
    <version>2.0.0</version>
</dependency>
```
