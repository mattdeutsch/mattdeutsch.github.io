---
title: Isn't More Information Better?
date: 2025-07-09 13:00:00 -0500
categories: [modularity]
tags: [modularity, complexity, information-hiding]     # TAG names should always be lowercase
---

I recently was a part of a meeting with a non-engineer on our company's leadership team where we discussed efforts our engineering team was taking to make our codebase simpler. One of my teammates excitedly mentioned that after their work is complete, our systems will be able to function while sending less information to each other.

The member of the leadership team asked an extremely reasonable follow-up question: 
> "Isn't more information better?"
{: .prompt-tip }

This isn't a bad question - it's quite a good question, and cuts to the heart of good engineering. This question does, however, have an unambiguous answer, and that answer is "NO!".

## The Harm of Irrelevant Information

If you go to a barber to get a haircut, you don't want to have to share your social security card with them. Why not? What is the harm of requiring an irrelevant piece of information?

1. The barber shouldn't need this information - what could they possibly need to do with it? Even asking for it is a huge red flag.
1. You may not have your social security card handy at the time when you want a haircut.
1. You may not be a US citizen. This should not impact your ability to get a haircut.
1. You may value keeping that piece of information private (as you probably should).

Irrelevant information limits what you and your barber can do and incentivizes workaround behavior from you:

* Maybe you start carrying your social security card everywhere in case you decide you want a haircut later. 
* Maybe you learn that the barber doesn't perform much of a check to ensure that your social security card is genuine, so to make things easier, you fabricate a fake card just for the barber. 
* Maybe the barber wants to open a branch in Vancouver, but their system won't allow them because Canadians don't have social security cards.
* Maybe you decide this barber isn't worth using, since they're asking for too much, so you cut your hair yourself.

In practice, it's difficult to determine when data is or is not "irrelevant", and we usually make mistakes on the side of sharing too much data between systems instead of too little. The barber/SSN example is, I admit, a little too contrived.

## A Less Contrived Example

Here is an example of a mistake I used to commonly make. Suppose you need to write a function that calculates a user's age based on their date of birth. What's wrong with the following code?

```py
def calculate_age(user_id: str) -> int:
    user = grab_user_from_db(user_id)
    current_date = get_current_date()
    years_since_birth_year = current_date.year - user.dob.year
    has_had_birthday_this_year = (
        current_date.month > user.dob.month or 
        (current_date.month == user.dob.month and current_date.day >= user.dob.day)
    )
    if has_had_birthday_this_year:
        return years_since_birth_year
    else:
        return years_since_birth_year - 1
```

The logic is correct, but this code has a problem - it's asking for your social security number when it's giving you a haircut.

This function is about calculating someone's age, which requires some logic around dates. Why does it require the user's database ID? As written, this code will have the same problems as the barber:

* Other places will start carrying around user_id when they don't need it, in case they need to use this function. 
* If this function is needed in some other context where we don't have a DB connection, or we don't have a user object, it can't be reused.
* Maybe someone who just needs this function ends up creating a dummy user object in the DB, just so they can use this function.
* Or, the most likely outcome - the next time someone needs to calculate an age from a date of birth, they rewrite this logic themselves, losing out on the existing, tested method and potentially introducing bugs.

Here's a better version.

```py
def calculate_age_from_user_id(user_id: str) -> int:
    user = get_user_from_db(user_id)
    return calculate_age(user.dob)

def calculate_age(dob: date) -> int:
    current_date = get_current_date()
    years_since_birth_year = current_date.year - dob.year
    has_had_birthday_this_year = (
        current_date.month > dob.month or 
        (current_date.month == dob.month and current_date.day >= dob.day)
    )
    if has_had_birthday_this_year:
        return years_since_birth_year
    else:
        return years_since_birth_year - 1
```

Now `calculate_age` appropriately only asks for the information that it needs to do its job - a date of birth.

## But That's Not What They Meant

The question "Isn't more information better?" is clearly not referring to irrelevant information. It's referring to information that might help us gain insight into our product. Information that might be logged, or saved in event data, that can become fodder for data scientists or AI algorithms so that we can learn from them. What's important to recognize is that data reporting is *just another module in our system*, and the data reporting module should be asking for pretty much every scrap of information we have. Other modules should still require as little information sharing as possible.

## For Services, More Information Is Worse

In most contexts of life, more information sharing is better. It's better to know when your bus is scheduled to arrive than not to know. It's better to know what ingredients were used in packaged food. It's better to share feelings with a partner instead of swallowing them. Services are the exception to this rule.

If one entity needs to hand responsibility off to another to perform a task, it's better for that task to be as small as possible, with as little information shared between parties as possible. This is the case when writing software, as well as when cutting hair.
