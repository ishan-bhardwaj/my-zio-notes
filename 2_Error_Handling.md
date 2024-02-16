# Error Handling #

- Transforming error channel into something else -
```
val failedZIO = ZIO.fail(new RuntimeException("Error!"))
failedZIO.mapError(_.getMessage)
```

- Attempt - run an effect that might throw an exception
```
val anAttempt: Task[Int] = ZIO.attempt {
    val string: String = null
    string.length
}
```

## Effectfully catch errors ##
```
val catchAllError: ZIO[Any, Nothing, Any] = anAttempt.catchAll(e => ZIO.succeed("Returning default value because $e"))
val catchSelectiveError: ZIO[Any, Throwable, Any] = anAttempt.catchSome {   // partial function
    case e: RuntimeException => ZIO.succeed(s"Ignoring runtime exceptions: $e")
    case _ => ZIO.succeed("Ignoring everything else")
}
```

## Chaining effects ##
```
val aBetterAttempt: ZIO[Any, Nothing, Int] = anAttempt.orElse(ZIO.succeed(45))

// Handling both success and failure
val handleBoth: URIO[Any, String] = anAttempt.fold(ex => s"Something bad happened: $ex", value => s"Length of the string: $value")

// Effectful fold
val handleBothZIO: URIO[Any, String] = anAttempt.fold(ex => ZIO.succeed(s"Something bad happened: $ex"), value => ZIO.succeed(s"Length of the string: $value"))
```

## Conversions between Option/Try/Either to ZIO
```
// Try -> ZIO
val aTryToZIO: Task[Int] = ZIO.fromTry(Try(42 / 0))

// Either -> ZIO
val anEither: Either[Int, String] = Right("Success")
val anEitherToZIO: IO[Int, String] = ZIO.fromEither(anEither)

// ZIO -> 
val anAttempt: ZIO[Any, Throwable, Int] = ZIO.attempt(42)
val eitherZIO: ZIO[Any, Nothing, Either[Throwable, Int]] = anAttempt.either

// Reverse: ZIO with Either as the value channel -> ZIO with left side of either as error channel
val anAttempt_v2: ZIO[Any, Throwable, Int] = eitherZIO.absolve

// Option -> ZIO
val anOption: ZIO[Any, Option[Nothing], Int] = ZIO.fromOption(Some(42))
```