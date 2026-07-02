# Open-Closed Principle (OCP)

The **Open-Closed Principle (OCP)** states:

> **Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification.**

Or, more simply:

> **You should be able to add new behavior without modifying existing, tested code.**

---

## Key Concepts

### Open for Extension

Being **open for extension** means:

> You can add new functionality without changing existing code.

For example:

Today your application supports:

* PayPal
* Stripe

Tomorrow you want to support:

* Apple Pay

You should be able to add an `ApplePayPayment` class **without modifying the existing code.**

---

### Closed for Modification

Being **closed for modification** means:

> Once a class has been completed and works correctly, you should not have to modify it every time a new requirement appears.

Why?

Because every modification:

* can introduce bugs,
* can break existing functionality,
* requires additional testing.

---

## Bad Example (Violating OCP)

Suppose we have a discount calculation service:

```php
class DiscountService
{
    public function calculate(
        string $customerType,
        float $price
    ): float {
        if ($customerType === 'regular') {
            return $price;
        }

        if ($customerType === 'premium') {
            return $price * 0.9;
        }

        return $price;
    }
}
```

Usage:

```php
$service->calculate('premium', 100);
```

Everything works.

---

### A New Requirement

Now we introduce a new customer type:

```text
vip
```

So we modify the class:

```php
if ($customerType === 'vip') {
    return $price * 0.8;
}
```

Later, another requirement appears:

```text
gold
```

Again, we modify the class:

```php
if ($customerType === 'gold') {
    return $price * 0.85;
}
```

The class keeps changing.

Every new customer type requires you to:

* open the file,
* modify existing code,
* increase the risk of introducing bugs.

❌ **OCP is violated.**

---

## Good Example (Following OCP)

Let's introduce an abstraction.

```php
interface DiscountStrategy
{
    public function calculate(
        float $price
    ): float;
}
```

Implementations:

```php
class RegularDiscount
    implements DiscountStrategy
{
    public function calculate(
        float $price
    ): float {
        return $price;
    }
}
```

```php
class PremiumDiscount
    implements DiscountStrategy
{
    public function calculate(
        float $price
    ): float {
        return $price * 0.9;
    }
}
```

Service:

```php
class DiscountService
{
    public function calculate(
        DiscountStrategy $discount,
        float $price
    ): float {
        return $discount->calculate($price);
    }
}
```

Usage:

```php
$service->calculate(
    new PremiumDiscount(),
    100
);
```

---

## A New Requirement

We add another implementation:

```php
class VipDiscount
    implements DiscountStrategy
{
    public function calculate(
        float $price
    ): float {
        return $price * 0.8;
    }
}
```

We do **not** modify:

* `DiscountService`
* `PremiumDiscount`
* `RegularDiscount`

We simply add a new class.

That is the Open-Closed Principle.

---

## Payment Example

### Bad Design

```php
class PaymentService
{
    public function pay(
        string $method,
        float $amount
    ) {
        if ($method === 'paypal') {
            //
        }

        if ($method === 'stripe') {
            //
        }

        if ($method === 'apple') {
            //
        }
    }
}
```

Every new payment method requires you to:

* modify the class,
* add another `if` statement,
* retest the class.

❌ **OCP is violated.**

---

### Better Design

```php
interface PaymentGateway
{
    public function pay(
        float $amount
    ): void;
}
```

Implementations:

```php
class PayPalGateway
    implements PaymentGateway
{
    public function pay(
        float $amount
    ): void {
        //
    }
}
```

```php
class StripeGateway
    implements PaymentGateway
{
    public function pay(
        float $amount
    ): void {
        //
    }
}
```

Service:

```php
class PaymentService
{
    public function pay(
        PaymentGateway $gateway,
        float $amount
    ): void {
        $gateway->pay($amount);
    }
}
```

Adding support for Apple Pay:

```php
class ApplePayGateway
    implements PaymentGateway
{
    public function pay(
        float $amount
    ): void {
        //
    }
}
```

No existing files need to be modified.

You simply add a new class.

✅ **OCP is respected.**

---

## How Does This Look in Laravel?

Interface:

```php
interface NotificationChannel
{
    public function send(
        string $message
    ): void;
}
```

Implementations:

```php
class EmailChannel
    implements NotificationChannel
{
    public function send(
        string $message
    ): void {
        //
    }
}
```

```php
class SmsChannel
    implements NotificationChannel
{
    public function send(
        string $message
    ): void {
        //
    }
}
```

Service:

```php
class NotificationService
{
    public function send(
        NotificationChannel $channel,
        string $message
    ) {
        $channel->send($message);
    }
}
```

Tomorrow you decide to support Slack:

```php
class SlackChannel
    implements NotificationChannel
{
    public function send(
        string $message
    ): void {
        //
    }
}
```

You do **not** modify:

* `NotificationService`
* `EmailChannel`
* `SmsChannel`

You simply extend the system.

That is the essence of OCP.

---

## How to Recognize an OCP Violation

A common sign is a growing chain of:

```php
if ($type === '...')
```

or:

```php
switch ($type)
```

For example:

```php
switch ($paymentMethod) {
    case 'paypal':
    case 'stripe':
    case 'apple':
    case 'google':
    case 'bank':
        ...
}
```

Every new case requires modifying existing code.

This is often a sign that the design is **not open for extension.**

---

## Relationship with the Dependency Inversion Principle (DIP)

The Open-Closed Principle is usually achieved through:

* interfaces,
* abstract classes,
* polymorphism,
* dependency injection.

That's why you'll often see a design like this:

```text
PaymentService
       ↓
PaymentGateway
      ↑
 ┌────┼────────┐
PayPal Stripe ApplePay
```

Adding a new implementation:

```text
GooglePay
```

does **not** require modifying any existing code.

---

## An Easy Rule to Remember

❌ Bad:

```text
PaymentService
 ├── if PayPal
 ├── if Stripe
 ├── if Apple Pay
 ├── if Google Pay
 └── if Bank Transfer
```

Every new feature requires modifying the class.

---

✅ Good:

```text
PaymentGateway
       ↑
 ┌─────┼────────┐
PayPal Stripe ApplePay
```

Adding a new feature:

```text
GooglePay
```

means simply adding another implementation.

---

## The Essence of OCP

> **Do not keep modifying existing code just to add new behavior.**

> **Design your system so that new functionality is added by extending it (through new classes or implementations), rather than by modifying existing classes.**

In other words:

> **Add new functionality—don't change what already works.**
