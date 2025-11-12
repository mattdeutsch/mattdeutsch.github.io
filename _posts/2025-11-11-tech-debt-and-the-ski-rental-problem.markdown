---
title: Tech Debt and the Ski Rental Problem
date: 2025-11-11 9:06:00 -0500
categories: [engineering-practices]
tags: [tech-debt, ski-rental, algorithms-to-live-by]
---

There’s a part of your codebase that burns you. You run into it all the time and each time you do, it takes time out of your day as you have to figure out how to work around it. Each new engineer unfortunate enough to chance across it must learn its arcane ways, lest they push bugs and break tests. It’s just bad code. Maybe if you had a week, you could rewrite it and fix it for everyone forever.

But it’s hard to figure out when you have an extra week to spend. Even if you had that week, is fixing this problem the best thing you can do with that week? Your coworkers in design and product certainly don’t care about it - they don’t know it exists, and they aren’t paid to have the technical know-how to even understand it. To them, spending a week rewriting code to solve an already-solved problem looks like wasting a week - or worse: the “bad” code, as it exists, works in production, and code that works in production can be precious. Why risk pushing new code that might introduce new bugs just to rewrite something that isn't even broken?

This is the problem of technical debt. When, as an organization, is it correct to have your engineers work on an internal problem like the one described above instead of implementing new features? There’s a correct answer that’s somewhere between “never” and “whenever”, which this post will attempt to answer.

## The Ski Rental Problem

But first, a toy problem from theoretical computer science. It goes like this:

You are going on a trip to a ski resort. Since you have unlimited PTO, you have literally no idea how long you plan on being on this trip - i.e., you cannot assign any probability distribution to the number of days you are going to be here. Could be 5 days. Could be 500 trillion years. Which is more likely? You don’t know{% fn %}.

While on this trip, you’d like to ski every day. Good thing this ski resort offers two options:

1. You can rent a pair of skis for 1 dollar a day.
2. You can purchase a pair of skis for 10 dollars, after which you never have to buy or rent another set of them.

Each day, you have to decide whether to rent or to buy. What should you do?

No, seriously, think about it. How do you even evaluate a strategy in the face of the massive amount of uncertainty in the number of days you’re going to be spending? You can’t calculate the “expected value” of a strategy - to do that, you’d need to have a probability distribution in mind.

## Comparison with Perfect

One way to try to limit your losses is to compare a ski rent/buy strategy with what someone who knew exactly what amount of time they were spending would have done. In other words, how does your strategy compare with someone who’s cheating{% fn %}, able to pay the perfect minimum amount of money every time, because they know the number of days they’re going to be spending in advance? This person would rent all 5 days of their 5 day trip, but would purchase skis on day 1 for their 500 trillion year trip, totaling just 10 dollars in expenses in this case.

An algorithm that gives you a reasonable chance at competing with this person looks like this:

- On days 1 through 9, rent.
- On day 10, if you’re still there, buy.

If your trip ends up being 9 days or fewer, you match the perfect algorithm exactly. If the trip ends up being longer than that, then your cheating adversary will pay 10 dollars, where you will pay 19 dollars. If you’re trying to talk up this strategy, you can argue that this strategy is never worse than the cheater’s by a factor of 2.

This is referred to as the “competitive ratio” of the algorithm, and it’s provable that no deterministic algorithm exists with a better competitive ratio than this. Thinking about competitive ratios, i.e. comparing your strategy against that of an all-knowing entity, is useful for “online” problems like this one, where your strategy has to contend with a stream of unpredictable incoming information{% fn %}.

## Applying the Ski Rental Problem to Your Life

What strikes me about the Ski Rental problem is how applicable it is to my everyday life. With slight modification, it can be re-skinned to apply to a ton of different scenarios:

1. I can deal with my half-broken dishwasher each day, though I’ll have to spend 10 extra minutes scrubbing the dishes before putting them into the machine, and I might have to run an extra cycle.
2. I can buy a better dishwasher.

Or,

1. I can deal with a lingering cold by taking OTC medication and hoping the problem goes away.
2. I can go to urgent care, get diagnosed, and have a deterministic amount of time to recovery.

Or, maybe,

1. I can deal with the pain of a problem in our codebase, even though that means added time spent in design and testing for new features.
2. I can re-implement the problematic code and save everyone this trouble forever.

The Ski Rental problem gives an answer to the question, “How much recurring pain is it reasonable to tolerate before committing to a known but costly permanent solution?” And the answer it gives is astonishingly concrete: Tolerate the pain until the cumulative pain comes to match the amount that the permanent solution costs. This way, you’ll never over-”pay” by more than a factor of 2.

## Wait, no, you haven’t factored in the “unknown distribution” part of the problem, which was crucial to your argument that this strategy made any sense at all

True, fair. Fair. If the problem is going to continue to cause pain for you forever, then the optimal strategy is to just commit to the permanent solution immediately. The optimal solution that minimizes dishwasher-related pain is to buy the new dishwasher right now… supposing that your current dishwasher will cause you problems indefinitely.

As soon as you consider that your problem may *go away on its own*, the Ski Rental problem becomes immediately applicable. Maybe you’ll eventually find a configuration that solves your issues with your dishwasher without having to buy a new one. Maybe you’ll feel better from your cold tomorrow morning. Maybe this piece of “tech debt” you’re bashing your head against today won’t need to be edited at all for the next 10 months.

Technically, all of your problems are temporary and unpredictable in length, as your life has both of those features as well (this is not a threat. crossing my fingers that all readers of my blog may live forever). But more pragmatically, it’s just difficult to predict which problems are going to remain relevant moving forward. Maybe you’ll naturally start using other modules instead of the one with the bad code you’re annoyed with right now. Maybe this whole project will be scrapped. Maybe the features that customers end up caring about just aren’t related to this one, so you can abandon it. Maybe Sam Altman creates AGI and we move to a post-scarcity society where you get to spend the rest of your days as a goat farmer in the mountains of Vermont. The amount of time that a particular piece of code will be relevant for is extremely unpredictable.

## Quantifying developer pain

If you’re still with me, then the only thing left to iron out before accepting that tech debt is similar to the Ski Rental problem is that it’s difficult to compare the recurring pain of tech debt to the amount of time it will take to implement a permanent solution.

My recommendation is to literally track the amount of time a particular piece of bad code steals from you. If it took you an extra hour today because it forced you into a painful debugging session, add it to a tally. If a coworker had to get a download from you on how this code actually worked, and that meeting took 1 hour, add 2 hours to the tally (one for you, one for your coworker). Once that time eclipses the estimate for how long it’d take to resolve this problem once and for all, it becomes smart to commit to solving it. You’ll never over-spend developer time by a factor of 2, and it’s theoretically impossible to do any better.

<hr>

{% footnotes %}
   {% fnbody %}
      And neither does your employer.
   {% endfnbody %}
   {% fnbody %}
      In this case, I suppose “cheating” means “planning out their ski trip with a predetermined number of days”, which might be unfair to call “cheating” and more fair to call “responsible trip planning”
   {% endfnbody %}
   {% fnbody %}
      One prominent example of where this type of thinking has had real-world application is the problem of which values of a cache to evict based on which values have been most recently accessed.
   {% endfnbody %}
{% endfootnotes %}
