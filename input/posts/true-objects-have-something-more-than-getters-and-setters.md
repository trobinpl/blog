---
Title:  "True objects have something more than getters and setters"
Published:   2018-03-21
Tags: [software craftsmanship, architecture, oop, object oriented programming, clean code]
Comments: true
---
Object-oriented languages are very popular. That's a fact. Writing object-oriented code is one of the most popular requirements for developers building enterprise systems. Yet, despite popularity and everyone talking about objects and object-oriented code, I still think many programmers think, that just by writing `class` they make their code OO.

<!--more-->

Don't get me wrong. I did this (and probably still do from time to time ;-) ). I remember when I first started my adventure with programming. My first language I was learning was PHP. At more or less this time PHP 5 came out. Object-oriented version of PHP! I didn't quite understand why it's such a big deal, but I liked the way source code started to look like. I mean this:

    myObject->doWork();

Instead of 

    doWork(myObject);

For some reason it was really big change for me to use this new super-fancy operator `->`. Everyone was saying "you should write object-oriented code now!". I wanted to be good programmer, so I created a class, took all my functions and pasted them as methods. Voila! My code is now object-oriented! I was posting my code on PHP forum for others to give me feedback. It was really weird for me, when they said "This is not object! You're not doing it right!". But why? I create instance of my `class` sooo... I'm doing objects! 

Then I switched to C# and did exactly the same thing. It really doesn't matter what language are you using - paradigm stays the same. It took me couple of years to realize what OOP truly is.

If I had to put this into the least words possible I would say - your object should not be just data structure, it should have also behaviors. Data structures are not bad! They are perfect solutions to certain problems. Serializing? Passing data from frontend to backend? Gathering some primitive types together? Go ahead - create data structure. Just let it be conscious choice.

Think about when did you use `private` keyword? If your answer is long time ago, **probably** you write good old fashioned functions. It's because your object should hide it's internals. There is absolutely no hiding anything without access modifiers other than `public`.

    ticket.Close();

Instead of

    ticket.State = TicketState.Closed;

Why this is so important. Let me make this example more real worldish.

    List<UserPermission> permissions = this.userRepository.GetPermissionsForUser(user);

    if(!permissions.Contains(UserPermission.CanCloseTicket) || !user.Roles.Contains(UserRoles.Admin)){
        // user don't have permissions to close ticket. Let's see if he is an admin
        List<UserRole> roles = this.userRepository.GetRolesForUser(user);
        if(!roles.Contains(UserRoles.Admin)){
            throw new NotAuthorizedException("User doesn't have permissions to close this ticket");
        }
    }

    if(ticket.State == TicketState.Closed){
        throw new NotSupportedException("Ticket is already closed");
    }

    if(ticket.Priority == 3 && string.IsNullOrEmpty(ticket.CloseReason)){
        return null;
    }

    ticket.Assignee = null;

    ticket.State = TicketState.Closed;

This example code is simple for now. It will grow though. You know it will. It always does. With more and more business requirements implemented like "we should check this before we set `Assignee` to null, so I will place it here!" understanding what this method is supposed to do would be really hard. If we wrap direct access to properties into methods, we end up with better code

    if(!user.HasPermission(UserPermission.CanCloseTicket) || !user.InRole(UserRoles.Admin)){
        throw new NotAuthorizedException("User doesn't have permissions to close this ticket");
    }

    if(ticket.AlreadyClosed()){
        throw new NotSupportedException("Ticket is already closed");
    }

    if(ticket.ShouldHaveCloseReason() && string.IsNullOrEmpty(ticket.CloseReason)){
        return null;
    }

    ticket.RemoveAssignee();

    ticket.State = TicketState.Closed;

By no means this code is perfect. Just better. You can read it and don't have to decode what it does. There are **names** for operation being performed in closing ticket process.

It's easy to imagine lines like `if(ticket.Priority == 3 && string.IsNullOrEmpty(ticket.CloseReason)){` would be spread across whole system. What if we would like to introduce another condition to check? We would have to change every occurrence of this line. It's very plausible we would miss at least one. What if I would like to change `Priority` enum to something calculated on the fly? It gets worse and worse. Internal representation of data should not leak outside, because it can change dramatically. Being dependent of internals is asking for troubles.

What do you think about this? Let me know in comments below!
