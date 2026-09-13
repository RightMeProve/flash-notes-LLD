# Design Patterns

A collection of commonly used **Object-Oriented Design Patterns** implemented and explained in Java.

## Patterns Covered

### Behavioral Patterns
- [Strategy Pattern](#1-strategy-pattern)
- [Observer Pattern](#2-observer-pattern)
- [State Pattern](#9-state-pattern)
- [Chain of Responsibility Pattern](#10-chain-of-responsibility-pattern)
- [Null Object Pattern](#11-null-object-pattern)

### Structural Patterns
- [Decorator Pattern](#3-decorator-pattern)
- [Proxy Pattern](#8-proxy-design-pattern)
- [Composite Pattern](#12-composite-pattern)

### Creational Patterns
- [Builder Pattern](#4-builder-pattern)
- [Factory Pattern](#5-factory-pattern)
- [Abstract Factory Pattern](#6-abstract-factory-pattern)

### Comparisons
- [Builder vs Decorator](#builder-vs-decorator)
- [Proxy vs Decorator](#proxy-vs-decorator)
- [Factory vs Abstract Factory vs Builder](#factory-vs-abstract-factory-vs-builder)

---

# 1. Strategy Pattern

## Problem Statement

Suppose we have a parent `Vehicle` class with a `drive()` method, and multiple types of vehicles extend it:

- `SportsVehicle`
- `FamilyVehicle`
- `OffRoadVehicle`
- etc.

The problem occurs when multiple child classes have the **same implementation** of `drive()`.

For example, suppose `SportsVehicle` and `FamilyVehicle` both use the same normal driving behavior.

We might end up writing the same code in both classes.

If the driving behavior needs to change, we have to modify it in multiple classes.

This leads to:

- Code duplication
- Difficult maintenance
- Tight coupling between the vehicle and its driving behavior

## Solution

Instead of implementing `drive()` directly inside every vehicle class, we extract the behavior into a separate **Strategy**.

```java
interface DrivingStrategy {
    void drive();
}
```

Different strategies can then be implemented:

```java
class NormalDrivingStrategy implements DrivingStrategy {

    @Override
    public void drive() {
        System.out.println("Normal driving");
    }
}

class SportsDrivingStrategy implements DrivingStrategy {

    @Override
    public void drive() {
        System.out.println("Sports driving");
    }
}
```

The `Vehicle` can have a `DrivingStrategy`:

```java
class Vehicle {

    private DrivingStrategy drivingStrategy;

    Vehicle(DrivingStrategy drivingStrategy) {
        this.drivingStrategy = drivingStrategy;
    }

    public void drive() {
        drivingStrategy.drive();
    }
}
```

The required strategy can be passed using **constructor injection**:

```java
Vehicle familyVehicle =
        new Vehicle(new NormalDrivingStrategy());

Vehicle sportsVehicle =
        new Vehicle(new SportsDrivingStrategy());
```

Now, if the implementation of `NormalDrivingStrategy` changes, we only need to change it in one place.

## Key Idea

> Encapsulate a behavior that can vary and make it interchangeable.

Instead of inheriting behavior from a parent class, we **compose** the required behavior into the object.

This follows:

> **Favor composition over inheritance.**

---

# 2. Observer Pattern

## Problem Statement

Suppose we have a weather system.

Whenever the weather changes, multiple devices need to be updated:

- Mobile App
- Monitor
- Weather Station
- Digital Billboard

If the weather system directly manages every device, it becomes tightly coupled to them.

```text
Weather
   |
   ├── Mobile
   ├── Monitor
   ├── Weather Station
   └── Billboard
```

Adding or removing a device would require modifying the weather system.

## Solution

The **Observer Pattern** establishes a **one-to-many relationship** between an object and its dependents.

There are two main components:

### Observable / Subject

The object whose state changes.

In our example:

```text
WeatherObservable
```

It maintains a list of observers and provides methods such as:

```text
addObserver()
removeObserver()
notifyObservers()
```

### Observer

The objects that want to receive updates.

For example:

```text
MobileObserver
MonitorObserver
BillboardObserver
```

The relationship becomes:

```text
              WeatherObservable
                     |
          -------------------------
          |           |           |
       Mobile      Monitor    Billboard
      Observer     Observer    Observer
```

Whenever the weather changes:

```text
WeatherObservable
       |
       ↓
notifyObservers()
       |
       ↓
All registered observers are updated
```

## Relationships

An observer can maintain a reference to the observable:

```java
class MonitorObserver implements Observer {

    private WeatherObservable weather;

    MonitorObserver(WeatherObservable weather) {
        this.weather = weather;
    }
}
```

Therefore:

```text
MonitorObserver IS-A Observer
WeatherObservable IS-A Observable

MonitorObserver HAS-A WeatherObservable
```

The `HAS-A` relationship allows the observer to register itself with the particular observable it wants to observe.

## Key Idea

> When the state of one object changes, automatically notify all interested objects without tightly coupling the subject to its observers.

---

# 3. Decorator Pattern

## Problem Statement

Suppose we have a pizza and want to customize it with different toppings:

- Capsicum
- Onion
- Extra Cheese
- Mushroom
- Hand Tossed
- Fresh Pan
- etc.

If we create a separate class for every possible combination, we can end up with a huge number of classes.

For example:

```text
Pizza
PizzaWithCheese
PizzaWithOnion
PizzaWithCheeseAndOnion
PizzaWithCheeseOnionAndCapsicum
PizzaWithOnionAndCapsicum
...
```

This leads to **class explosion** because of the permutations and combinations of customizations.

## Solution

Instead of creating a class for every combination, we use the **Decorator Pattern**.

We create a common component:

```java
interface Pizza {
    int getCost();
}
```

The base pizza implements it:

```java
class BasicPizza implements Pizza {

    @Override
    public int getCost() {
        return 100;
    }
}
```

Now we create decorators.

A decorator has both:

- **IS-A relationship** with the component
- **HAS-A relationship** with the component

For example:

```java
class CheeseDecorator implements Pizza {

    private Pizza pizza;

    CheeseDecorator(Pizza pizza) {
        this.pizza = pizza;
    }

    @Override
    public int getCost() {
        return pizza.getCost() + 30;
    }
}
```

We can now combine decorators:

```java
Pizza pizza = new BasicPizza();

pizza = new CheeseDecorator(pizza);
pizza = new OnionDecorator(pizza);
pizza = new CapsicumDecorator(pizza);
```

Conceptually:

```text
CapsicumDecorator
        ↓
 OnionDecorator
        ↓
CheeseDecorator
        ↓
   BasicPizza
```

Each decorator wraps the previous object and adds its own behavior.

## Key Idea

> Dynamically add responsibilities or behavior to an existing object without modifying its original class.

---

# 4. Builder Pattern

## Problem Statement

Suppose we have a class with many fields, and many of them are optional:

```java
class Student {

    String name;
    int age;
    String email;
    String phone;
    String address;
    String college;
    String branch;
}
```

One approach would be to create multiple constructors:

```text
Student(name)

Student(name, age)

Student(name, age, email)

Student(name, age, email, phone)

...
```

This creates several problems:

- Too many constructors
- Long parameter lists
- Poor readability
- Easy to pass arguments in the wrong order
- Difficult to maintain

Also, Java does not consider parameter names while overloading.

For example:

```java
Student(String name, String email)
Student(String email, String name)
```

These cannot coexist because they have the same parameter types:

```text
(String, String)
```

## Solution

Use the **Builder Pattern**.

Create a builder that contains the values required to construct the `Student`.

The builder provides methods such as:

```text
name()
age()
email()
phone()
```

Each method sets the corresponding value and returns the builder itself.

This enables method chaining:

```java
Student student = new StudentBuilder()
        .name("Satyam")
        .age(25)
        .email("abc@gmail.com")
        .phone("9999999999")
        .build();
```

The `build()` method finally creates and returns the actual `Student` object.

## Director

A `Director` is **optional**.

It is useful when we have predefined construction processes or business rules.

```text
Director
   ↓
Builder
   ↓
Student
```

The Director can define predefined construction flows such as:

```text
createEngineeringStudent()
createMedicalStudent()
createMBAStudent()
```

The Director decides the **sequence/procedure of construction**, while the Builder performs the construction.

For simple Builder implementations, a Director is often unnecessary.

## Key Idea

> Separate the construction of a complex object from the object itself, allowing the object to be created step-by-step with different configurations.

---

# Builder vs Decorator


| Builder                         | Decorator                                 |
| ------------------------------- | ----------------------------------------- |
| Creational Design Pattern       | Structural Design Pattern                 |
| Focuses on creating an object   | Focuses on adding behavior/responsibility |
| Configures object properties    | Combines additional behaviors             |
| Used during object construction | Can be applied dynamically at runtime     |
| Example:`StudentBuilder`        | Example:`CheeseDecorator`                 |

### Easy Way to Remember

**Builder:**

> "How do I create/configure this object?"

```text
Builder → Object
```

**Decorator:**

> "I already have an object. How can I add more behavior to it?"

```text
Decorator → Decorator → Object
```

---

# 5. Factory Pattern

## Problem Statement

Suppose we have different implementations of a `Notification`:

```text
EmailNotification
SMSNotification
PushNotification
```

All of them implement:

```java
interface Notification {
    void send();
}
```

Now imagine the client receives the notification type from a request:

```java
String type = request.getNotificationType();
```

The client needs a `Notification`, but it should not be responsible for deciding which concrete class to create.

Without a Factory, the client may contain:

```java
Notification notification;

if (type.equals("EMAIL")) {
    notification = new EmailNotification();
}
else if (type.equals("SMS")) {
    notification = new SMSNotification();
}
else if (type.equals("PUSH")) {
    notification = new PushNotification();
}
```

Now the client has **two responsibilities**:

1. Decide which concrete object to create
2. Use that object

This also couples the client directly to all concrete notification classes.

## Solution

Delegate the object-creation responsibility to a **Factory**.

Create:

```text
NotificationFactory
```

which is responsible for deciding which concrete notification should be created.

```java
class NotificationFactory {

    public static Notification create(String type) {

        if (type.equals("EMAIL")) {
            return new EmailNotification();
        }

        if (type.equals("SMS")) {
            return new SMSNotification();
        }

        if (type.equals("PUSH")) {
            return new PushNotification();
        }

        throw new IllegalArgumentException("Unknown type");
    }
}
```

Now the client simply asks for the required abstraction:

```java
Notification notification =
        NotificationFactory.create(type);

notification.send();
```

The client doesn't need to know which concrete implementation is being created.

## Key Idea

> Encapsulate the decision of which concrete object to create and separate object creation from object usage.

### Important Point

Factory is **not needed simply because we want to avoid writing `new`**.

If we always know that we need a `Car`, this is perfectly fine:

```java
Vehicle vehicle = new Car();
```

A Factory becomes useful when the **concrete type needs to be selected dynamically** or when object-creation logic is complex/repeated and should be centralized.

---

# 6. Abstract Factory Pattern

## Problem Statement

Suppose our application needs a group of related objects.

For example, a GUI application supports:

- Windows
- Mac

Each platform has its own family of UI components.

### Windows Family

```text
WindowsButton
WindowsCheckbox
WindowsTextField
```

### Mac Family

```text
MacButton
MacCheckbox
MacTextField
```

We want to ensure that objects from the same family are created together.

For example:

```text
WindowsButton + WindowsCheckbox      ✅

MacButton + MacCheckbox              ✅

WindowsButton + MacCheckbox          ❌
```

The problem is that the client should not have to worry about:

> "Which Windows/Mac implementation should I create for every component?"

It should simply say:

> "I want the Windows UI family."

## Solution

Create an **Abstract Factory** that defines methods for creating the entire family of related objects.

```java
interface GUIFactory {

    Button createButton();

    Checkbox createCheckbox();

    TextField createTextField();
}
```

Now create concrete factories for each family.

### Windows Factory

```java
class WindowsFactory implements GUIFactory {

    public Button createButton() {
        return new WindowsButton();
    }

    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }

    public TextField createTextField() {
        return new WindowsTextField();
    }
}
```

### Mac Factory

```java
class MacFactory implements GUIFactory {

    public Button createButton() {
        return new MacButton();
    }

    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }

    public TextField createTextField() {
        return new MacTextField();
    }
}
```

Now the client chooses the factory/family:

```java
GUIFactory factory = new WindowsFactory();

Button button = factory.createButton();
Checkbox checkbox = factory.createCheckbox();
TextField textField = factory.createTextField();
```

The client gets a consistent family of objects without knowing their concrete classes.

## Key Idea

> Provide an interface for creating a family of related objects without specifying their concrete classes.

---

# Factory vs Abstract Factory

## Factory

Factory generally focuses on creating **one type/category of product**.

```text
VehicleFactory
      |
      ├── Car
      ├── Bike
      └── Truck
```

The question is:

> **"Which Vehicle should I create?"**

Example:

```java
Vehicle vehicle =
        VehicleFactory.createVehicle(type);
```

---

## Abstract Factory

Abstract Factory focuses on creating a **family of related products**.

```text
                 GUIFactory
                  /       \
                 /         \
       WindowsFactory     MacFactory
             |                 |
       -------------       -------------
       |     |     |       |     |     |
    Button Checkbox Text  Button Checkbox Text
```

The question is:

> **"Which family of related objects should I create?"**

Example:

```java
GUIFactory factory = new WindowsFactory();

Button button = factory.createButton();
Checkbox checkbox = factory.createCheckbox();
```

Both objects belong to the Windows family.

---

# 7. Chain of Responsibility Pattern

## Problem Statement

Suppose we have a request that can be handled by **one of many possible handlers**.

We don't know beforehand which handler will be able to handle the request.

For example, imagine an expense approval system:

```text
Manager → Director → VP → CEO
```

Depending on the amount:

- Manager can approve small expenses
- Director can approve larger expenses
- VP can approve even larger expenses
- CEO can approve the highest amounts

Instead of writing a large chain of `if-else` statements:

```java
if (amount <= 1000) {
    manager.approve();
}
else if (amount <= 10000) {
    director.approve();
}
else if (amount <= 100000) {
    vp.approve();
}
else {
    ceo.approve();
}
```

we can pass the request through a **chain of handlers**.

Each handler gets an opportunity to handle the request.

If it cannot handle it, it delegates the request to the next handler.

## Solution

Create a common `Processor`/`Handler` interface or abstract class.

Each handler contains a reference to the **next handler**.

```text
Request
   ↓
Handler 1
   ↓
Handler 2
   ↓
Handler 3
   ↓
Handler 4
```

Each handler follows the same basic logic:

```text
Can I handle this request?
       |
   ┌───┴───┐
   ↓       ↓
  YES      NO
   ↓       ↓
Handle   Pass to next
```

For example:

```java
abstract class ExpenseHandler {

    protected ExpenseHandler next;

    public void setNext(ExpenseHandler next) {
        this.next = next;
    }

    public abstract void handle(double amount);
}
```

Different handlers can implement their own conditions:

```java
class Manager extends ExpenseHandler {

    @Override
    public void handle(double amount) {

        if (amount <= 1000) {
            System.out.println("Manager approved");
        }
        else if (next != null) {
            next.handle(amount);
        }
    }
}
```

Another handler:

```java
class Director extends ExpenseHandler {

    @Override
    public void handle(double amount) {

        if (amount <= 10000) {
            System.out.println("Director approved");
        }
        else if (next != null) {
            next.handle(amount);
        }
    }
}
```

The client constructs the chain:

```java
ExpenseHandler manager = new Manager();
ExpenseHandler director = new Director();
ExpenseHandler vp = new VP();
ExpenseHandler ceo = new CEO();

manager.setNext(director);
director.setNext(vp);
vp.setNext(ceo);
```

Now the client only needs to send the request to the first handler:

```java
manager.handle(5000);
```

The request flows through the chain:

```text
Request: ₹5000
       ↓
   Manager
       ↓
   Director
       ↓
     VP
```

The Director handles it because the amount is within its limit.

The client doesn't need to know which handler will ultimately handle the request.

## Key Idea

> Pass a request through a chain of handlers, where each handler can either handle the request or delegate it to the next handler.

### Important Characteristics

- The sender doesn't need to know which handler will process the request.
- Each handler decides whether to handle or forward the request.
- The chain can be changed by adding, removing, or reordering handlers.
- It helps avoid large `if-else` or `switch` statements in the client.

### Easy Way to Remember

> **"Try me. If I can't handle it, I'll pass it to the next person."**

```text
Client
  ↓
Handler 1 → Handler 2 → Handler 3 → Handler 4
   ↓            ↓           ↓           ↓
 Can't        Can't       Can          ...
 handle       handle      handle
```

## Real-World Examples

Common use cases include:

- Approval systems
- Logging systems
- Exception/error handling
- Authentication/authorization pipelines
- Request processing pipelines
- Middleware chains
- Servlet filters
- Customer support escalation systems


# 8. Proxy Design Pattern

## Problem Statement

Suppose we have an object that performs some operation, but we want to perform some additional work **before or after the actual operation**.

For example, we may want to:

- Restrict access to certain requests
- Perform authentication/authorization
- Add caching
- Perform logging
- Perform preprocessing/post-processing
- Control access to an expensive resource
- Delay object creation until it is actually required

If we put all this logic directly inside the actual implementation, the class can become tightly coupled with these additional responsibilities.

For example:

```text id="7d9k2p"
Client
   ↓
EmployeeDao
   ↓
Database
```

Suppose we now want to check whether the client has permission before accessing the database.

We could add the permission logic inside `EmployeeDao`, but now `EmployeeDao` is responsible for both:

1. Performing the actual database operation
2. Controlling access to that operation

## Solution

Use the **Proxy Design Pattern**.

Create an interface:

```java id="w6h8tk"
interface EmployeeDao {

    void createEmployee();

    void deleteEmployee();
}
```

The **Real Object** contains the actual business logic:

```java id="k4d1pf"
class EmployeeDaoImpl implements EmployeeDao {

    @Override
    public void createEmployee() {
        System.out.println("Employee created");
    }

    @Override
    public void deleteEmployee() {
        System.out.println("Employee deleted");
    }
}
```

Now create a **Proxy** that implements the same interface.

The Proxy maintains a reference to the real object:

```java id="j2r4k8"
class EmployeeDaoProxy implements EmployeeDao {

    private EmployeeDao employeeDao;

    EmployeeDaoProxy(EmployeeDao employeeDao) {
        this.employeeDao = employeeDao;
    }

    @Override
    public void createEmployee() {

        // Pre-processing
        System.out.println("Checking permission...");

        employeeDao.createEmployee();

        // Post-processing
        System.out.println("Logging...");
    }

    @Override
    public void deleteEmployee() {

        System.out.println("Checking permission...");

        employeeDao.deleteEmployee();
    }
}
```

The client interacts with the **Proxy**, not directly with the real implementation:

```java id="0nd8sn"
EmployeeDao employeeDao =
        new EmployeeDaoProxy(new EmployeeDaoImpl());

employeeDao.createEmployee();
```

The request flows as:

```text id="6x4w9z"
             Client
                |
                ↓
        EmployeeDaoProxy
          /           \
      Pre-process      |
                       ↓
                EmployeeDaoImpl
                       |
                       ↓
                 Actual Operation
                       |
                       ↓
                  Post-process
```

## Important Relationships

The Proxy and Real Object implement the **same interface**:

```text id="3r8x6y"
EmployeeDao
    ↑
    |
    ├───────────────┐
    |               |
    |               |
EmployeeDaoImpl   EmployeeDaoProxy
                    |
                    | HAS-A
                    ↓
             EmployeeDaoImpl
```

Therefore:

```text id="y3g0bq"
EmployeeDaoImpl IS-A EmployeeDao

EmployeeDaoProxy IS-A EmployeeDao

EmployeeDaoProxy HAS-A EmployeeDao
```

This is what allows the client to use the Proxy exactly like the real object.

## Why is this useful?

The client doesn't need to know whether it is talking to the real object or a proxy.

```java id="j1j8yq"
EmployeeDao employeeDao =
        new EmployeeDaoProxy(new EmployeeDaoImpl());
```

The client simply does:

```java id="e8c4gk"
employeeDao.createEmployee();
```

The Proxy can decide what should happen before allowing the request to reach the real object.

For example:

```java id="3c6l3w"
if (!user.isAuthorized()) {
    throw new SecurityException("Access denied");
}

employeeDao.createEmployee();
```

The actual `EmployeeDaoImpl` doesn't need to contain this authorization logic.

---

# Proxy for Caching

Proxy can also be used to add caching.

Suppose the real object performs an expensive operation:

```java id="z5g9l1"
class EmployeeServiceImpl implements EmployeeService {

    public Employee getEmployee(int id) {
        // Expensive database operation
    }
}
```

The proxy can check the cache first:

```text id="q3n8s1"
Client
  ↓
Proxy
  ↓
Is data in cache?
  |
  ├── YES → Return cached data
  |
  └── NO
       ↓
   Real Object
       ↓
   Database
       ↓
   Store in cache
       ↓
   Return result
```

The real implementation remains focused on its actual responsibility.

---

# Proxy vs Decorator

Proxy and Decorator can look very similar because both commonly use:

```text
IS-A + HAS-A
```

For example:

```text id="x7f2k3d"
Proxy
  |
  ├── IS-A → EmployeeDao
  |
  └── HAS-A → EmployeeDaoImpl
```

and:

```text id="c8k1p4"
Decorator
  |
  ├── IS-A → Pizza
  |
  └── HAS-A → Pizza
```

The **intent** is what differentiates them.

### Proxy

> **Controls access to an existing object.**

Examples:

- Authorization
- Authentication
- Caching
- Lazy loading
- Remote access
- Logging/access control

Think:

> **"Should I allow this request to reach the real object?"**

### Decorator

> **Adds additional responsibilities or behavior to an object.**

Examples:

```text id="8k0r2j"
Basic Pizza
     ↓
Cheese
     ↓
Onion
     ↓
Capsicum
```

Think:

> **"What additional behavior can I add to this object?"**

---

# Proxy in Spring

Spring heavily uses **proxies** internally.

For example, features such as:

- `@Transactional`
- `@Cacheable`
- `@PreAuthorize`
- Aspect-oriented programming (AOP)

can be implemented using proxies.

Conceptually:

```text id="n2w7vc"
Your Code
    ↓
Spring Proxy
    |
    ├── Transaction handling
    ├── Security checks
    ├── Caching
    ├── Logging
    |
    ↓
Actual Bean
```

So when you call a Spring-managed bean, you may actually be interacting with a **proxy object** that performs additional processing before delegating to the actual bean.

## Key Idea

> **Proxy provides a substitute or placeholder for another object and controls access to that object while keeping the client unaware of the underlying implementation.**

### Easy Way to Remember

**Proxy:**

> **"I'll stand in front of the real object and decide/control what happens before the request reaches it."**

```text id="b5m3zq"
Client
  ↓
Proxy
  ↓
Real Object
```

# Factory vs Abstract Factory vs Builder


| Pattern              | Main Question                                         |
| -------------------- | ----------------------------------------------------- |
| **Factory**          | Which object should I create?                         |
| **Abstract Factory** | Which family of related objects should I create?      |
| **Builder**          | How should I construct/configure this complex object? |

### Factory

```text
Choose the concrete type
          ↓
     Create object
```

### Abstract Factory

```text
Choose the product family
          ↓
Create related objects from that family
```

### Builder

```text
Choose properties step-by-step
          ↓
     Build object
```

---

# Quick Revision

## Behavioral Patterns

### Strategy

> Encapsulates interchangeable behaviors and allows them to be selected independently of the class using them.

### Observer

> Establishes a one-to-many relationship where a change in one object notifies its dependents.

### State

> Allows an object to change its behavior when its internal state changes, encapsulating state-specific logic.

### Chain of Responsibility

> Passes a request along a chain of handlers where each handler can decide to process it or pass it along.

### Null Object

> Provides a neutral object as a substitute for null, eliminating the need for null checks.

## Structural Patterns

### Decorator

> Dynamically adds responsibilities or behavior to an existing object without modifying its original class.

### Proxy

> Controls access to another object by providing a substitute or placeholder for it.

### Composite

> Composes objects into tree structures to represent part-whole hierarchies uniformly.

## Creational Patterns

### Builder

> Separates complex object construction from its representation and allows step-by-step configuration.

### Factory

> Encapsulates the decision of which concrete object to create and separates object creation from object usage.

### Abstract Factory

> Creates families of related objects without exposing their concrete implementations.

---

# 9. State Pattern

## Problem Statement

Suppose we have a product with multiple states:

- Draft
- Approved
- Published
- Archived

At each state, only certain operations are allowed.

For example:

```text
In Draft state:
  ✓ Edit
  ✓ Delete
  ✗ Publish
  ✗ Archive

In Published state:
  ✗ Edit
  ✗ Delete
  ✓ Archive
```

If we manage these restrictions directly in the product class with multiple if-else blocks, it becomes messy and difficult to maintain.

## Solution

Use the **State Pattern**.

Create a state interface:

```java
interface ProductState {
    void edit(Product product);
    void publish(Product product);
    void archive(Product product);
}
```

Implement each state:

```java
class DraftState implements ProductState {

    @Override
    public void edit(Product product) {
        System.out.println("Editing draft...");
    }

    @Override
    public void publish(Product product) {
        System.out.println("Publishing...");
        product.setState(new PublishedState());
    }

    @Override
    public void archive(Product product) {
        System.out.println("Cannot archive draft");
    }
}

class PublishedState implements ProductState {

    @Override
    public void edit(Product product) {
        System.out.println("Cannot edit published product");
    }

    @Override
    public void publish(Product product) {
        System.out.println("Already published");
    }

    @Override
    public void archive(Product product) {
        System.out.println("Archiving...");
        product.setState(new ArchivedState());
    }
}
```

The product maintains a reference to its state:

```java
class Product {

    private ProductState state;

    Product() {
        this.state = new DraftState();
    }

    public void setState(ProductState state) {
        this.state = state;
    }

    public void edit() {
        state.edit(this);
    }

    public void publish() {
        state.publish(this);
    }

    public void archive() {
        state.archive(this);
    }
}
```

Usage:

```java
Product product = new Product();
product.edit();      // Allowed
product.publish();   // Changes state to PublishedState
product.edit();      // Not allowed
product.archive();   // Allowed
```

## Key Idea

> **Encapsulate varying behavior based on the object's internal state, allowing the object to change its behavior as its internal state changes.**

---

# 10. Chain of Responsibility Pattern

## Problem Statement

Suppose we have a request that can be handled by multiple processors.

For example, a leave request in a company:

- HR can approve leaves up to 2 days
- Manager can approve leaves up to 5 days
- Director can approve leaves up to 10 days
- CEO can approve any leave

If a handler cannot process the request, it should pass it to the next handler in the chain.

If we hardcode these conditions, it becomes tightly coupled and difficult to modify.

## Solution

Use the **Chain of Responsibility Pattern**.

Create a handler interface:

```java
abstract class LeaveApprover {

    protected LeaveApprover nextApprover;

    public void setNextApprover(LeaveApprover nextApprover) {
        this.nextApprover = nextApprover;
    }

    public void approveLeave(int days) {
        if (canApprove(days)) {
            approve(days);
        } else if (nextApprover != null) {
            nextApprover.approveLeave(days);
        } else {
            System.out.println("Leave request rejected");
        }
    }

    protected abstract boolean canApprove(int days);
    protected abstract void approve(int days);
}
```

Implement concrete handlers:

```java
class HRApprover extends LeaveApprover {

    @Override
    protected boolean canApprove(int days) {
        return days <= 2;
    }

    @Override
    protected void approve(int days) {
        System.out.println("HR approved " + days + " days leave");
    }
}

class ManagerApprover extends LeaveApprover {

    @Override
    protected boolean canApprove(int days) {
        return days <= 5;
    }

    @Override
    protected void approve(int days) {
        System.out.println("Manager approved " + days + " days leave");
    }
}

class DirectorApprover extends LeaveApprover {

    @Override
    protected boolean canApprove(int days) {
        return days <= 10;
    }

    @Override
    protected void approve(int days) {
        System.out.println("Director approved " + days + " days leave");
    }
}
```

Build the chain:

```java
LeaveApprover hr = new HRApprover();
LeaveApprover manager = new ManagerApprover();
LeaveApprover director = new DirectorApprover();

hr.setNextApprover(manager);
manager.setNextApprover(director);

hr.approveLeave(3);  // Manager approves
hr.approveLeave(7);  // Director approves
hr.approveLeave(15); // Rejected
```

## Common Use Cases

- Approval systems (leave, purchase orders)
- Logging frameworks
- Exception/error handling pipelines
- Authentication/authorization systems
- Request processing in web frameworks
- Middleware chains
- Servlet filters
- Customer support escalation

## Key Idea

> **Pass a request along a chain of handlers where each handler can decide either to process the request or pass it to the next handler in the chain.**

---

# 11. Null Object Pattern

## Problem Statement

Suppose we have an operation that may return null:

```java
Employee employee = employeeService.getEmployee(id);

if (employee != null) {
    employee.work();
}
```

This pattern forces the client to check for null every time, which is:

- Repetitive
- Error-prone (easy to forget the null check)
- Clutters the code with null checks

## Solution

Use the **Null Object Pattern**.

Instead of returning null, create a null object with default behavior:

```java
interface Employee {
    void work();
}

class RealEmployee implements Employee {
    private String name;

    RealEmployee(String name) {
        this.name = name;
    }

    @Override
    public void work() {
        System.out.println(name + " is working");
    }
}

class NullEmployee implements Employee {

    @Override
    public void work() {
        // Default behavior - do nothing
        System.out.println("No employee to work");
    }
}
```

The service returns a null object instead of null:

```java
class EmployeeService {

    public Employee getEmployee(int id) {
        // If employee not found
        return new NullEmployee();
        
        // Instead of:
        // return null;
    }
}
```

Client code becomes simpler:

```java
Employee employee = employeeService.getEmployee(id);
employee.work();  // No need for null check
```

## Benefits

- Eliminates null checks
- Prevents `NullPointerException`
- Cleaner, more readable code
- Default behavior is explicit

## Key Idea

> **Provide an object with neutral/default behavior instead of null, eliminating the need for null checks.**

---

# 12. Composite Pattern

## Problem Statement

Suppose we have a file system with files and folders.

A folder can contain:
- Files
- Other folders

A file has a size, but a folder's size is the sum of all files and folders inside it.

If we try to represent this with separate classes:

```text
File (leaf)
├─ getSize()
└─ delete()

Folder (composite)
├─ getSize()        ← Sum of all contents
├─ delete()         ← Delete all contents
├─ add()
├─ remove()
└─ List of File/Folder
```

The problem is that a folder needs to treat files and folders differently, leading to:

- Type checking and casting
- Code duplication
- Tight coupling

## Solution

Use the **Composite Pattern**.

Create a common interface:

```java
interface FileSystemElement {
    int getSize();
    void delete();
}
```

Implement the leaf (File):

```java
class File implements FileSystemElement {

    private String name;
    private int size;

    File(String name, int size) {
        this.name = name;
        this.size = size;
    }

    @Override
    public int getSize() {
        return size;
    }

    @Override
    public void delete() {
        System.out.println("Deleting file: " + name);
    }
}
```

Implement the composite (Folder):

```java
class Folder implements FileSystemElement {

    private String name;
    private List<FileSystemElement> elements = new ArrayList<>();

    Folder(String name) {
        this.name = name;
    }

    public void add(FileSystemElement element) {
        elements.add(element);
    }

    public void remove(FileSystemElement element) {
        elements.remove(element);
    }

    @Override
    public int getSize() {
        int totalSize = 0;
        for (FileSystemElement element : elements) {
            totalSize += element.getSize();
        }
        return totalSize;
    }

    @Override
    public void delete() {
        System.out.println("Deleting folder: " + name);
        for (FileSystemElement element : elements) {
            element.delete();
        }
    }
}
```

Usage:

```java
Folder root = new Folder("root");

File file1 = new File("document.txt", 100);
File file2 = new File("image.jpg", 500);

Folder documents = new Folder("Documents");
documents.add(file1);

Folder pictures = new Folder("Pictures");
pictures.add(file2);

root.add(documents);
root.add(pictures);

System.out.println("Total size: " + root.getSize());  // 600
root.delete();  // Deletes all nested elements
```

## Structure

```text
         FileSystemElement (Component)
              ↑              ↑
              |              |
           File (Leaf)   Folder (Composite)
                            ↑
                            |
                    Contains FileSystemElement
```

## Common Use Cases

- File systems (files and folders)
- GUI components (containers and widgets)
- Organization hierarchies (departments and employees)
- Menu systems (menus and menu items)
- DOM trees (elements and containers)
- Graphics editors (shapes and groups)

## Key Idea

> **Compose objects into tree structures to represent part-whole hierarchies. This allows clients to treat individual objects and compositions of objects uniformly.**