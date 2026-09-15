# Design Patterns

A collection of commonly used **Object-Oriented Design Patterns** implemented and explained in Java.

## Patterns Covered

### Behavioral Patterns
- [Strategy Pattern](#1-strategy-pattern)
- [Observer Pattern](#2-observer-pattern)
- [State Pattern](#9-state-pattern)
- [Chain of Responsibility Pattern](#10-chain-of-responsibility-pattern)
- [Null Object Pattern](#11-null-object-pattern)
- [Command Pattern](#18-command-pattern)
- [Iterator Pattern](#19-iterator-pattern)
- [Mediator Pattern](#20-mediator-pattern)
- [Visitor Pattern](#21-visitor-pattern)

### Structural Patterns
- [Decorator Pattern](#3-decorator-pattern)
- [Proxy Pattern](#8-proxy-design-pattern)
- [Composite Pattern](#12-composite-pattern)
- [Adapter Pattern](#13-adapter-pattern)
- [Facade Pattern](#14-facade-pattern)
- [Bridge Pattern](#15-bridge-pattern)
- [Flyweight Pattern](#22-flyweight-pattern)

### Creational Patterns
- [Builder Pattern](#4-builder-pattern)
- [Factory Pattern](#5-factory-pattern)
- [Abstract Factory Pattern](#6-abstract-factory-pattern)
- [Prototype Pattern](#16-prototype-pattern)
- [Singleton Pattern](#17-singleton-pattern)

### Comparisons
- [Builder vs Decorator](#builder-vs-decorator)
- [Proxy vs Decorator](#proxy-vs-decorator)
- [Facade vs Proxy](#facade-vs-proxy)
- [Facade vs Adapter](#facade-vs-adapter)
- [Bridge vs Strategy](#bridge-vs-strategy)
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

### Command

> Encapsulates a request as an object, allowing parameterization of clients with different requests, queuing, and support for undo/redo operations.

### Iterator

> Provides a way to access elements of a collection sequentially without exposing its underlying representation.

### Mediator

> Defines an object that encapsulates how a set of objects interact, promoting loose coupling by keeping objects from referring to each other explicitly.

### Visitor

> Represents an operation to be performed on elements of an object structure, allowing new operations to be added without changing the structure.

## Structural Patterns

### Decorator

> Dynamically adds responsibilities or behavior to an existing object without modifying its original class.

### Proxy

> Controls access to another object by providing a substitute or placeholder for it.

### Composite

> Composes objects into tree structures to represent part-whole hierarchies uniformly.

### Adapter

> Converts the interface of a class into another interface clients expect, making incompatible interfaces work together.

### Facade

> Provides a unified, simplified interface to a set of interfaces in a subsystem, shielding clients from complexity.

### Bridge

> Decouples an abstraction from its implementation so the two can vary independently, preventing cartesian explosion.

### Flyweight

> Uses sharing to support large numbers of fine-grained objects efficiently by sharing common state among multiple objects.

## Creational Patterns

### Builder

> Separates complex object construction from its representation and allows step-by-step configuration.

### Factory

> Encapsulates the decision of which concrete object to create and separates object creation from object usage.

### Abstract Factory

> Creates families of related objects without exposing their concrete implementations.

### Prototype

> Creates new objects by copying an existing object instead of creating from scratch, improving performance for expensive operations.

### Singleton

> Restricts instantiation of a class to a single object and provides a global point of access to it.

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
---

# 13. Adapter Pattern

## Problem Statement

Suppose we have a legacy system with an existing interface that a client uses.

Now we have a new third-party library or module that we want to integrate, but its interface is incompatible.

For example:

```text
Client expects: PaymentProcessor interface with process(amount)

Existing Adapter: PaymentGateway with pay(value)

Incompatible ✗
```

We cannot modify either the client code or the third-party library.

## Solution

Use the **Adapter Pattern** to create a bridge between the incompatible interfaces.

Create an adapter that wraps the incompatible object and translates calls to the expected interface:

```java
interface PaymentProcessor {
    void process(int amount);
}

class PaymentGateway {
    public void pay(int value) {
        System.out.println("Processing payment: " + value);
    }
}

class PaymentProcessorAdapter implements PaymentProcessor {

    private PaymentGateway gateway;

    PaymentProcessorAdapter(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    @Override
    public void process(int amount) {
        // Adapt the call from process() to pay()
        gateway.pay(amount);
    }
}
```

Now the client can use the adapter:

```java
PaymentGateway gateway = new PaymentGateway();
PaymentProcessor processor = new PaymentProcessorAdapter(gateway);

processor.process(100);  // Works seamlessly
```

## Two Types of Adapters

### 1. Class Adapter (Using Inheritance)

```java
class PaymentProcessorAdapter extends PaymentGateway implements PaymentProcessor {

    @Override
    public void process(int amount) {
        pay(amount);  // Call inherited method
    }
}
```

### 2. Object Adapter (Using Composition - Preferred)

```java
class PaymentProcessorAdapter implements PaymentProcessor {

    private PaymentGateway gateway;

    PaymentProcessorAdapter(PaymentGateway gateway) {
        this.gateway = gateway;  // HAS-A relationship
    }

    @Override
    public void process(int amount) {
        gateway.pay(amount);
    }
}
```

## Key Idea

> **Convert the interface of a class into another interface clients expect. This allows classes with incompatible interfaces to work together.**

---

# 14. Facade Pattern

## Problem Statement

Suppose we have a complex system with multiple interrelated components:

```text
Computer System
├── CPU
├── Memory
├── HardDrive
├── GraphicsCard
├── PowerSupply
└── Cooling
```

Starting a computer requires coordinating all these components:

```java
cpu.start();
memory.initialize();
hardDrive.spin();
graphicsCard.enable();
powerSupply.activate();
cooling.start();
```

A client that wants to use the computer should not need to know all these internal details.

## Solution

Use the **Facade Pattern** to provide a simplified interface to the complex subsystem.

```java
class Computer {

    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;
    private GraphicsCard graphicsCard;
    private PowerSupply powerSupply;
    private Cooling cooling;

    Computer() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
        this.graphicsCard = new GraphicsCard();
        this.powerSupply = new PowerSupply();
        this.cooling = new Cooling();
    }

    // Simple interface that hides complexity
    public void start() {
        cpu.start();
        memory.initialize();
        hardDrive.spin();
        graphicsCard.enable();
        powerSupply.activate();
        cooling.start();
    }

    public void shutdown() {
        cpu.shutdown();
        memory.shutdown();
        hardDrive.stop();
        graphicsCard.disable();
        powerSupply.deactivate();
        cooling.stop();
    }
}
```

Client code becomes simple:

```java
Computer computer = new Computer();
computer.start();     // Handles all complexity internally
computer.shutdown();
```

## Layered Facades

Facades can use other facades:

```java
class SmartHome {

    private Computer computer;
    private ElectricalSystem electrical;
    private HVAC hvac;

    public void morningMode() {
        computer.start();
        electrical.enableLights();
        hvac.setTemperature(21);
    }
}
```

## Key Idea

> **Provide a unified, simplified interface to a set of interfaces in a subsystem. This shields clients from subsystem complexity and promotes loose coupling.**

---

# 15. Bridge Pattern

## Problem Statement

Suppose we have a hierarchy of devices:

```text
Device
├── Mobile
│   ├── Android
│   └── iOS
└── Laptop
    ├── Windows
    └── Mac
```

With inheritance, this leads to an exponential explosion of classes.

For example:
- `AndroidPhone`
- `iPhonePhone`
- `WindowsLaptop`
- `MacLaptop`

The problem worsens when we need to add new dimensions (e.g., screen types, processors, etc.).

This is known as the **Cartesian Product Problem**.

## Solution

Use the **Bridge Pattern** to separate abstraction from implementation.

Create separate hierarchies:

### Implementation Hierarchy (Operating System)

```java
interface OS {
    void bootUp();
    void shutdown();
}

class AndroidOS implements OS {
    @Override
    public void bootUp() {
        System.out.println("Booting Android...");
    }

    @Override
    public void shutdown() {
        System.out.println("Shutting down Android...");
    }
}

class iOSOS implements OS {
    @Override
    public void bootUp() {
        System.out.println("Booting iOS...");
    }

    @Override
    public void shutdown() {
        System.out.println("Shutting down iOS...");
    }
}
```

### Abstraction Hierarchy (Device Type)

```java
abstract class Device {

    protected OS os;  // Bridge to implementation

    Device(OS os) {
        this.os = os;
    }

    public void powerOn() {
        os.bootUp();
    }

    public void powerOff() {
        os.shutdown();
    }

    public abstract void displaySpecs();
}

class Mobile extends Device {

    Mobile(OS os) {
        super(os);
    }

    @Override
    public void displaySpecs() {
        System.out.println("Mobile Device");
        powerOn();
    }
}

class Laptop extends Device {

    Laptop(OS os) {
        super(os);
    }

    @Override
    public void displaySpecs() {
        System.out.println("Laptop Device");
        powerOn();
    }
}
```

Usage:

```java
Device androidPhone = new Mobile(new AndroidOS());
androidPhone.displaySpecs();  // Mobile Device, Booting Android

Device macLaptop = new Laptop(new iOSOS());
macLaptop.displaySpecs();  // Laptop Device, Booting iOS
```

## Structure

```text
      Abstraction
      (Device)
      ↑  ↑
      |  |
   Mobile Laptop

      Implementation
      (OS)
      ↑  ↑
      |  |
   Android iOS
```

Instead of creating 4 classes, we now have 4 classes without cartesian explosion.

## Key Idea

> **Decouple an abstraction from its implementation so the two can vary independently.**

---

# 16. Prototype Pattern

## Problem Statement

Suppose we have an expensive object to create (e.g., loading from database, complex calculations).

If we need multiple copies of this object with slight variations, creating each from scratch is inefficient.

For example:

```java
class Document {
    String content;
    int fontSize;
    String fontFamily;
    
    Document() {
        // Expensive initialization
        this.content = loadFromDatabase();
        this.fontSize = 12;
    }
}
```

Every time we create a new document, it performs expensive operations.

## Solution

Use the **Prototype Pattern** to clone existing objects instead of creating from scratch.

Implement `Cloneable`:

```java
class Document implements Cloneable {

    String content;
    int fontSize;
    String fontFamily;

    Document() {
        // Expensive initialization only once
        this.content = "Default content";
        this.fontSize = 12;
        this.fontFamily = "Arial";
    }

    @Override
    public Document clone() {
        try {
            return (Document) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }

    public void setFontSize(int fontSize) {
        this.fontSize = fontSize;
    }
}
```

Usage:

```java
Document original = new Document();  // Expensive operation once

Document copy1 = original.clone();   // Cheap - just copies
copy1.setFontSize(14);

Document copy2 = original.clone();
copy2.setFontSize(16);
```

## Deep vs Shallow Copy

### Shallow Copy (Default)

```java
public Object clone() {
    return super.clone();  // Copies references
}
```

### Deep Copy (For Complex Objects)

```java
@Override
public Document clone() {
    try {
        Document cloned = (Document) super.clone();
        cloned.content = new String(this.content);  // Deep copy mutable objects
        return cloned;
    } catch (CloneNotSupportedException e) {
        throw new RuntimeException(e);
    }
}
```

## Benefits

- Avoid expensive object creation
- Efficient copying of complex objects
- Avoids dependencies on concrete classes

## Key Idea

> **Create new objects by copying an existing object (prototype) instead of creating from scratch, improving performance when object creation is expensive.**

---

# 17. Singleton Pattern

## Problem Statement

Suppose we need an object that should exist only once throughout the application's lifetime.

For example:
- Database connection pool
- Logger
- Configuration manager
- Thread pool

Creating multiple instances would waste resources and cause inconsistency.

## Solution

Use the **Singleton Pattern** to ensure only one instance exists.

### Eager Initialization

```java
class Singleton {

    // Created at class loading time
    private static final Singleton instance = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

**Pros:** Thread-safe by default, simple  
**Cons:** Instance created even if not used

### Lazy Initialization (Not Thread-Safe)

```java
class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

**Pros:** Instance created only when needed  
**Cons:** Not thread-safe in multi-threaded environments

### Synchronized Method (Thread-Safe but Slow)

```java
class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

**Pros:** Thread-safe  
**Cons:** Every call is synchronized (performance penalty)

### Double-Checked Locking (Industry Standard)

```java
class Singleton {

    private static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {  // First check (without lock)
            synchronized (Singleton.class) {
                if (instance == null) {  // Second check (with lock)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

## Why Use `volatile`?

The double-checked locking pattern has two critical issues:

### Issue 1: Instruction Reordering

```text
instance = new Singleton();

Behind the scenes, the CPU performs:
1. Allocate memory
2. Initialize fields  ← Reordering can happen here
3. Assign reference to instance

If reordering occurs:
1. Allocate memory
3. Assign reference to instance  (before initialization!)
2. Initialize fields

Another thread sees instance != null and uses it with default values
```

**Solution:** `volatile` keyword prevents instruction reordering.

### Issue 2: L1 Cache Synchronization

```text
Thread 1: Creates instance, it sits in L1 cache
          ↓ (hasn't synced with main memory yet)
Thread 2: Checks main memory, sees null
         Creates another instance ✗

Core-1 L1 Cache     Core-2 L1 Cache
    ↑                   ↑
    └─────────────────┬─────────────────┘
                      ↓
                  Main Memory
```

**Solution:** `volatile` keyword forces write to main memory and read from main memory.

### How `volatile` Solves Both Issues

```java
private static volatile Singleton instance;
```

**Properties of `volatile`:**

1. **Forces Memory Visibility:** Every read/write goes to main memory, not just cache
2. **Prevents Instruction Reordering:** All instructions before `volatile` complete before `volatile` executes, and all instructions after wait for `volatile` to complete

```
Before volatile ────┐
                    │ All complete
                    ↓
              volatile write/read
                    ↑
Before after volatile ┘
```

### Enum (Simplest Thread-Safe Solution)

```java
enum Singleton {
    INSTANCE;

    public void doSomething() {
        // Implementation
    }
}

// Usage
Singleton.INSTANCE.doSomething();
```

**Pros:**
- Thread-safe by default
- Serialization-safe
- Reflection-proof
- Simplest code

**Cons:** Less flexible than class-based approach

## Comparison of Singleton Implementations

| Approach | Thread-Safe | Lazy | Performance | Reflection-Safe |
|----------|:----------:|:----:|:-----------:|:---------------:|
| Eager | ✓ | ✗ | Fast | ✗ |
| Synchronized | ✓ | ✓ | Slow | ✗ |
| Double-Checked | ✓ | ✓ | Fast | ✗ |
| Enum | ✓ | ✗ | Fast | ✓ |

## Key Idea

> **Restrict the instantiation of a class to a single object and provide a global point of access to it.**

---

# Facade vs Proxy

| Aspect | Facade | Proxy |
|--------|--------|-------|
| **Purpose** | Simplifies a complex system | Controls access to an object |
| **Number of Objects** | Multiple subsystems | Single object |
| **Focus** | Reduces complexity | Controls behavior |
| **Relationship** | HAS-A with subsystems | HAS-A with wrapped object |
| **Use Case** | Hide system complexity | Add functionality (caching, auth) |

**Easy Way to Remember:**

**Facade:**
> "I'm a receptionist who handles all your interactions with the complex company behind me."

**Proxy:**
> "I'm a security guard who stands in front of one specific person and decides if you can talk to them."

---

# Facade vs Adapter

| Aspect | Facade | Adapter |
|--------|--------|---------|
| **Purpose** | Simplify complex interface | Make incompatible interfaces compatible |
| **Problem** | Too many subsystems to manage | Interfaces don't match |
| **Approach** | Hide complexity | Bridge the gap |
| **Client Expectation** | Wants simple interface | Wants to use existing interface |

**Easy Way to Remember:**

**Facade:**
> "Let me hide all this complexity behind a simple door."

**Adapter:**
> "Let me convert this plug so it works with your socket."

---

# Bridge vs Strategy

| Aspect | Bridge | Strategy |
|--------|--------|----------|
| **Focus** | Separates abstraction from implementation | Encapsulates interchangeable algorithms |
| **Problem** | Cartesian explosion of classes | Multiple ways to do the same task |
| **Dimensions** | Two independent hierarchies | One hierarchy with behavior variations |
| **Change Type** | Change implementation | Change algorithm at runtime |

**Easy Way to Remember:**

**Bridge:**
> "I have different device types AND different operating systems. I need both to vary independently."

**Strategy:**
> "I have one object but different ways it can behave. I choose the behavior based on context."

---

# 18. Command Pattern

## Problem Statement

Suppose we have a text editor with various operations:
- Save
- Print
- Undo
- Redo

If each operation is implemented directly in the editor class, it becomes:
- Tightly coupled
- Difficult to add new operations
- Hard to implement undo/redo functionality
- No separation between the invoker (button) and the receiver (editor)

For example:
```java
class TextEditor {
    public void save() { /* implementation */ }
    public void print() { /* implementation */ }
}

class Button {
    public void onClick() {
        editor.save();  // Tight coupling
    }
}
```

## Solution

Use the **Command Pattern** to encapsulate requests as objects.

### 1. Define Command Interface

```java
interface Command {
    void execute();
    void undo();
}
```

### 2. Create Concrete Commands

```java
class SaveCommand implements Command {

    private TextEditor editor;

    SaveCommand(TextEditor editor) {
        this.editor = editor;
    }

    @Override
    public void execute() {
        editor.save();
    }

    @Override
    public void undo() {
        editor.undoSave();
    }
}

class PrintCommand implements Command {

    private TextEditor editor;

    PrintCommand(TextEditor editor) {
        this.editor = editor;
    }

    @Override
    public void execute() {
        editor.print();
    }

    @Override
    public void undo() {
        // Print cannot be undone
        System.out.println("Cannot undo print");
    }
}
```

### 3. Receiver (TextEditor)

```java
class TextEditor {

    private String content = "Sample content";

    public void save() {
        System.out.println("Saving: " + content);
    }

    public void print() {
        System.out.println("Printing: " + content);
    }

    public void undoSave() {
        System.out.println("Undo save operation");
    }
}
```

### 4. Invoker (Button/Menu)

```java
class Button {

    private Command command;

    Button(Command command) {
        this.command = command;
    }

    public void click() {
        command.execute();
    }
}
```

### 5. Usage

```java
TextEditor editor = new TextEditor();

Command saveCmd = new SaveCommand(editor);
Command printCmd = new PrintCommand(editor);

Button saveButton = new Button(saveCmd);
Button printButton = new Button(printCmd);

saveButton.click();   // Calls SaveCommand.execute()
printButton.click();  // Calls PrintCommand.execute()
```

## Implementing Undo/Redo

```java
class CommandInvoker {

    private Stack<Command> history = new Stack<>();
    private Stack<Command> redoStack = new Stack<>();

    public void execute(Command command) {
        command.execute();
        history.push(command);
        redoStack.clear();  // Clear redo stack on new command
    }

    public void undo() {
        if (!history.isEmpty()) {
            Command command = history.pop();
            command.undo();
            redoStack.push(command);
        }
    }

    public void redo() {
        if (!redoStack.isEmpty()) {
            Command command = redoStack.pop();
            command.execute();
            history.push(command);
        }
    }
}
```

## Benefits

- Decouples invoker from receiver
- Encapsulates requests as objects
- Supports undo/redo functionality
- Allows queuing and logging of commands
- Easy to add new commands

## Common Use Cases

- Text editor operations (save, undo, redo)
- GUI buttons and menu items
- Transaction management
- Macro recording
- Queuing and scheduling
- Logging and auditing

## Key Idea

> **Encapsulate a request as an object, allowing you to parameterize clients with different requests, queue requests, and support undoable operations.**

---

# 19. Iterator Pattern

## Problem Statement

Suppose we have different collection types:
- Array
- LinkedList
- Set
- HashTable

Each collection has a different internal structure. If a client wants to iterate through any collection, they need to know:
- How the collection stores data
- How to access elements

This violates encapsulation and couples the client to the collection's implementation.

```java
// Client must know internal structure
if (collection instanceof Array) {
    for (int i = 0; i < array.length; i++) {
        process(array[i]);
    }
} else if (collection instanceof LinkedList) {
    Node current = linkedList.getFirst();
    while (current != null) {
        process(current.value);
        current = current.next;
    }
}
```

## Solution

Use the **Iterator Pattern** to provide a unified way to traverse collections.

### 1. Define Iterator Interface

```java
interface Iterator<T> {
    boolean hasNext();
    T next();
    void remove();
}
```

### 2. Define Collection Interface

```java
interface Collection<T> {
    Iterator<T> iterator();
}
```

### 3. Implement Iterator for Array

```java
class ArrayIterator<T> implements Iterator<T> {

    private T[] items;
    private int index = 0;

    ArrayIterator(T[] items) {
        this.items = items;
    }

    @Override
    public boolean hasNext() {
        return index < items.length;
    }

    @Override
    public T next() {
        if (!hasNext()) {
            throw new NoSuchElementException();
        }
        return items[index++];
    }

    @Override
    public void remove() {
        // Array removal is complex, might not support
    }
}
```

### 4. Implement Iterator for LinkedList

```java
class LinkedListIterator<T> implements Iterator<T> {

    private Node<T> current;

    LinkedListIterator(Node<T> head) {
        this.current = head;
    }

    @Override
    public boolean hasNext() {
        return current != null;
    }

    @Override
    public T next() {
        if (!hasNext()) {
            throw new NoSuchElementException();
        }
        T value = current.value;
        current = current.next;
        return value;
    }

    @Override
    public void remove() {
        // LinkedList removal is simpler
    }
}
```

### 5. Collection Implementations

```java
class MyArray<T> implements Collection<T> {

    private T[] items;

    MyArray(T[] items) {
        this.items = items;
    }

    @Override
    public Iterator<T> iterator() {
        return new ArrayIterator<>(items);
    }
}

class MyLinkedList<T> implements Collection<T> {

    private Node<T> head;

    @Override
    public Iterator<T> iterator() {
        return new LinkedListIterator<>(head);
    }
}
```

### 6. Unified Client Code

```java
Collection<Integer> array = new MyArray<>(new Integer[]{1, 2, 3});
Collection<Integer> linkedList = new MyLinkedList<>();

// Same client code works for any collection
for (Iterator<Integer> it = array.iterator(); it.hasNext(); ) {
    System.out.println(it.next());
}

for (Iterator<Integer> it = linkedList.iterator(); it.hasNext(); ) {
    System.out.println(it.next());
}

// Or using enhanced for loop (if Iterable is implemented)
for (Integer item : array) {
    System.out.println(item);
}
```

## Key Idea

> **Provide a way to access the elements of an object sequentially without exposing its underlying representation.**

---

# 20. Mediator Pattern

## Problem Statement

Suppose we have an online auction system with multiple components:
- `User`
- `Auction`
- `Bid`
- `Payment`
- `Notification`

These components interact with each other:

```
User ---> Auction <--- Bid
  ^         ^          ^
  |         |          |
  +-----+---+-----+----+
        |
    Payment
```

If every component communicates directly with every other:
- Complex interconnections (mesh network)
- Difficult to modify one component without affecting others
- Hard to reuse components
- Difficult to test

## Solution

Use the **Mediator Pattern** to centralize communication through a mediator.

```
User     Bid     Auction
  \      |       /
   \     |      /
    \    |     /
     \   |    /
      \  |   /
       \ |  /
        \| /
      AuctionMediator
        / | \
       /  |  \
    Payment Notification
```

### 1. Define Component Interface

```java
interface Component {
    void setMediator(Mediator mediator);
}
```

### 2. Define Mediator Interface

```java
interface Mediator {
    void placeBid(User user, int amount);
    void acceptBid(Auction auction, Bid bid);
    void processPayment(Payment payment);
    void notifyUser(User user, String message);
}
```

### 3. Implement Components

```java
class User implements Component {

    private String name;
    private Mediator mediator;

    User(String name) {
        this.name = name;
    }

    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    public void bid(int amount) {
        System.out.println(name + " wants to bid " + amount);
        mediator.placeBid(this, amount);
    }

    public void receiveNotification(String message) {
        System.out.println(name + " received: " + message);
    }
}

class Auction implements Component {

    private Mediator mediator;

    @Override
    public void setMediator(Mediator mediator) {
        this.mediator = mediator;
    }

    public void endAuction(Bid highestBid) {
        mediator.acceptBid(this, highestBid);
    }
}
```

### 4. Implement Concrete Mediator

```java
class AuctionMediator implements Mediator {

    private Bid highestBid;
    private List<User> users = new ArrayList<>();

    public void registerUser(User user) {
        user.setMediator(this);
        users.add(user);
    }

    @Override
    public void placeBid(User user, int amount) {
        if (highestBid == null || amount > highestBid.getAmount()) {
            highestBid = new Bid(user, amount);
            notifyAllUsers(user.getName() + " has the highest bid: " + amount);
        } else {
            notifyUser(user, "Bid is too low");
        }
    }

    @Override
    public void acceptBid(Auction auction, Bid bid) {
        System.out.println("Bid accepted: " + bid.getAmount());
        processPayment(new Payment(bid));
    }

    @Override
    public void processPayment(Payment payment) {
        System.out.println("Processing payment: " + payment.getAmount());
        notifyAllUsers("Payment processed successfully");
    }

    @Override
    public void notifyUser(User user, String message) {
        user.receiveNotification(message);
    }

    private void notifyAllUsers(String message) {
        for (User user : users) {
            user.receiveNotification(message);
        }
    }
}
```

### 5. Usage

```java
AuctionMediator mediator = new AuctionMediator();

User user1 = new User("Alice");
User user2 = new User("Bob");
User user3 = new User("Charlie");

mediator.registerUser(user1);
mediator.registerUser(user2);
mediator.registerUser(user3);

user1.bid(100);  // Alice bids 100
user2.bid(150);  // Bob bids 150
user1.bid(200);  // Alice bids 200
```

## Mediator vs Observer

| Aspect | Mediator | Observer |
|--------|----------|----------|
| **Relationship** | Many-to-Many | One-to-Many |
| **Coupling** | Objects coupled to mediator | Objects coupled to subject |
| **Communication** | Through mediator | Direct notifications |
| **Control** | Mediator controls interaction | Subject controls notification |
| **Use Case** | Complex interactions | Simple notifications |

## Key Idea

> **Define an object that encapsulates how a set of objects interact, promoting loose coupling by keeping objects from referring to each other explicitly and letting you vary their interaction independently.**

---

# 21. Visitor Pattern

## Problem Statement

Suppose we have a document structure with different elements:
- `Text`
- `Image`
- `Table`
- `Heading`

Now we need to perform various operations on these elements:
- Export to PDF
- Export to HTML
- Export to XML
- Count characters
- Validate content

If we add methods directly to each class:

```java
class Text {
    public void exportToPDF() { /* */ }
    public void exportToHTML() { /* */ }
    public void exportToXML() { /* */ }
    public int countCharacters() { /* */ }
}

class Image {
    public void exportToPDF() { /* */ }
    public void exportToHTML() { /* */ }
    public void exportToXML() { /* */ }
}
```

This leads to:
- **Class explosion** with many methods
- **Violates Single Responsibility Principle** - each class does too much
- **Difficult to add new operations** - requires modifying all classes

## Solution

Use the **Visitor Pattern** to add new operations without modifying the class structure.

### 1. Define Element Interface

```java
interface DocumentElement {
    void accept(Visitor visitor);
}
```

### 2. Define Visitor Interface

```java
interface Visitor {
    void visit(Text text);
    void visit(Image image);
    void visit(Table table);
}
```

### 3. Implement Concrete Elements

```java
class Text implements DocumentElement {

    private String content;

    Text(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}

class Image implements DocumentElement {

    private String fileName;

    Image(String fileName) {
        this.fileName = fileName;
    }

    public String getFileName() {
        return fileName;
    }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}

class Table implements DocumentElement {

    private int rows, columns;

    Table(int rows, int columns) {
        this.rows = rows;
        this.columns = columns;
    }

    public int getRows() { return rows; }
    public int getColumns() { return columns; }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}
```

### 4. Implement Concrete Visitors

```java
class PDFExporter implements Visitor {

    @Override
    public void visit(Text text) {
        System.out.println("Exporting text to PDF: " + text.getContent());
    }

    @Override
    public void visit(Image image) {
        System.out.println("Exporting image to PDF: " + image.getFileName());
    }

    @Override
    public void visit(Table table) {
        System.out.println("Exporting table to PDF: " + table.getRows() + "x" + table.getColumns());
    }
}

class HTMLExporter implements Visitor {

    @Override
    public void visit(Text text) {
        System.out.println("<p>" + text.getContent() + "</p>");
    }

    @Override
    public void visit(Image image) {
        System.out.println("<img src='" + image.getFileName() + "' />");
    }

    @Override
    public void visit(Table table) {
        System.out.println("<table rows=" + table.getRows() + " cols=" + table.getColumns() + " />");
    }
}

class CharacterCounter implements Visitor {

    private int totalCharacters = 0;

    @Override
    public void visit(Text text) {
        totalCharacters += text.getContent().length();
    }

    @Override
    public void visit(Image image) {
        // Images don't have characters
    }

    @Override
    public void visit(Table table) {
        // Tables don't directly have characters
    }

    public int getTotalCharacters() {
        return totalCharacters;
    }
}
```

### 5. Usage

```java
List<DocumentElement> document = new ArrayList<>();
document.add(new Text("Hello World"));
document.add(new Image("photo.jpg"));
document.add(new Table(3, 4));

// Export to PDF
Visitor pdfExporter = new PDFExporter();
for (DocumentElement element : document) {
    element.accept(pdfExporter);
}

// Export to HTML
Visitor htmlExporter = new HTMLExporter();
for (DocumentElement element : document) {
    element.accept(htmlExporter);
}

// Count characters
CharacterCounter counter = new CharacterCounter();
for (DocumentElement element : document) {
    element.accept(counter);
}
System.out.println("Total characters: " + counter.getTotalCharacters());
```

## Benefits

- Add new operations without modifying element classes
- Keeps operations and data structures separate
- Easy to add new visitors
- Follows Single Responsibility Principle

## Drawbacks

- Difficult to add new element types (requires modifying all visitors)
- More complex than direct methods

## Key Idea

> **Represent an operation to be performed on elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.**

---

# 22. Flyweight Pattern

## Problem Statement

Suppose we have a text editor displaying a document with thousands of characters.

Each character object stores:

```java
class Character {
    char value;           // e.g., 'A'
    int fontSize;         // 12
    String fontFamily;    // "Arial"
    int positionX;        // Where it appears on screen
    int positionY;        // Where it appears on screen
    Color color;          // Red
}
```

If the document has 100,000 characters, and most characters are 'A' in Arial font size 12, we're wasting memory storing:
- Duplicate font information
- Duplicate size information
- Duplicate color information

This can lead to **memory exhaustion**.

## Solution

Use the **Flyweight Pattern** to share common data.

Separate data into:
- **Intrinsic State** (shared): Font family, font size, color - data that is the same for many objects
- **Extrinsic State** (unique): Position X, Position Y - data that is different for each object

### 1. Flyweight Object (Shared)

```java
class CharacterFlyweight {

    private char value;
    private String fontFamily;
    private int fontSize;
    private Color color;

    CharacterFlyweight(char value, String fontFamily, int fontSize, Color color) {
        this.value = value;
        this.fontFamily = fontFamily;
        this.fontSize = fontSize;
        this.color = color;
    }

    public void render(int positionX, int positionY) {
        System.out.println("Rendering '" + value + "' at (" + positionX + "," + positionY + 
                          ") with " + fontFamily + " " + fontSize + "pt");
    }
}
```

### 2. Flyweight Factory

```java
class CharacterFactory {

    private Map<String, CharacterFlyweight> flyweights = new HashMap<>();

    public CharacterFlyweight getCharacter(char value, String fontFamily, int fontSize, Color color) {
        String key = value + fontFamily + fontSize + color;

        if (!flyweights.containsKey(key)) {
            flyweights.put(key, new CharacterFlyweight(value, fontFamily, fontSize, color));
        }

        return flyweights.get(key);
    }

    public int getFlyweightCount() {
        return flyweights.size();
    }
}
```

### 3. Context (Extrinsic State)

```java
class CharacterContext {

    private CharacterFlyweight flyweight;
    private int positionX;
    private int positionY;

    CharacterContext(CharacterFlyweight flyweight, int positionX, int positionY) {
        this.flyweight = flyweight;
        this.positionX = positionX;
        this.positionY = positionY;
    }

    public void render() {
        flyweight.render(positionX, positionY);
    }
}
```

### 4. Usage

```java
CharacterFactory factory = new CharacterFactory();

List<CharacterContext> document = new ArrayList<>();

// Add 100,000 'A' characters in Arial
for (int i = 0; i < 100000; i++) {
    CharacterFlyweight flyweight = factory.getCharacter('A', "Arial", 12, Color.BLACK);
    document.add(new CharacterContext(flyweight, i * 10, 0));
}

// Only 1 flyweight object for 'A' in Arial!
System.out.println("Flyweight objects created: " + factory.getFlyweightCount());  // Output: 1

// Render a few characters
for (int i = 0; i < 10; i++) {
    document.get(i).render();
}
```

## Memory Optimization

**Without Flyweight:**
```
100,000 Character objects × 50 bytes each = 5 MB
```

**With Flyweight:**
```
1 CharacterFlyweight object × 50 bytes = 50 bytes
100,000 CharacterContext objects × 8 bytes (reference + 2 ints) = 800 KB
Total: ~800 KB (84% memory savings!)
```

## When to Use Flyweight

1. **Many similar objects** - Thousands or millions
2. **Memory constraints** - Limited memory available
3. **Immutable shared data** - Intrinsic state doesn't change
4. **Can separate state** - Intrinsic and extrinsic data can be separated

## Common Use Cases

- Text editors (characters)
- Games (particles, trees, textures)
- Web browsers (fonts, styles)
- Map applications (map tiles)
- Caching of frequently used objects

## Key Idea

> **Use sharing to support large numbers of fine-grained objects efficiently by sharing common state between multiple objects, thereby reducing memory consumption.**

---

---

# Mediator vs Observer vs Proxy

| Aspect | Mediator | Observer | Proxy |
|--------|----------|----------|-------|
| **Purpose** | Centralize complex interactions | Notify dependents of changes | Control access to object |
| **Relationship** | Many-to-Many | One-to-Many | One-to-One |
| **Communication** | Through mediator | Direct notifications | Through proxy |
| **Coupling** | Objects coupled to mediator | Objects coupled to subject | Client coupled to proxy |
| **Use Case** | Complex business logic | Event notifications | Access control, caching |

**Easy Way to Remember:**

**Mediator:**
> "I'm the post office. Everyone sends their mail to me, and I deliver it to the right person."

**Observer:**
> "I'm a notification service. When something happens, I notify all who are interested."

**Proxy:**
> "I'm a gatekeeper for one specific object. I control who can access it and what they can do."

---

# Iterator vs Visitor

| Aspect | Iterator | Visitor |
|--------|----------|---------|
| **Purpose** | Traverse a collection | Perform operations on elements |
| **Focus** | How to access elements | What operations to perform |
| **Structure** | Single collection | Multiple different operations |
| **Element Modification** | May modify elements | Typically read-only |
| **Adding New Features** | No | Yes (new visitors) |

**Easy Way to Remember:**

**Iterator:**
> "I help you go through each item in a collection one by one, without knowing the collection's structure."

**Visitor:**
> "I let you add new things you can do with items, without changing the items themselves."

---

# Command vs Strategy

| Aspect | Command | Strategy |
|--------|---------|----------|
| **Purpose** | Encapsulate a request | Encapsulate an algorithm |
| **Focus** | What to do (execution) | How to do it (algorithm choice) |
| **Queue/Log** | Yes | No |
| **Undo/Redo** | Yes | No |
| **Timing** | Can be deferred | Immediate execution |

**Easy Way to Remember:**

**Command:**
> "I package up a request to do something, store it, and maybe do it later or undo it."

**Strategy:**
> "I let you pick how to solve a problem, but I solve it right now."

---

# Flyweight vs Object Pool

| Aspect | Flyweight | Object Pool |
|--------|-----------|-------------|
| **Purpose** | Share intrinsic state | Reuse expensive objects |
| **Sharing** | Direct sharing of objects | Reuse from pool |
| **State** | Immutable intrinsic state | Stateful objects |
| **Memory** | Reduced via sharing | Reduced via reuse |
| **Complexity** | Separate intrinsic/extrinsic | Manage pool lifecycle |
| **Use Case** | Immutable data (fonts, colors) | Expensive resources (threads, connections) |

---

# Design Pattern Quick Reference

## Pattern Selection Guide

### Need to handle complex object creation?
→ **Builder** (step-by-step configuration)  
→ **Factory** (hide creation logic)  
→ **Abstract Factory** (families of objects)  
→ **Prototype** (copy existing objects)

### Need to manage state or behavior?
→ **Strategy** (interchangeable algorithms)  
→ **State** (different behavior per state)  
→ **Command** (encapsulate requests with undo/redo)  
→ **Visitor** (add operations to structures)

### Need to simplify complexity?
→ **Facade** (simplify subsystem)  
→ **Adapter** (make incompatible interfaces work)  
→ **Bridge** (separate abstraction/implementation)  
→ **Composite** (tree structures)

### Need to manage object relationships?
→ **Observer** (one-to-many notifications)  
→ **Mediator** (many-to-many interactions)  
→ **Iterator** (traverse collections)  
→ **Proxy** (control access to one object)

### Need to optimize resources?
→ **Flyweight** (share intrinsic state)  
→ **Singleton** (one instance globally)  
→ **Object Pool** (reuse expensive objects)

### Need to add capabilities without modification?
→ **Decorator** (wrap objects dynamically)  
→ **Proxy** (add access control)

---

# Common Mistakes and Best Practices

## Singleton Pattern

❌ **Wrong:**
```java
private static Singleton instance;
public static Singleton getInstance() {
    if (instance == null) {
        instance = new Singleton();
    }
    return instance;
}
```
Not thread-safe!

✅ **Right:**
```java
private static volatile Singleton instance;
public static Singleton getInstance() {
    if (instance == null) {
        synchronized (Singleton.class) {
            if (instance == null) {
                instance = new Singleton();
            }
        }
    }
    return instance;
}
```
Or use Enum for simplicity.

## Observer Pattern

❌ **Wrong:**
```java
observer.update(this, null);  // Passing vague data
```

✅ **Right:**
```java
observer.update(this, new WeatherData(temp, humidity));  // Specific data
```

## Factory Pattern

❌ **Wrong:**
```java
String type = getUserInput();
if (type.equals("PDF")) {
    // Create PDF exporter
} else if (type.equals("Excel")) {
    // Create Excel exporter
}
// Repeated in multiple places
```

✅ **Right:**
```java
ExporterFactory factory = new ExporterFactory();
Exporter exporter = factory.createExporter(type);
exporter.export(data);
```

## Decorator vs Inheritance

❌ **Wrong:**
```java
class CoffeWithMilk extends Coffee {}
class CoffeWithSugarAndMilk extends Coffee {}
class CoffeWithWhippedCreamAndSugarAndMilk extends Coffee {}
// Class explosion!
```

✅ **Right:**
```java
Coffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
coffee = new WhippedCreamDecorator(coffee);
```

---
