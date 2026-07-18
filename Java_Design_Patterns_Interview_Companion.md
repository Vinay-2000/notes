# Java Design Patterns - Interview Companion

> Companion notes to the main cheat sheet.
>
> Focus: - Tricky interview questions - Similar pattern comparisons -
> Spring/Java framework examples

------------------------------------------------------------------------

# 1. Singleton

## Tricky Questions

### Why is Singleton considered an anti-pattern?

-   Introduces global state.
-   Harder to unit test.
-   Creates hidden dependencies.
-   Dependency Injection is usually preferred today.

### Best implementation?

-   Enum Singleton (most robust)
-   Holder pattern (most common)

### Spring Interview

Is Spring Singleton the same as GoF Singleton?

**No.**

GoF: - One instance per ClassLoader.

Spring: - One instance per IoC Container.

------------------------------------------------------------------------

# 2. Factory Method

## Tricky Questions

### Factory vs Simple Factory

Simple Factory - Not a GoF pattern - Usually static - Uses if/switch

Factory Method - Official GoF - Uses inheritance - Subclass decides
creation

### Factory vs Builder

Factory - Chooses WHICH object.

Builder - Decides HOW to construct.

### Spring Examples

-   BeanFactory
-   ApplicationContext
-   Calendar.getInstance()

------------------------------------------------------------------------

# 3. Abstract Factory

## Tricky Questions

### Factory vs Abstract Factory

Factory - One product.

Abstract Factory - Family of related products.

### Weakness

Adding a new product type requires modifying every factory.

### Spring Examples

-   ApplicationContext
-   BeanFactory

------------------------------------------------------------------------

# 4. Builder

## Tricky Questions

### Why Builder instead of constructors?

-   Readable
-   Optional parameters
-   Immutable objects

### Builder vs Factory

Factory chooses object.

Builder builds object.

### Spring/Java Examples

-   Lombok @Builder
-   HttpRequest.newBuilder()
-   UriComponentsBuilder

------------------------------------------------------------------------

# 5. Prototype

## Tricky Questions

### Shallow vs Deep Copy

Shallow - Reference fields are shared.

Deep - Nested objects are copied.

### Why avoid Cloneable?

-   Awkward API
-   Shallow copy
-   Protected clone()
-   Copy constructors are usually preferred.

### Spring Example

Prototype bean scope.

------------------------------------------------------------------------

# 6. Adapter

## Tricky Questions

### Adapter vs Strategy

Adapter - Fixes incompatible interfaces.

Strategy - Swaps algorithms.

### Adapter vs Facade

Adapter - One object. - Interface translation.

Facade - Many objects. - Simplifies subsystem.

### Java Examples

-   InputStreamReader
-   Arrays.asList()
-   Spring HandlerAdapter

------------------------------------------------------------------------

# 7. Decorator

## Tricky Questions

### Decorator vs Adapter

Decorator - Same interface - Adds behavior

Adapter - Different interface - Converts behavior

### Decorator vs Chain

Decorator - Always delegates.

Chain - May stop delegation.

### Java Examples

-   BufferedInputStream
-   BufferedReader
-   Spring AOP

------------------------------------------------------------------------

# 8. Facade

## Tricky Questions

### Facade vs Adapter

Facade - Simplifies API

Adapter - Changes API

### Java Examples

-   JdbcTemplate
-   SLF4J
-   Service Layer

------------------------------------------------------------------------

# 9. Strategy

## Tricky Questions

### Strategy vs State

Strategy - Client chooses algorithm.

State - Object changes behavior internally.

### Strategy vs Command

Strategy - Algorithm

Command - Request/Action

### Java Examples

-   Comparator
-   AuthenticationProvider
-   Payment gateway selection

------------------------------------------------------------------------

# 10. Observer

## Tricky Questions

### Observer vs Pub/Sub

Classic Observer - Usually synchronous - Direct reference to observers

Modern Pub/Sub - Often asynchronous - Broker/event bus

### Memory Issue

Observers should be removed to avoid memory leaks.

### Spring Examples

-   ApplicationEventPublisher
-   @EventListener

------------------------------------------------------------------------

# 11. Command

## Tricky Questions

### Command vs Strategy

Command - Encapsulates action.

Strategy - Encapsulates algorithm.

### Biggest Advantage

Supports: - Retry - Queue - Undo - Scheduling - Async

### Java Examples

-   Runnable
-   ExecutorService
-   ScheduledExecutorService

------------------------------------------------------------------------

# 12. State

## Tricky Questions

### State vs Strategy

Strategy - External choice.

State - Internal transition.

### Real Examples

-   Order status
-   ATM
-   Workflow engines

------------------------------------------------------------------------

# 13. Template Method

## Tricky Questions

### Why final template method?

To prevent subclasses from changing algorithm order.

### Template vs Strategy

Template - Inheritance - Compile-time variation

Strategy - Composition - Runtime variation

### Java Examples

-   JdbcTemplate
-   HttpServlet
-   AbstractList

------------------------------------------------------------------------

# 14. Chain of Responsibility

## Tricky Questions

### Chain vs Decorator

Decorator - Always reaches final object.

Chain - Can stop processing.

### Chain vs Strategy

Chain - Multiple handlers

Strategy - One algorithm

### Spring Examples

-   Spring Security Filter Chain
-   Servlet Filters
-   Netty Pipeline

------------------------------------------------------------------------

# Ultimate Comparison Table

  Pattern                   One-Line Memory Trick
  ------------------------- -------------------------------
  Singleton                 One instance
  Factory                   Create object
  Abstract Factory          Create object families
  Builder                   Build step-by-step
  Prototype                 Copy object
  Adapter                   Translate interface
  Decorator                 Add behavior
  Facade                    Hide complexity
  Strategy                  Swap algorithm
  Observer                  Notify subscribers
  Command                   Action as object
  State                     Behavior changes with state
  Template Method           Skeleton algorithm
  Chain of Responsibility   Pass request through handlers

------------------------------------------------------------------------

# Spring Mapping (Must Remember)

  Spring / Java Component        Pattern
  ------------------------------ --------------------------
  @Component (default scope)     Singleton
  BeanFactory                    Factory
  ApplicationContext             Abstract Factory
  Lombok @Builder                Builder
  Prototype Bean Scope           Prototype
  HandlerAdapter                 Adapter
  BufferedInputStream            Decorator
  JdbcTemplate                   Facade + Template Method
  Comparator                     Strategy
  @EventListener                 Observer
  Runnable / ExecutorService     Command
  Spring Security Filter Chain   Chain of Responsibility
