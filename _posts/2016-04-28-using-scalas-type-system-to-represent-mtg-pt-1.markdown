---
layout: post
title: "Using Scala's Type System to represent Magic: The Gathering Pt. 1"
date: 2016-04-28 02:03:00 -0700
tags: experimental,mtg,scala
---
I've been working with Scala a lot as of late, on a lot of personal projects. I've decided I need to get more
out to show the world. At a loss of what to do, this kind of came out of nowhere. I've always been a huge fan of Magic,
and last night while observing card and creature types, I got to thinking. "Hey... I can probably represent this and
their rules using Scala and it's type system."

As an aside: functional programming has changed the way I look at programming and has (in my opinion) turned me into
a more robust engineer. As functional programming uses a few different concepts heavily, it has also solidified my
understanding and confidence with all sorts of concepts such as recursion and immutability-focused ("pure") functions.
I'll write more about this soon.

Anyways... Scala's type system is rather verbose and allows for one to express all sorts of concepts rather abstractly.
I am going to be using a few key concepts for this project: [monads], [first-class functions], implicit [parameters] and
[classes], [typeclasses], and [structural subtyping] (also known as duck typing). I am also considering using trait
mix-in composition to represent concepts such as a card's colour, but I'm not sure if this is as viable as simple using
case objects.

All source code for the project can be found at {% include icon-github.html %}[github].

For now, let's go over some of the basic structures. Here is the basic layout for a card:

<figure>
<figcaption>File: Card.scala</figcaption>
{% highlight scala %}
trait Card {
  val name: String
  val flavourText: Option[String]
  val cardSetInfo: CardSetInfo
  val artist: String

  val manaCost: Option[ManaCost]
  val colours: Seq[Colour]
  val types: Seq[CardSubTypes]
  val power: Int
  val toughness: Int

  def convertedManaCost: Int = manaCost match {
    case Some(m) => m.convertedManaCost
    case None => 0
  }
}
{% endhighlight %}
</figure>

That's a lot of things! It's ok, they're all rather simple. _name_, _artist_, _power_, and _toughness_ are all rather
straight-forward. Our card's _flavourText_ is represented with an Option\[String\] as not all cards will contain
flavour text. However, our _cardSetInfo_, _colours_, _types_, and _manaCost_ are more complex, with the latter being
considerably more complex than those prior. Let's take a closer look at the first three:

<figure>
<figcaption>File: CardSetInfo.scala</figcaption>
{% highlight scala %}
case class CardSetInfo(
  expansions: Seq[Expansion],
  rarity: Rarity,
  collectorsNumbers: CollectorsNumbers
)
{% endhighlight %}
<figure>

I won't go to into the details about the implementation of the classes represented in _CardSetInfo_, they can be viewed
in the source code. _expansions_ is a Seq\[Expansion\] as cards can belong to multiple expansions due to reprinting.

<figure>
<figcaption>File: Colour.scala</figcaption>
{% highlight scala %}
sealed trait Colour

object Colour {
  case object White extends Colour
  case object Blue extends Colour
  case object Black extends Colour
  case object Red extends Colour
  case object Green extends Colour

  // not really a colour, but easier to represent as one instead of checking for the absence of colours
  case object Colourless extends Colour
}
{% endhighlight %}
</figure>

<figure>
<figcaption>File: CardSubTypes.scala</figcaption>
sealed trait CardSubTypes

object CardSubTypes {
  case object Human extends CardSubTypes
  case object Soldier extends CardSubTypes
}
</figure>

I am considering implementing the two above traits as mix-ins as opposed to what are essentially enumerations. I have
run into problems in the past with going too far with sub-typing in Scala as the language does not support
multiple-inheritence. I have yet to experiment. A post coming soon will contain the results, no doubt.

Now, _ManaCost_ is a bit more complicated:

<figure>
<figcaption>File: ManaCost.scala </figcaption>
{% highlight scala %}
case class ManaCost(iv: SortedMap[Mana, Int]) extends SortedMap[Mana, Int] {
  override implicit def ordering: Ordering[Mana] = iv.ordering

  override def valuesIteratorFrom(start: Mana): Iterator[Int] = iv.valuesIteratorFrom(start)
  override def +[B1 >: Int](kv: (Mana, B1)): SortedMap[Mana, B1] = ManaCost.simplify(iv + kv)
  override def rangeImpl(from: Option[Mana], until: Option[Mana]): SortedMap[Mana, Int] = iv.rangeImpl(from, until)
  override def iteratorFrom(start: Mana): Iterator[(Mana, Int)] = iv.iteratorFrom(start)
  override def keysIteratorFrom(start: Mana): Iterator[Mana] = iv.keysIteratorFrom(start)
  override def get(key: Mana): Option[Int] = iv.get(key)
  override def iterator: Iterator[(Mana, Int)] = iv.iterator
  override def -(key: Mana): SortedMap[Mana, Int] = ManaCost.simplify(iv - key)

  override def toString(): String = {
    val sb = new StringBuilder

    iv.iterator foreach {
      case (m: Mana, i: Int) => m match {
        case Mana.WhiteMana => sb append "W" * i
        case Mana.BlueMana => sb append "U" * i
        case Mana.BlackMana => sb append "B" * i
        case Mana.RedMana => sb append "R" * i
        case Mana.GreenMana => sb append "G" * i
        case Mana.ColourlessMana => sb append "C" * i
        case Mana.AnyMana => sb append i.toString
      }
    }

    sb.toString()
  }

  def convertedManaCost: Int = iv.foldLeft[Int](0)((z, h) => z + h._2)
}

object ManaCost {
  def apply(items: (Mana, Int)*): ManaCost = new ManaCost(simplify(items.toSortedMap))

  private def simplify[B1 >: Int](map: SortedMap[Mana, B1]): SortedMap[Mana, B1] = map.filterNot(_._2 == 0)
}
{% endhighlight %}
</figure>

[monads]: https://github.com/fpinscala/fpinscala/wiki/Chapter-11:-Monads
[first-class functions]: https://en.wikipedia.org/wiki/First-class_function
[parameters]: http://docs.scala-lang.org/tutorials/tour/implicit-parameters.html
[classes]: http://docs.scala-lang.org/overviews/core/implicit-classes.html
[typeclasses]: http://www.cakesolutions.net/teamblogs/demystifying-implicits-and-typeclasses-in-scala
[structural subtyping]: http://langexplr.blogspot.ca/2007/07/structural-types-in-scala-260-rc1.html
[github]: https://github.com/schwitzerm/scala-mtg-types
