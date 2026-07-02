# Single Responsibility Principle (SRP)

The **Single Responsibility Principle (SRP)** states:

> **A class should have one and only one reason to change, meaning that a class should have only one responsibility (one job).**

Or, more simply:

> **A class should do one thing and do it well.**

---

## Key Concepts

### Responsibility

A **responsibility** is:

> A single, clearly defined purpose or job that a class performs.

For example:

### Good Responsibilities

```text
UserValidator
```

Responsibility:

* Validating user data

---

```text
EmailSender
```

Responsibility:

* Sending emails

---

```text
InvoiceRepository
```

Responsibility:

* Managing invoice persistence in the database

---

### Reason to Change

This is the most important part of SRP.

A class should change **only if its single responsibility changes.**

For example:

```text
EmailSender
```

It should change if:

* the SMTP server changes
* the email delivery mechanism changes

It should **not** change because of:

* user validation
* logging
* database operations

---

## Bad Example (Violating SRP)

```php
class UserService
{
    public function register(array $data)
    {
        // validation

        // save to database

        // send email

        // logging

        // generate PDF
    }
}
```

This class is responsible for:

1. validation
2. database operations
3. sending emails
4. logging
5. PDF generation

It has **five different responsibilities.**

---

### How Many Reasons to Change Does It Have?

If any of the following changes, the class must also change:

* validation rules
* database implementation
* email provider
* logging format
* PDF library

This class has **five reasons to change.**

❌ **SRP is violated.**

---

## Good Example

Separate each responsibility into its own class.

### Validator

```php
class UserValidator
{
    public function validate(array $data)
    {
        //
    }
}
```

### Repository

```php
class UserRepository
{
    public function create(array $data)
    {
        //
    }
}
```

### Email Service

```php
class EmailSender
{
    public function sendWelcomeEmail(
        string $email
    )
    {
        //
    }
}
```

### Logger

```php
class Logger
{
    public function info(string $message)
    {
        //
    }
}
```

### PDF Generator

```php
class PdfGenerator
{
    public function generate()
    {
        //
    }
}
```

Now the registration service becomes:

```php
class UserService
{
    public function __construct(
        private UserValidator $validator,
        private UserRepository $repository,
        private EmailSender $emailSender,
        private Logger $logger
    ) {
    }

    public function register(array $data)
    {
        $this->validator->validate($data);

        $user = $this->repository
            ->create($data);

        $this->emailSender
            ->sendWelcomeEmail($user->email);

        $this->logger
            ->info('User registered');
    }
}
```

---

### Does `UserService` Violate SRP?

**No.**

Its responsibility is:

> **Coordinating (orchestrating) the user registration process.**

It does **not**:

* validate data
* send emails
* interact with the database

It simply coordinates the collaboration between other classes.

It still has **one responsibility**.

---

## Real-World Analogy

Imagine an employee:

```text
Accountant
```

Their responsibility is:

* managing finances

If you also ask them to:

* write software
* clean the office
* manage marketing
* repair computers

then one person has multiple responsibilities.

The same idea applies to classes.

---

## Bad Invoice Example

```php
class Invoice
{
    public function calculateTotal()
    {
        //
    }

    public function saveToDatabase()
    {
        //
    }

    public function print()
    {
        //
    }

    public function sendEmail()
    {
        //
    }
}
```

Responsibilities:

1. business logic
2. database persistence
3. printing
4. email delivery

❌ **SRP is violated.**

---

## Better Design

### Domain Model

```php
class Invoice
{
    public function calculateTotal()
    {
        //
    }
}
```

### Repository

```php
class InvoiceRepository
{
    public function save(
        Invoice $invoice
    )
    {
        //
    }
}
```

### Printer

```php
class InvoicePrinter
{
    public function print(
        Invoice $invoice
    )
    {
        //
    }
}
```

### Mailer

```php
class InvoiceMailer
{
    public function send(
        Invoice $invoice
    )
    {
        //
    }
}
```

Now each class has **one responsibility**.

---

## How Does This Look in Laravel?

A common anti-pattern is a controller that does everything:

```text
Controller
 ├── Validation
 ├── Business logic
 ├── SQL queries
 ├── Sending emails
 ├── File uploads
 └── Logging
```

A controller with 1,000 lines of code.

❌ This is almost always an SRP violation.

---

A better design is:

```text
RegisterUserController
        ↓
RegisterUserService
        ↓
UserRepository
EmailSender
AvatarUploader
Logger
```

Each class has a single responsibility.

---

## How to Recognize an SRP Violation

A class is often violating SRP if it:

* has many methods
* has many constructor dependencies
* works with the database, emails, files, and external APIs at the same time
* changes for completely unrelated reasons

For example:

```php
class ReportService
{
    // SQL
    // PDF
    // Email
    // CSV
    // FTP
    // Logging
}
```

This is usually a sign that the class is doing too much.

---

## An Easy Rule to Remember

❌ Bad:

```text
UserService
 ├── validate
 ├── save
 ├── send email
 ├── upload image
 ├── generate PDF
 └── log
```

Multiple responsibilities.

---

✅ Good:

```text
UserValidator
UserRepository
EmailSender
AvatarUploader
PdfGenerator
Logger
```

Each class has **one job**.

---

## The Essence of SRP

> **A class should have only one responsibility and only one reason to change.**

In other words:

> **If you can think of multiple unrelated reasons why a class would need to be modified, it is probably violating the Single Responsibility Principle.**
