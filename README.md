[//]: # (This file was autogenerated using `zio-sbt-website` plugin via `sbt generateReadme` command.)
[//]: # (So please do not edit it manually. Instead, change "docs/index.md" file or sbt setting keys)
[//]: # (e.g. "readmeDocumentation" and "readmeSupport".)


# ZIO JSON

[ZIO Json](https://github.com/zio/zio-json) is a fast and secure JSON library with tight ZIO integration.

Project Stage | CI | Release | Snapshot | Discord | Github |
--------------|----|---------|----------|---------|--------|
[![Production Ready](https://img.shields.io/badge/Project%20Stage-Production%20Ready-brightgreen.svg)](https://github.com/zio/zio/wiki/Project-Stages)        |![CI Badge](https://github.com/zio/zio-json/workflows/CI/badge.svg) |[![Sonatype Releases](https://img.shields.io/nexus/r/https/oss.sonatype.org/dev.zio/zio-json_2.12.svg)](https://oss.sonatype.org/content/repositories/releases/dev/zio/zio-json_2.12/) |[![Sonatype Snapshots](https://img.shields.io/nexus/s/https/oss.sonatype.org/dev.zio/zio-json_2.12.svg)](https://oss.sonatype.org/content/repositories/snapshots/dev/zio/zio-json_2.12/) |[![Chat on Discord!](https://img.shields.io/discord/629491597070827530?logo=discord)](https://discord.gg/2ccFBr4) |[![ZIO JSON](https://img.shields.io/github/stars/zio/zio-json?style=social)](https://github.com/zio/zio-json) |


## Introduction

The goal of this project is to create the best all-round JSON library for Scala:

- **Performance** to handle more requests per second than the incumbents, i.e. reduced operational costs.
- **Security** to mitigate against adversarial JSON payloads that threaten the capacity of the server.
- **Fast Compilation** no shapeless, no type astronautics.
- **Future-Proof**, prepared for Scala 3 and next-generation Java.
- **Simple** small codebase, concise documentation that covers everything.
- **Helpful errors** are readable by humans and machines.
- **ZIO Integration** so nothing more is required.

## Installation

In order to use this library, we need to add the following line in our `build.sbt` file:

```scala
libraryDependencies += "dev.zio" %% "zio-json" % "0.0.0+487-bfc3bd28-SNAPSHOT"
```

## Example

Let's try a simple example of encoding and decoding JSON using ZIO JSON.

All the following code snippets assume that the following imports have been declared

```scala
import zio.json._
```

Say we want to be able to read some JSON like

```json
{"curvature":0.5}
```

into a Scala `case class`

```scala
case class Banana(curvature: Double)
```

To do this, we create an *instance* of the `JsonDecoder` typeclass for `Banana` using the `zio-json` code generator. It is best practice to put it on the companion of `Banana`, like so

```scala
object Banana {
  implicit val decoder: JsonDecoder[Banana] = DeriveJsonDecoder.gen[Banana]
}
```

_Note: If you’re using Scala 3 and your case class is defining default parameters, `-Yretain-trees` needs to be added to `scalacOptions`._

Now we can parse JSON into our object

```
scala> """{"curvature":0.5}""".fromJson[Banana]
val res: Either[String, Banana] = Right(Banana(0.5))
```

Likewise, to produce JSON from our data we define a `JsonEncoder`

```scala
object Banana {
  ...
  implicit val encoder: JsonEncoder[Banana] = DeriveJsonEncoder.gen[Banana]
}

scala> Banana(0.5).toJson
val res: String = {"curvature":0.5}

scala> Banana(0.5).toJsonPretty
val res: String =
{
  "curvature" : 0.5
}
```

And bad JSON will produce an error in `jq` syntax with an additional piece of contextual information (in parentheses)

```
scala> """{"curvature": womp}""".fromJson[Banana]
val res: Either[String, Banana] = Left(.curvature(expected a number, got w))
```

Say we extend our data model to include more data types

```scala
sealed trait Fruit
case class Banana(curvature: Double) extends Fruit
case class Apple (poison: Boolean)   extends Fruit
```

we can generate the encoder and decoder for the entire `sealed` family

```scala
object Fruit {
  implicit val decoder: JsonDecoder[Fruit] = DeriveJsonDecoder.gen[Fruit]
  implicit val encoder: JsonEncoder[Fruit] = DeriveJsonEncoder.gen[Fruit]
}
```

allowing us to load the fruit based on a single field type tag in the JSON

```
scala> """{"Banana":{"curvature":0.5}}""".fromJson[Fruit]
val res: Either[String, Fruit] = Right(Banana(0.5))

scala> """{"Apple":{"poison":false}}""".fromJson[Fruit]
val res: Either[String, Fruit] = Right(Apple(false))
```

Almost all of the standard library data types are supported as fields on the case class, and it is easy to add support if one is missing.

```scala
import zio.json._

sealed trait Fruit                   extends Product with Serializable
case class Banana(curvature: Double) extends Fruit
case class Apple(poison: Boolean)    extends Fruit

object Fruit {
  implicit val decoder: JsonDecoder[Fruit] =
    DeriveJsonDecoder.gen[Fruit]

  implicit val encoder: JsonEncoder[Fruit] =
    DeriveJsonEncoder.gen[Fruit]
}

val json1         = """{ "Banana":{ "curvature":0.5 }}"""
val json2         = """{ "Apple": { "poison": false }}"""
val malformedJson = """{ "Banana":{ "curvature": true }}"""

json1.fromJson[Fruit]
json2.fromJson[Fruit]
malformedJson.fromJson[Fruit]

List(Apple(false), Banana(0.4)).toJsonPretty
```

# How

Extreme **performance** is achieved by decoding JSON directly from the input source into business objects (inspired by [plokhotnyuk](https://github.com/plokhotnyuk/jsoniter-scala)). Although not a requirement, the latest advances in [Java Loom](https://wiki.openjdk.java.net/display/loom/Main) can be used to support arbitrarily large payloads with near-zero overhead.

Best in class **security** is achieved with an aggressive *early exit* strategy that avoids costly stack traces, even when parsing malformed numbers. Malicious (and badly formed) payloads are rejected before finishing reading.

**Fast compilation** and **future-proofing** is possible thanks to [Magnolia](https://propensive.com/opensource/magnolia/) which allows us to generate boilerplate in a way that will survive the exodus to Scala 3. `zio-json` is internally implemented using a [`java.io.Reader`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/io/Reader.html) / [`java.io.Writer`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/io/Writer.html)-like interface, which is making a comeback to center stage in Loom.

**Simplicity** is achieved by using well-known software patterns and avoiding bloat. The only requirement to use this library is to know about Scala's encoding of typeclasses, described in [Functional Programming for Mortals](https://leanpub.com/fpmortals/read#leanpub-auto-functionality).

**Helpful errors** are produced in the form of a [`jq`](https://stedolan.github.io/jq/) query, with a note about what went wrong, pointing to the exact part of the payload that failed to parse.

## Documentation

Learn more on the [ZIO JSON homepage](https://zio.dev/zio-json/)!

## Contributing

For the general guidelines, see ZIO [contributor's guide](https://zio.dev/about/contributing).

## Code of Conduct

See the [Code of Conduct](https://zio.dev/about/code-of-conduct)

## Support

Come chat with us on [![Badge-Discord]][Link-Discord].

[Badge-Discord]: https://img.shields.io/discord/629491597070827530?logo=discord "chat on discord"
[Link-Discord]: https://discord.gg/2ccFBr4 "Discord"

## License

[License](LICENSE)

## Acknowledgement

- Uses [JsonTestSuite](https://github.com/nst/JSONTestSuite) to test parsing. (c) 2016 Nicolas Seriot)

- Uses [YourKit Java Profiler](https://www.yourkit.com/java/profiler/) for performance optimisation. ![YourKit Logo](https://www.yourkit.com/images/yklogo.png)

