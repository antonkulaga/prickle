#Prickle

Prickle is a library for serializing (pickling) object graphs between Scala and Scala.js. It is based upon scala-js-pickling, but adds three key improvements:

* Better support for class hierarchies
* Support for sharing and cycles in the serialized object graph
* Pickles to/from Strings, so no platform-specific JSON dependencies are required

Currently, prickle supports automatic, reflection-free recursive pickling of
* Case classes
* Case objects
* Seqs, Sets and Maps
* Primitives


##[Example](https://github.com/benhutchison/prickle/blob/master/example/src/main/scala/Example.scala)

```scala
import prickle._

sealed trait Fruit
case class Apple(isJuicy: Boolean) extends Fruit
case class Lemon(sourness: Double) extends Fruit
case class FruitSalad(components: Seq[Fruit]) extends Fruit
case object TheDurian extends Fruit

object Example extends App {

  println("\n1. No preparation is needed to pickle or unpickle values whose static type is exactly known:")

  val apples = Seq(Apple(true), Apple(false))
  val pickledApples = Pickle.intoString(apples)
  val rehydratedApples = Unpickle[Seq[Apple]].fromString(pickledApples)


  println(s"A bunch of Apples: ${apples}")
  println(s"Pickled apples: ${pickledApples}")
  println(s"Rehydrated apples: ${rehydratedApples}\n")


  println("2. To pickle a class hierarchy (aka 'Sum Type'), create a CompositePickler and enumerate the concrete types")

  //implict defs/vals should have an explicitly declared type to work properly
  implicit val fruitPickler: PicklerPair[Fruit] = CompositePickler[Fruit].
    concreteType[Apple].concreteType[Lemon].concreteType[FruitSalad].concreteType[TheDurian.type]

  val sourLemon = Lemon(sourness = 100.0)
  //fruitSalad's concrete type has been forgotten, replaced by more general supertype
  val fruitSalad: Fruit = FruitSalad(Seq(Apple(true), sourLemon, sourLemon, TheDurian))

  val fruitPickles = Pickle.intoString(fruitSalad)

  val rehydratedSalad = Unpickle[Fruit].fromString(fruitPickles)

  println(s"Notice how the fruit salad has multiple references to the same object 'sourLemon':\n${fruitSalad}")

  println(s"In the JSON, 2nd and subsequent occurences of an object are replaced by refs:\n${fruitPickles}")

  println(s"The rehydrated object graph doesnt contain duplicated lemons:\n${rehydratedSalad}\n")
}
```

##Basic Concepts

Like scala-pickling and scala-js-pickling, prickle uses implicit Pickler[T] and Unpickler[T] type-classes to transform values of type T. For case classes, these type classes will be automatically generated via an implicit macro, if they are not already available in implicit scope. This is a recursive process, so that picklers & unpicklers for each field of type T will also be resolved and possibly generated.



##Support for Class Hierarchies & Sum-types

##Support for Shared objects

##Pickling to/from String
