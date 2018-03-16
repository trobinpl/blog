---
Title:  "True objects have something more than getters and setters"
Published:   2018-03-22
Tags: [software craftsmanship, architecture, oop, object oriented programming, clean code]
Comments: true
---
Object-oriented languages are very popular. That's a fact. Writing object-oriented code is one of the most popular requirements for developers building enterprise systems. To be honest I don't think OOP is the only nor the best way to write all kinds of software, but it works pretty well in many applications. Yet, despite popularity and everyone talking about objects and object-oriented code, I still think many programmers think, that just by writing `class` they make their code OO.

Don't get me wrong. I did this (and probably still do from time to time ;-) ). I remember when I first started my adventure with programming. My first language I was learning was PHP. At more or less this time PHP 5 came out. Object-oriented version of PHP! I didn't quite understand why it's such a big deal, but I liked the way source code started to look like. I mean this:

    myObject->doWork();

Instead of 

    doWork(myObject);

For some reason it was really big change for me, just to use this new super-fancy operator `->`. Everyone was saying "you should write object-oriented code now!". I wanted to be good programmer, so I created a class, took all my functions and pasted them as methods. Voila! My code is now object-oriented! I was posting my code on PHP forum for others to give me feedback. It was really weird for me, when other said "This is not object! You're not doing it right!". But why? I create instance of my `class` sooo... I was doing objects! It took me couple of years to really get, what they tried to say.

