# Software Design Patterns

> Collection of design patterns, summarizing [McGill's Comp 303](https://github.com/prmr/SoftwareDesign)

## Design by Contract

Allows method signatures to have a set of pre and post conditions to better specify how the method intends to execute.

```java
public class Card {
    /**
     * ...
     * @pre rank != null && suit != null
     */
    public Card(Rank rank, Suit suit) {
        ...
    }
    ...
}
```

A big part of contract design resides in javadocs and annotations.
Should the pre conditions not be met, a method is not required to have a valid or expected output. Note that these contracts may not always be enforced; it is up to the developer to ensure that the conditions are met.

## Encapsulation

Encapsulation will
* restrict direct access to object components
* limit component access where appropriate
* hide the underlying implementation

to allow for code that
* is generalized to meet its needs without outputting an overly specific interface
* can be reimplemented without modifying the exposed API

### Example

Assume that we have a deck of cards

```java
public class Deck {

    private Stack<Card> cards = ... // some stack

    ...

    public Stack<Card> getCards() {
        return cards;
    }
}
```

An implementor who wishes to read the cards may save the data in another `Stack<Card>` reference. Not only are they aware that this is a stack, but they are free to pop and push other cards onto the stack, which will directly affect the `Deck`. Additionally, if we later notice that we wish to implement the card stack as a `LinkedList`, we would be required to change our interface completely. 

If our `Deck` was instead:

```java
public class Deck {

    private Stack<Card> cards = ... // some stack

    ...

    public Iterable<Card> getCards() {
        return Collections.unmodifiableCollection(stack).iterator();
    }
}
```

1. Changing the underling implementation to a `LinkedList` will not affect the output of `getCards()`.
1. Removing a value from the `Iterable` will raise an exception.

## Strategy Design Pattern

The goal of the strategy design pattern is to provide multiple implementations of a specific interface, so that a user may select a particular strategy and invoke its respective action.

Consider the following:

```java
public class Card {

    private Rank rank;
    private Suit suit;

    public static enum Rank {
        ONE, TWO, THREE, FOUR // assume complete implementation
    }

    public static enum Suit {
        SPADES, HEARTS, CLUBS, DIAMONDS
    }

    public static Comparator<Card> sortByRank() {
        return Comparator.comparing(card -> card.rank);
    }

    public static Comparator<Card> sortBySuit() {
        return Comparator.comparing(card -> card.suit);
    }

}
```

The last two public static methods serve as factories in creating different comparison strategies. By having multiple options, users are free to sort cards in whatever way they would like (eg `Collections.sort(cards, Card.sortByRank())`). Additionally, by having the factories inside the same class, we may access private values through anonymous classes and avoid unnecessary exposure for our implementation.

## Equality

In short, objects can be compared in two ways: reference and structure. The former involves an equality check with `==`, and checks if two variables refer to the same object. Note that for primitives, this is essentially structural equality. To compare the actual contents of objects, we use the `equals(obj)` method.

Proper implementations involve modifying both `equals(obj)` and `hashCode()`, as both are used in classifying objects. Two objects that are meant to be equal must return `true` from `equals(obj)` and have the same `hashCode()`

Example: 

```java
public class Card {

    private Rank rank; // enum
    private Suit suit; // enum

    @Override
    public int hashCode() {
        return rank.ordinal() * 10 + suit.ordinal();
    }

    @Override
    public boolean equals(Object obj) {
        // optimize by checking for same reference
        if (this == obj) return true;
        // check correct type and for nonnull obj
        if (!(obj instanceof Card)) return false;
        Card other = (Card) obj;
        // actual comparison
        return rank == other.rank && suit == other.suit;
    }
}
```

Note that `enums` can be directly compared through references as they exist as singletons.