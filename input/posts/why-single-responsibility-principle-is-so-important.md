---
Title:  "Why single responsibility principle is so important?"
Published:   2018-04-11
Tags: [design patterns, software craftsmanship, clean code, uncle bob, solid, single responsibility principle]
Comments: true
---

SOLID. One of the most known abbreviation in programming world. Five rules to rule them all. First question on every interview. About 403 000 000 Google search results for phrase "solid principles" (for comparison for "grasp principles" there are about 72 400 000 results). Knowing SOLID is the first step to transform from junior into regular. It's easy to say - they are popular. Very popular.
<!--more-->
That's awesome! Applying those rules really improves code quality. Young programmers should learn them as quickly as possible, so they have good habits from the beginning. Let's look at this from slightly different perspective though. We know, that applying those principles is good idea, but... why? Today I would like to focus on single responsibility principle and try to answer question - why this principle is so important?

Writing software is all about... reading. First we have to understand what there already is inside our application. Then we have to think about where we want to put our code. Then we will discover some strange dependencies. We will play Sherlock and discover what is really going on inside our codebase. Only after this process we can start writing. All above steps is reading, reading, reading.

Okay, let's be honest. It's not only reading. It's also, and unfortunately this is the **hard part**, understanding. We read lots of code, in order to understand it, in order to write new code someone will have to read, in order to understand it, in order to write some more code someone will have to read, in order to... I think you get my point.

Understanding is really the key here. Take a look at this code

    application.DoYourThing();

It's nice, isn't it? So clean and neat - just one line. There is only one problem though - what does this function do? Is it possible to understand this method right away? To get to know what this function does, we have to start digging. As I said before - it's all about reading and understanding. When something is simple, it's easier to understand how it works. One way to make things simpler, is to divide them into smaller pieces. Then we can analyze one, simple, small thing at a time. We don't have to remember, that 300 lines higher there is some `if`...

Look at your car. If you don't have a car, just... look at someone else's. It's highly modular, right? You can swap parts easily (I mean if you actually _know how_...). That's why every part serves only one purpose. Starting a car is a whole process. If you will look closely though, you'll see it's a cooperation of single-responsible parts. Fuel pump provides fuel. That's it! It's not like it also has to trigger spark plugs or do some other mechanical magic, so I can drive to work. Of course all those steps are being orchestrated by [ECU](https://en.wikipedia.org/wiki/Engine_control_unit) - it's job is to compose whole process.

Why microservices are so popular right now and we look at monoliths with genuine disgust? The first is (or rather should be) smaller and easier to change. The latter is big, messy and harder to maintain. We split big system into smaller chunks. We do this with complexity as well, because it's easier to comprehend many smaller modules, than one big-ass module.

That's why I think SRP is really important. Naturally it's not like we should apply it without deeper consideration, [whether it will make our code better or worse](https://hackernoon.com/you-dont-understand-the-single-responsibility-principle-abfdd005b137)!

What are your thoughts about this subject? Let me know in the comments below!