---
Title:  "True objects have something more than getters and setters"
Published:   2018-03-22
Tags: [software craftsmanship, architecture, oop, object oriented programming, clean code]
Comments: true
---
Object-oriented languages are very popular. That's a fact. Writing object-oriented code is one of the most popular requirements for developers building enterprise systems. Yet, despite popularity and everyone talking about objects and object-oriented code, I still think many programmers think, that just by writing `class` they make their code OO.

Don't get me wrong. I did this (and probably still do from time to time ;-) ). I remember when I first started my adventure with programming. My first language I was learning was PHP. At more or less this time PHP 5 came out. Object-oriented version of PHP! I didn't quite understand why it's such a big deal, but I liked the way source code started to look like. I mean this:

    myObject->doWork();

Instead of 

    doWork(myObject);

For some reason it was really big change for me to use this new super-fancy operator `->`. Everyone was saying "you should write object-oriented code now!". I wanted to be good programmer, so I created a class, took all my functions and pasted them as methods. Voila! My code is now object-oriented! I was posting my code on PHP forum for others to give me feedback. It was really weird for me, when they said "This is not object! You're not doing it right!". But why? I create instance of my `class` sooo... I'm doing objects! 

Then I switched to C# and did exactly the same thing. It really doesn't matter what language are you using - paradigm stays the same. It took me couple of years to realize what OOP truly is.

If I had to put this into the least words possible I would say - your object should not be just data structure, it should have also behaviors. Data structures are not bad! They are perfect solution to certain problems. Serializing? Passing data from frontend to backend? Gathering some primitive types together? Go ahead - create data structure. Just let it be conscious choice.

Think about when did you use `private` keyword? If your answer is long time ago, **probably** you write good old fashioned functions. Your object should hide how he stores data. Only thing visible from outside world should be public API of public methods. Changing internal state can be possible only through this API - never through accessing property directly.

    ticket.Close();

Instead of

    ticket.State = TicketState.Closed;

Why this is so important. Let me make this example more real worldish.

    if(!user.Permissions.HasPermission(UserPermission.CanCloseTickets)){
        throw new NotAuthorizedException("User doesn't have permissions to close this ticket")
    }

    if(ticket.State == TicketState.Closed){
        throw new NotSupportedException("Ticket is already closed");
    }

    

    ticket.State = TicketState.Closed;

---------------------------------------------------------------------------------------------
I'm sure everyone heard following terms: encapsulation, abstraction, inheritance and polymorphism. Did you ever take a minute to think what they really mean? I discovered that understanding the term itself is one thing, but actually _using_ it is completely different story.

Good object should not be just data structure. It should encapsulate both data **and** methods. Changing data should be possible only through object's methods. It is really important, because object knows best how to stay in cohesive state. It gives us API to play with - that's it! Let's look at this from real-life perspective. When you turn on your computer, do you just press one button or have to boot every component by yourself? Someone designed an API for you - one elegant button - so you don't have to deal with this boring low-level stuff you don't want to know anything about.

It doesn't matter if you change any of computer's parts. It is still switched on using the same button. In programming world this would mean, that no matter how you decide to represent your data _inside_ your object, it is still doing the same job. You can switch from `array[]` to `List<T>`, but from business perspective it's just technical detail. No business process changes if you do that switch, doesn't it? Keep internal stuff internal and expose public API.

Abstraction, inheritance and polymorphism are also very important, but I want to talk about something a little bit different. I bet every programmer faced problem of deciding whether to use `int`, `double`, `long` or `decimal`. It is valid and important problem. I just think that we are trying to solve it too soon.

What is more important - correct data type or understanding what this method we're writing should really do? It is of course rhetorical question. We put so much effort into this technical stuff, often not understanding problem we want to solve.

I talked about encapsulation, but I don't want to go further with abstraction, inheritance and polymorphism. I want to focus on something else. In my opinion this is more related with topic of this post, than those terms.

Have you ever been wondering whether you should use `int` or `long`? Maybe `float` would be more appropriate? It's absolutely valid and important question. Answering it often includes thinking about needed precision or memory usage. There is one more option though. You can create your own type, right? You can wrap this value into meaningful class. You can abstract returning type away making it easier to change if you made wrong choice at the very beginning. 

The same is for input parameters. Passing bunch of primitives into function doesn't tell anything about what they truly represent. Are they connected with each other somehow? Like for example `string firstname, string lastname`? Or maybe they are completely independent? I personally think that passing `PersonData personData` tells better story.

To clarify - I don't recommend changing every method not to accept primitives and wrapping everything into complex types, even if we want to pass simple `int count`. Overengineering is just as bad as no engineering at all.

