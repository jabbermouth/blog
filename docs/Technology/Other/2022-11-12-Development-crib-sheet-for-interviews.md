# Development crib sheet for interviews

This article was written ahead of doing some interviews a while back. These are some of the development principles you should probably be familiar with that may come up at an interview. This is obviously not an exhaustive list or comprehensive guide.

## SOLID Principles

| Principle | Description | Purpose / Meaning | Example |
| --- | --- | --- | --- |
| Single Responsibility | A class should only have one responsibility in an application | To reduce complexity and minimise breaking changes | ✓ `GetData()`<br>✕ `GetDataAndOutput()` |
| Open/Closed | A class is open for extension but closed for modification | This means that a class should be extendable without needing to modify the original class. | Given a method such as `TotalArea(IList<IShape> shapes)`, logic within shouldn’t have specific area calculating logic but rather should call an `GetArea()` method of the `IShape`-compliant object. |
| Liskov Substitution | Any derived class should be usable where only a base class is required without the called method needing any awareness of the derived class | Any derived class can be passed as if it was the base class and the called method has no need to be aware of this. | Given:<br>`BaseClass()`<br>`DerivedClass() : BaseClass`<br>`CalledMethod(BaseClass param)`<br><br>Then:<br>`CalledMethod` should take any instance of a `BaseClass` or `DerivedClass` and treat them both as `BaseClass` |
| Interface Segregation | Specific interfaces should be used in preference to more generic ones | A consumer of an interface should not be forced to implement a method they don’t use | Instead of just `IVehicle`, also have interfaces like `ICar`, `IBike`, `IVan`, etc… which inherit from `IVehicle` |
| Dependency Inversion | Depend on abstractions instead of concrete elements | A method or class should not be tightly coupled to another class | ✓ `GetData(IDbConnection conn)`<br>✕ `GetData(SqlConnection conn)` |

## DRY

This means Don’t Repeat Yourself and simply means that, wherever possible, code should not be repeated. If code is repeating, consider it for extraction to another method, class or even library (such as a NuGet package).

## Boxing and Unboxing

This is the process of putting a typed object (e.g. an int) into an object type (boxing) and the reverse process of extracting (or unboxing) and (e.g. an int) from an object. This doesn’t do conversion so, if an int was boxed, you can’t unbox a short.

## Middleware

This provides a bridge to functionality that isn’t part of the application or the operating system. In .NET, examples include authentication and authorisation. It is usually registered within an application prior to to the main application process being called.

When added as part of a web application pipeline, the order in which it is added is important so, for example placing the UseSwaggerUI() before any request logging middleware means that requests to the Swagger UI won’t be logged where as placing it before would mean Swagger requests are also logged.

### Test Driven Development (TDD)

The simple description of this is writing tests before writing code. The concept being writing tests for expected behaviour before coding to help deliver expected requirements.

A good walkthrough of a TDD example can be found on YouTube.

## Behaviour Driven Development (BDD)

This takes TDD to the next level by using plain English and concrete example to describe the requirements. Using the Gerkin syntak, Given… When… Then… statements are written to describe the expected behaviour. This can then be directly mapped to unit, integration and, where appropriate, UI tests using tooling such as SpecFlow.

## Arrange, Act, Assert
When writing unit tests, typically a three stage approach is used.

**Arrange:** Setup the “system under test” and any required dependencies using mocks and fakes as required

**Act:** Perform the action(s) you are testing

**Assert:** Test the output of running the action(s) under “Act” and confirm the outputs match what is expected

Sometimes sections will be combined such as when using the assert to check for a particularly thrown exception.