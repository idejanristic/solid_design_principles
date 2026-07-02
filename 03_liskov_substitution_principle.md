# Liskov Substitution Principle (LSP)

The **Liskov Substitution Principle (LSP)** states:

> **Let q(x) be a property provable about objects x of type T. Then q(y) should be provable for objects y of type S, where S is a subtype of T.**

In simpler terms:

> **Every subclass or derived class should be substitutable for its base (parent) class without affecting the correctness of the program.**

---

## Understanding the Formal Definition

Let:

* **T** be the base type (parent class)
* **S** be a subtype (derived class)
* **q(x)** be any property or behavior that is true for objects of type **T**

Then:

> If a property holds for objects of type **T**, it must also hold for objects of type **S**.

---

### A Simpler Explanation

An object of a derived class should be able to replace an object of its base class **without breaking the application.**

Or even more simply:

> **If a method expects a parent class, I should be able to pass any of its subclasses and the program should continue to work correctly.**

---

## Key Concepts

### Base Class (Parent Class)

```php
class Bird
{
}
```

`Bird` is the parent (base) class.

---

### Derived Class (Subclass)

```php
class Sparrow extends Bird
{
}
```

`Sparrow` is a subclass of `Bird`.

---

### Substitutability

Suppose a method expects a `Bird`:

```php
function makeSound(Bird $bird)
{
}
```

I should be able to pass:

```php
makeSound(new Sparrow());
makeSound(new Eagle());
```

without any issues.

This property is called **substitutability**.

---

## Bad Example (Violating LSP)

Many people assume:

> Birds can fly.

So they write:

```php
class Bird
{
    public function fly()
    {
        echo "Flying";
    }
}
```

Pigeon:

```php
class Pigeon extends Bird
{
}
```

Everything works.

But then someone adds:

```php
class Penguin extends Bird
{
    public function fly()
    {
        throw new Exception(
            'Penguins cannot fly'
        );
    }
}
```

Now consider this function:

```php
function letBirdFly(Bird $bird)
{
    $bird->fly();
}
```

Calls:

```php
letBirdFly(new Pigeon());
letBirdFly(new Penguin());
```

The second call throws an exception.

Why?

Because:

* `letBirdFly()` expects **any** `Bird`
* `Penguin` **is** a `Bird`
* but it cannot behave like every other `Bird`

Therefore:

❌ `Penguin` is **not** a proper substitute for `Bird`.

The **Liskov Substitution Principle is violated.**

---

### The Problem Is Not Inheritance

The problem is an incorrect class hierarchy.

By writing:

```text
Bird
 └── fly()
```

we implicitly claimed:

> Every bird can fly.

That statement is false.

---

### A Better Design

```php
class Bird
{
}
```

Create an interface:

```php
interface Flyable
{
    public function fly();
}
```

Pigeon:

```php
class Pigeon extends Bird
    implements Flyable
{
    public function fly()
    {
        echo "Flying";
    }
}
```

Penguin:

```php
class Penguin extends Bird
{
}
```

Now:

```php
function letBirdFly(Flyable $bird)
{
    $bird->fly();
}
```

This works:

```php
letBirdFly(new Pigeon());
```

And this is no longer possible:

```php
letBirdFly(new Penguin());
```

Which is exactly what we want.

---

## Another Classic Example

### Bad Design

```php
class Rectangle
{
    protected int $width;
    protected int $height;

    public function setWidth(int $width)
    {
        $this->width = $width;
    }

    public function setHeight(int $height)
    {
        $this->height = $height;
    }

    public function area(): int
    {
        return $this->width * $this->height;
    }
}
```

Square:

```php
class Square extends Rectangle
{
    public function setWidth(int $width)
    {
        $this->width = $width;
        $this->height = $width;
    }

    public function setHeight(int $height)
    {
        $this->width = $height;
        $this->height = $height;
    }
}
```

Function:

```php
function printArea(Rectangle $rectangle)
{
    $rectangle->setWidth(5);
    $rectangle->setHeight(4);

    echo $rectangle->area();
}
```

For:

```php
printArea(new Rectangle());
```

Output:

```text
20
```

For:

```php
printArea(new Square());
```

Output:

```text
16
```

The program behaves differently.

Why?

Because the function assumes:

> If I set the width to **5** and the height to **4**, the area will be **20**.

That assumption is true for `Rectangle`, but not for `Square`.

Therefore:

❌ `Square` is **not** a valid substitute for `Rectangle`.

The **Liskov Substitution Principle is violated.**

---

## Example in a Laravel/PHP Application

Base class:

```php
abstract class PaymentMethod
{
    abstract public function pay(
        float $amount
    ): void;
}
```

Implementations:

```php
class CreditCardPayment
    extends PaymentMethod
{
    public function pay(float $amount): void
    {
        //
    }
}

class PayPalPayment
    extends PaymentMethod
{
    public function pay(float $amount): void
    {
        //
    }
}
```

Service:

```php
class CheckoutService
{
    public function checkout(
        PaymentMethod $payment,
        float $amount
    ) {
        $payment->pay($amount);
    }
}
```

We can use:

```php
$service->checkout(
    new CreditCardPayment(),
    100
);

$service->checkout(
    new PayPalPayment(),
    100
);
```

Everything works.

---

## LSP Violation

```php
class DisabledPayment
    extends PaymentMethod
{
    public function pay(float $amount): void
    {
        throw new Exception(
            'Payments are disabled'
        );
    }
}
```

Now:

```php
$service->checkout(
    new DisabledPayment(),
    100
);
```

The application fails.

`CheckoutService` expects:

> Every `PaymentMethod` can process a payment.

`DisabledPayment` cannot fulfill that contract.

Therefore, **LSP is violated.**

---

## How to Recognize an LSP Violation

A common warning sign is finding code like this in an overridden method:

```php
throw new Exception();
```

or:

```php
return null;
```

or:

```php
// unsupported
```

This often indicates that the subclass cannot fulfill the contract defined by its parent.

---

## Relationship Between LSP and ISP

These two principles are closely related.

A common sequence of mistakes is:

1. You create an interface that is too large (**violating ISP**).
2. Some classes cannot meaningfully implement every method, so they throw exceptions.
3. Those classes are no longer valid substitutes for the base type (**violating LSP**).

That's why **LSP and ISP often go hand in hand.**

---

## An Easy Rule to Remember

If you have:

```text
Animal
 └── makeSound()
```

then every subclass:

```text
Dog
Cat
Cow
```

must be usable wherever an `Animal` is expected.

However, if you have:

```text
Bird
 └── fly()
```

then:

```text
Pigeon ✔
Penguin ✘
```

because a penguin cannot fulfill the behavior expected from every `Bird`.

---

## The Essence of LSP

> **Inheritance is more than an "is-a" relationship.**

> **A subclass must honor the contract and expected behavior of its parent class so that it can replace the parent without affecting the correctness of the program.**

In other words:

> **Wherever a parent object is expected, any of its child objects should be usable without breaking the application's behavior or producing unexpected results.**
