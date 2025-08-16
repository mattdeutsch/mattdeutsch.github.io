---
title: On Exception-al Code
date: 2025-08-16 13:00:00 -0500
categories: [code-quality]
tags: [exceptions, dijkstra-mentioned]     # TAG names should always be lowercase
---

You and your caravan are traveling through the desert. In need of water, one of you spots an oasis in the distance. They trust you and your new `check_oasis` function to investigate it. After a long camel ride there, it's bad news. The oasis was just a mirage.

Now comes the important part of your story - does your code return `None` or does it raise a `NoWaterFound` exception?

## Feels like this really isn't the important part of this story

Whether you return `None` here or whether you return `NoWaterFound`, the immediate surroundings of your program will look fairly similar. In either case, code that calls this oasis-checking logic will have to look like one of the following:

```py
# Implementation 1: Return None
def check_oasis(location: Location) -> Water | None:
    if location.has_no_water():
        return None
    return location.water()

# ... later:
water = check_oasis(location)
if water is None:
    die_of_thirst()
else:
    drink(water)
```

```py
# Implementation 2: Raise NoWaterFound
def check_oasis(location: Location) -> Water:
    if location.has_no_water():
        raise NoWaterFound
    return location.water()

# ... later:
try:
    water = check_oasis(location)
except NoWaterFound:
    die_of_thirst()

drink(water)
```

Both of these pieces of code will work. They both type-check. They will both operate fast enough, even if one slightly beats out the other. So what would guide you toward one implementation over the other?

## Uncaught exceptions halt execution

The biggest difference between the `None` implementation and the `NoWaterFound` implementation is that the version that raises an exception has the potential to halt a program at runtime. In most systems and languages that support exception handling, an uncaught exception causes the currently running program to terminate with an error code and a hopefully-readable error message saying where you were and what exactly went wrong.

There is good reason why uncaught exceptions do this. When truly unplanned behavior occurs, it's good if your program immediately crashes. Continuing to run when you already know things are going wrong could do more damage and cause a much stranger error later on down the stack (i.e. calling `drink` on a null object - who knows how the author of `drink` handled that case, and when it happens, how will we be able to tie it back to this particular oasis?).

Is `NoWaterFound` truly unplanned behavior for the `check_oasis` function?

## What context should your module have?

In this case, while I believe both implementations are fine, I feel comfortable arguing that `check_oasis` should return `None` instead of raising an error: raising an exception in this context assumes too much about the context where `check_oasis` will be called.

It's easy to imagine a context in which it's not an emergency for there to be no water at a location. Maybe you have enough water and are just checking to be safe. Maybe you're running this function millenia in the future, where water is plentiful, and `check_oasis` is used to survey locations for some database, not for immediate survival. In those cases, it would be annoying at best to have to catch `NoWaterException` just to continue operating. At worst, you are careless and forget to catch the error, and your customers see your application crash.

When no water being at a location is an emergency, it's simple enough for the code that calls `check_oasis` to raise that exception themselves. But from within the function, we just don't have enough information to make the call - all we have is a location.

## The third, worse option: Leveraging your exception handler

There's a third way to call the `check_oasis` logic that's worse than both alternatives written above. In this style, the exception `NoWaterFound` is not caught within local code, and instead goes to a global-level exception handler.

```py
# Implementation 3: No Try/Except
def check_oasis(location: Location) -> Water:
    if location.has_no_water():
        raise NoWaterFound
    return location.water()

# ... later:
drink(check_oasis(location))

# ... and in a global exception handler:
def handle_exception(exc: Exception):
    if isinstance(exc, NoWaterFound):
        die_of_thirst()
    ...
```

This type-checks[^1]. And from a local readability perspective, what a win! "Drink check oasis at location"! It's pretty much English. And it works in fewer lines of code, which has got to be better.

## Why you shouldn't

Writing code this way could make extending the logic of the no-water case difficult here, since while it's easy to raise an exception and move control flow to the global exception handler, the exception handler is not designed to return control flow back to the main program.

It will also surprise your teammates reading `drink(check_oasis(location))` that there's a branch in the code even occuring - the statement is written like it's an imperative statement to drink some water, but it's secretly an `if` statement that could lead you down a one-way road to your exception handler.

Using the exception handler essentially treats this as a GOTO, which has negatives we've known about since the time of Dijkstra's famous ["Go To Statement Considered Harmful"](https://homepages.cwi.nl/~storm/teaching/reader/Dijkstra68.pdf) paper in 1968.

All of this is another minor reason to prefer Implementation 1, which returns `None`, to Implementation 2, which raises and locally catches an exception. Implementation 1 ensures that you and your teammates avoid touching the global exception handler.

## So where should I raise an exception?

Honestly, wherever you want. The impacts discussed here are all fairly minimal, even in large programs. But if you're looking for my recommendation, exceptions should be raised exclusively when:

1. You are comfortable halting execution of the program, if this exception is not caught.
2. You have enough context on the situation in which you are running to know whether statement #1 is true.

#### Footnotes

[^1]:  ...so long as you're using a language like Python whose type-checker has no interest in what types of Exception a function might raise.