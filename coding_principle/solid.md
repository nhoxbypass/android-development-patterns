# SOLID

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