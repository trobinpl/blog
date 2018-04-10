---
Title:  "Why single responsibility principle is so important?"
Published:   2018-02-24
Tags: [design patterns, software craftsmanship, clean code]
Comments: true
---

SOLID. One of the most known abbreviation in programming world. Five rules to rule them all. First question on every interview. About 403 000 000 Google search results for phrase "solid principles" (for comparison for "grasp principles" there are about 72 400 000 results). Knowing SOLID is the first step to transform from junior into regular. It's easy to say - they are popular. Very popular.

It's awesome! Applying those rules really improves code quality. Young programmers should learn them as quickly as possible, so they have good habits from the start. Let's look at this from slightly different perspective though. We know, that applying those principles is good idea, but... why? Today I would like to focus on single responsibility principle and try to answer question - why this principle is so important?

Writing software is all about... reading. First we have to understand what there already is inside our application. Then we have to think about where we want to put our code. Then we will discover some strange dependencies. We will play Sherlock and discover what is really going on inside codebase. Only after this process we can start writing. All above steps is reading, reading, reading.

Okay, let's be honest. It's not only reading. It's also, and unfortunately this is the **hard part**, understanding. We read lots of code, in order to understand it, in order to write new code someone will have to read, in order to understand it, in order to write some more code someone will have to read, in order to... I think you get my point.

Understanding is really the key here. Take a look at this code

    application.DoYourThing();

It's nice, isn't it? So clean and neat - only one line. Only one problem - what does it do? Is it possible to understand this method?





It's fair to say that in most of our systems there are many business processes handled. Those processes consist multiple steps.  