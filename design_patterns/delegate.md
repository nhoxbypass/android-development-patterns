# Delegate design pattern

Delegate pattern is an object-oriented design pattern that allows object composition to achieve the same code reuse as inheritance.. Interface A with function `foo()`, class B implement A, then other class can use A to execute `foo()`.

=> Decoupling -> Reduce coupling between client and implementation

=> Hide implement detail

Ex: class `AbstractDatePickerDelegate` have 2 child class: `DatePickerSpinnerDelegate` (select date using spinner) and `DatePickerCalendarDelegate` (select date using calendar). This abstract class also implements `DatePickerDelegate` interface (for picking up a date (d/m/y)). 

So why these 2 class do not implement `DatePickerDelegate` interface directly? To avoid duplicate code.