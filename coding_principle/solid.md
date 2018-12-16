# SOLID

S.O.L.I.D is an acronym for the first 5 object-oriented design (OOD) principles for writing good code that was invented by Robert C. Martin, popularly known as Uncle Bob.

These principles, when combined together, make it easy for a programmer to develop software that are easy to understand, maintain and extend. They also make it easy for developers to avoid code smells, easily refactor code, and are also a part of the agile or adaptive software development.

## Single responsibility
  A class should have **only one** reason to change, only do the task which it has been designed.
  To reduce coupling, code easier to read and maintain.

## Open closed
  Class, module should be **open** for extension but **close** for modification.
  Help easy to reuse, extend and implement new feature without need to modify old code.

## Liskov Substitution
  Object must be replacable by instance of their sub-type without breaking correct.
  Help reuse code, class hierachy easy to understand.

## Interface Segregation
  A class should not implememnt interface that it does NOT use. We split a general interface to separate & specific interface.
  Help easy to refactor.

## Dependency inversion
  High-level module should NOT depend on low-level module but should depend on abstractions (interface, abstract class). Use dependency injection.
  Help reduce coupling.