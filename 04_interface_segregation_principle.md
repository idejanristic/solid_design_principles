# Interface Segregation Principle (ISP)

The **Interface Segregation Principle (ISP)** states:

> **A client should never be forced to implement an interface that it doesn't use, nor should clients be forced to depend on methods they do not use.**

Or, more simply:

> **It is better to have multiple small, specialized interfaces than one large interface containing methods that some classes do not need.**

---

## Key Concepts

### Client

A **client** is any class that:

* implements an interface, or
* depends on an interface.

For example:

```php
class BirdKeeper
{
    public function __construct(
        private Bird $bird
    ) {
    }
}
```

Here:

```text
BirdKeeper
```

is a client of the:

```text
Bird
```

interface.

---

### Depending on Methods

Suppose an interface contains:

```php
interface Animal
{
    public function eat();

    public function fly();

    public function swim();
}
```

Every class implementing `Animal` must implement:

* `eat()`
* `fly()`
* `swim()`

even if some of those methods are not applicable.

This means the class is forced to depend on methods it does not actually use.

---

## Bad Example (Violating ISP)

```php
interface Worker
{
    public function work();

    public function eat();

    public function sleep();
}
```

Programmer:

```php
class Programmer implements Worker
{
    public function work()
    {
        //
    }

    public function eat()
    {
        //
    }

    public function sleep()
    {
        //
    }
}
```

So far, everything is fine.

Now suppose you want to introduce a robot:

```php
class Robot implements Worker
{
    public function work()
    {
        //
    }

    public function eat()
    {
        ???
    }

    public function sleep()
    {
        ???
    }
}
```

A robot:

* does not eat,
* does not sleep.

Yet the interface forces it to implement:

```php
eat()
sleep()
```

A common workaround is:

```php
public function eat()
{
    throw new Exception();
}
```

or:

```php
public function eat()
{
    // do nothing
}
```

This is a clear sign that **ISP is being violated.**

---

## Good Example (Following ISP)

Split the interface into smaller, focused interfaces.

```php
interface Workable
{
    public function work();
}

interface Eatable
{
    public function eat();
}

interface Sleepable
{
    public function sleep();
}
```

Programmer:

```php
class Programmer
    implements Workable, Eatable, Sleepable
{
    public function work()
    {
        //
    }

    public function eat()
    {
        //
    }

    public function sleep()
    {
        //
    }
}
```

Robot:

```php
class Robot implements Workable
{
    public function work()
    {
        //
    }
}
```

Now:

* the programmer can work, eat, and sleep,
* the robot only works.

No class is forced to implement methods it does not need.

---

## Another Good Example – Birds

### Bad Design

```php
interface Bird
{
    public function fly();

    public function walk();
}
```

Pigeon:

```php
class Pigeon implements Bird
{
    public function fly()
    {
        //
    }

    public function walk()
    {
        //
    }
}
```

Penguin:

```php
class Penguin implements Bird
{
    public function fly()
    {
        throw new Exception();
    }

    public function walk()
    {
        //
    }
}
```

The problem:

Penguins cannot fly.

Yet the interface forces them to implement:

```php
fly()
```

This violates the Interface Segregation Principle.

---

## Better Design

```php
interface Walkable
{
    public function walk();
}

interface Flyable
{
    public function fly();
}
```

Pigeon:

```php
class Pigeon implements Walkable, Flyable
{
    public function walk()
    {
        //
    }

    public function fly()
    {
        //
    }
}
```

Penguin:

```php
class Penguin implements Walkable
{
    public function walk()
    {
        //
    }
}
```

Each class implements only the behavior that actually applies to it.

---

## Example in a Laravel/PHP Application

### Bad Interface

```php
interface Storage
{
    public function put();

    public function get();

    public function resizeImage();

    public function createThumbnail();
}
```

Local storage:

```php
class LocalStorage implements Storage
{
    // implements everything
}
```

Amazon S3 storage:

```php
class S3Storage implements Storage
{
    public function resizeImage()
    {
        throw new Exception();
    }

    public function createThumbnail()
    {
        throw new Exception();
    }
}
```

The problem:

An S3 storage implementation should not be responsible for image processing.

---

### Better Design

```php
interface FileStorage
{
    public function put();

    public function get();
}

interface ImageProcessor
{
    public function resizeImage();

    public function createThumbnail();
}
```

S3 storage:

```php
class S3Storage implements FileStorage
{
}
```

Image service:

```php
class ImageService implements ImageProcessor
{
}
```

Each interface has a single, focused responsibility and is implemented only where it makes sense.

---

## How to Recognize an ISP Violation

Common warning signs include overridden methods that contain:

```php
throw new Exception();
```

or:

```php
// TODO
```

or:

```php
// not used
```

or:

```php
return null;
```

simply because the interface requires the method.

This is often an indication that the interface is too large and violates ISP.

---

## An Easy Rule to Remember

❌ One large interface:

```text
Animal
 ├── eat()
 ├── fly()
 ├── swim()
 ├── walk()
 └── climb()
```

Every animal is forced to implement every method.

---

✅ Multiple small interfaces:

```text
Eatable
Flyable
Swimmable
Walkable
Climbable
```

Each class implements only what it actually needs.

---

## The Essence of ISP

> **Do not force classes to implement methods they do not use.**

> **Design small, focused interfaces instead of one large "all-in-one" interface.**

That is why this principle is called the **Interface Segregation Principle**—it encourages splitting large interfaces into smaller, more specialized ones so that each client depends only on the behavior it actually needs.
