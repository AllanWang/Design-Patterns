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
 
## Flyweight Pattern

The flyweight pattern aims to preserve object uniqueness by ensuring that no two objects have the same state. Typically, a flyweight is backed by a private static map, whose values are immutable.

In the case of `Cards`, we have a finite set of valid cards, and can therefore use an array for an index to card map:

```java
public class Deck {

    private static final Card[] cards = new Card[52];
    private static final int suitSize = Suit.values().length; // 4

    public static Card getCard(Rank rank, Suit suit) {
        int index = rank.ordinal() * suitSize + suit.ordinal;
        Card card = cards[index];
        if (card == null) {                 // generate once
            card = new Card(suit, rank);
            cards[index] = card;
        }                                   // else reuse
        return card;
    }
}
```

Note that:

* The holder (`cards`) is private and final. No class should have direct access to the holder.
* The holder is `static`, as the `Card`s are shared among all other classes.
* The key to the map (in this case an `int`) is not the same type as the value (`Card`).
The point is to be able to reuse cards, so `getCard` cannot take in a `Card` argument.
* Cards are immutable. Immutability is important to ensure that a given value stays at that value. If cards were mutable, a second retrieval using the same arguments may return a card with a different set of attributes.
* Cards have structural equality. This is not a requirement, but is a plus, as the cards are already differentiated by structure when saved in the holder.
* The retriever is `static` and has all the arguments necessary to create a new `Card`.

Given that we have a small number of items, we may also choose to preinitialize the `cards` and avoid null checks as a whole. The option above will create Cards lazily, which is useful for large or limitness value sets.

## Singleton Design Pattern

In essence, a singleton ensures that there is always at most one instance of a given object. This seems oddly similar to `static` methods or fields, so what is the benefit? The big difference between singletons and static classes is that singletons still uphold OOP practices. `static` classes do not allow for inheritance, and therefore cannot be reused elsewhere. Singletons, at its core, is no different from any other class, and can be extended, can implement an interface, etc.

The common singleton design is like so:

```java
public class Data {
    @Nullable
    private static Data INSTANCE = null;

    @NonNull
    public static Data getInstance() {
        if (INSTANCE == null) 
            INSTANCE = ...; // create new instance
        return INSTANCE;
    }

    public void test()      // assume implementation
}
```

Note that:

* `INSTANCE` is private, static, and nullable (though you may choose to have it always initialized and nonnull)
* `getInstance()` is static and the returned value is always nonnull. The instance should be reused if it exists, and created & saved if it isn't.

If we look at the `test()` method, it would be called through `Data.test()` if it were static. As a singleton, it is simply `Data.getInstance().test()`.

## Unit Testing

A key aspect of good code is one that is proven to work. Unit testing helps address this by allowing incremental (unit) calls with specific arguments and validation for the received response.

Unit testing:
* Aims to ensure that a specific input returns a specific output.
* Is helpful in ensuring that future code changes do not break previously expected results.
* Allows for testing a portion of the code without considering states outside of the test.
* Is not required to test trivial methods that would otherwise completely break your program if it fails. (eg testing if reference equality works for two references of the same object).
* Is not required to test methods whose preconditions aren't met.

A very common library is `JUnit`. The documentation is pretty straightforward, but as a summary:

Use annotations to define aspects of a test class

| Annotation | Meaning |
|---|---|
| @Before | Run before each test |
| @BeforeClass | Run once when class is started, before any test is executed |
| @Test | Marks a method as a unit test |

All assertions can be found using the static import `import static org.junit.Assert.*;`.
Unforunately, static imports aren't typically suggested by IDEs, so you'll have to remember this for Java.

## Composite Design Pattern

Composite designs allow for an aggregation of objects to behave like a single object.
For this to work, each object will implement a primitive interface.
The composite class will:

* Take in multiple primitive interfaces to hold multiple objects.  
* Implement the primitive interface itself to remain compatible.
Note that with this design, a composite item can take in other composite items.

As an example, imagine a bunch of items, each with a name and price.

```java
public interface Item {
    String name();
    float price();
}
```

We can create singular items like so:

```java
public class SingleItem implements Item {

    private final String name;
    private final float price;

    public SingleItem(String name, float price) {
        this.name = name;
        this.price = price;
    }

    @Override
    public String name() {
        return name;
    }

    @Override
    public float price() {
        return price;
    }
}
```

Let's say that we also wish to have combo items, whose price is the sum of the components, with 10% off. We may do the following:

```java
public class ComboItem implements Item {

    private final String name;
    private final Item[] items;

    public ComboItem(String name, Item... items) {
        this.name = name;
        this.items = items;
    }

    @Override
    public String name() {
        return name;
    }
    
    @Override
    public float price() {
        // In this case, you could have saved the price upon creation and avoid
        // calculating it each time; this is just for show
        return (float) (Arrays.stream(items).mapToDouble(Item::price).sum() * 0.9);
    }

}
```

## Decorator Design Pattern

A decorator is used to add some features to a given object. 
Once again, each of our objects will implement a primitive interface.

A decorator will:

* Take in _one_ primitive interface
* Implement the primitive interface itself to remain compatible. 
Note that a decorator can take in another decorator, and stack the features on top of each other.

Going off of the `Item` example above, we may consider a case where an item will be sent as a gift. We may opt to remove the item name, and add a small price for the gift wrap:

```java
public class GiftItem implements Item {

    private final Item item;

    public GiftItem(Item item) {
        this.item = item;
    }

    @Override
    public String name() {
        return "???";
    }

    @Override
    public float price() {
        return item.price() + 0.50f;
    }
}
```

## Cloning

All Java objects have a `clone()` method, but it needs to be implemented by the developer.
The goal of cloning is to create a new object, which is independent of the first one, but structurally equal.

Example: 

```java
public class Card implements Cloneable { // implement this blank interface
    private Rank rank;
    private Suit suit;

    ...

    @Override
    public Card clone() {               // return type can be any subclass of the super class
        try {
            Card clone = super.clone(); // construct from super class, not by own constructor
            clone.rank = this.rank;     // update the remaining attributes
            clone.suit = this.suit;     // specific to this class
        } catch (CloneNotSupportedException e) {
            return null;                // occurs when super.clone is called from a class 
                                        // that doesn't implement Cloneable
                        
        }
    }
}
```

Unfortunately, this isn't as straightforward with final attributes.
You may consider reflection for this.

## Command Design Pattern

The command design pattern is a means of making a method callable without arguments.
This is possible be wrapping all the needed states into an object, so that it may be executeed independently.

Typically, we'll have a command interface:

```java
public interface Command {
    void execute();
}
```

And other concrete commands that implement the interface.

Example:

```java
public class ShuffleDeck implements Command {

    private final Deck deck;

    public ShuffleDeck(Deck deck) {
        this.deck = deck;
    }

    @Override
    public void execute() {
        deck.shuffle();
    }

}
```

## Observer Design Pattern

