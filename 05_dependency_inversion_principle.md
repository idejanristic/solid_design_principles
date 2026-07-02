# Dependency Inversion Principle (DIP)

The **Dependency Inversion Principle (DIP)** states:

> **Entities should depend on abstractions, not on concretions. High-level modules should not depend on low-level modules. Instead, both should depend on abstractions.**

Or, more simply:

> **Business logic should depend on interfaces or abstract classes, not on concrete implementations.**

Let's break down the key concepts.

---

## 1. Concretions (Concrete Implementations)

**Concretions** are concrete classes that contain the actual implementation.

For example:

```php
class MySqlLogger
{
    public function log(string $message)
    {
        // write to MySQL
    }
}
```

or:

```php
class PayPalPayment
{
    public function pay(float $amount)
    {
        // process payment through PayPal
    }
}
```

These are concrete implementations because we know exactly:

* MySQL is used
* PayPal is used

If you later decide to switch to:

* PostgreSQL
* Stripe
* file-based logging

you would need to modify the existing code.

---

## 2. Abstractions

**Abstractions** represent **contracts**. They define **what** something does, but not **how** it does it.

In PHP, abstractions are typically:

* interfaces
* abstract classes

Example:

```php
interface Logger
{
    public function log(string $message): void;
}
```

This does not specify:

* where the log is stored,
* how the logging is performed.

It simply says:

> "There is a way to log messages."

Implementations:

```php
class MySqlLogger implements Logger
{
    public function log(string $message): void
    {
        // MySQL implementation
    }
}

class FileLogger implements Logger
{
    public function log(string $message): void
    {
        // File implementation
    }
}
```

---

## 3. High-Level Module

A **high-level module** contains the application's **business logic**.

For example:

```php
class OrderService
{
    public function completeOrder()
    {
        // business rules
    }
}
```

or:

```php
class UserService
{
    public function register()
    {
        // user registration
    }
}
```

These classes represent **what the application does**.

---

## 4. Low-Level Module

**Low-level modules** deal with implementation details such as:

* sending emails,
* database access,
* logging,
* communicating with external APIs,
* storing files.

For example:

```php
class MySqlLogger
{
}

class EmailSender
{
}

class StripePayment
{
}
```

These classes do not contain business rules—they handle technical details.

---

## Bad Example (Violating DIP)

```php
class MySqlLogger
{
    public function log(string $message)
    {
        echo $message;
    }
}

class UserService
{
    private MySqlLogger $logger;

    public function __construct()
    {
        $this->logger = new MySqlLogger();
    }

    public function register()
    {
        // user registration

        $this->logger->log('User registered');
    }
}
```

Here:

**High-level module**

```text
UserService
```

depends directly on the **low-level module**

```text
MySqlLogger
```

The problem:

If you later want to use:

```text
FileLogger
```

you must modify `UserService`.

The business logic knows too much about the implementation details.

---

## Good Example (Following DIP)

```php
interface Logger
{
    public function log(string $message): void;
}
```

Implementations:

```php
class MySqlLogger implements Logger
{
    public function log(string $message): void
    {
        echo "MySQL: $message";
    }
}

class FileLogger implements Logger
{
    public function log(string $message): void
    {
        echo "File: $message";
    }
}
```

High-level module:

```php
class UserService
{
    public function __construct(
        private Logger $logger
    ) {
    }

    public function register()
    {
        $this->logger->log('User registered');
    }
}
```

Usage:

```php
$service = new UserService(
    new MySqlLogger()
);
```

or

```php
$service = new UserService(
    new FileLogger()
);
```

`UserService` no longer knows:

* whether the implementation uses MySQL,
* a file,
* Redis,
* or a cloud logging service.

It only cares that:

> "I have something that can log messages."

---

## Why Is It Called Dependency *Inversion*?

Without DIP:

```text
UserService
      ↓
MySqlLogger
```

The high-level module depends directly on the low-level module.

With DIP:

```text
            Logger
            ↑     ↑
            |     |
UserService      MySqlLogger
```

Both `UserService` and `MySqlLogger` depend on the same abstraction (`Logger`).

The dependency has been **inverted** because the business logic no longer depends on a concrete implementation.

---

## How Does This Look in Laravel?

Interface:

```php
interface PaymentGateway
{
    public function pay(float $amount): void;
}
```

Implementations:

```php
class StripePaymentGateway
    implements PaymentGateway
{
    public function pay(float $amount): void
    {
        //
    }
}

class PayPalPaymentGateway
    implements PaymentGateway
{
    public function pay(float $amount): void
    {
        //
    }
}
```

Service:

```php
class OrderService
{
    public function __construct(
        private PaymentGateway $gateway
    ) {
    }

    public function checkout(float $amount)
    {
        $this->gateway->pay($amount);
    }
}
```

In the Laravel Service Container:

```php
$this->app->bind(
    PaymentGateway::class,
    StripePaymentGateway::class
);
```

If you later decide to switch to PayPal:

```php
$this->app->bind(
    PaymentGateway::class,
    PayPalPaymentGateway::class
);
```

`OrderService` does not need to change at all.

---

## Summary

| Term                  | Meaning                                                         | Example                                              |
| --------------------- | --------------------------------------------------------------- | ---------------------------------------------------- |
| **Abstraction**       | A contract that defines behavior without implementation details | `interface Logger`                                   |
| **Concretion**        | A concrete implementation of an abstraction                     | `MySqlLogger`, `FileLogger`                          |
| **High-level module** | A module containing business logic                              | `UserService`, `OrderService`                        |
| **Low-level module**  | A module handling implementation details                        | `MySqlLogger`, `StripePaymentGateway`, `EmailSender` |

---

## The Essence of DIP

> **Business logic should not know how a particular task is implemented.**

> **Instead, it should depend only on abstractions (interfaces or abstract classes), while the concrete implementation can be replaced without modifying the business logic.**

In other words:

> **Depend on abstractions, not on concrete implementations.**
