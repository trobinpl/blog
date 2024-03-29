---
Title:  "Here's why you should do dependency injection"
Published:   2018-03-15
Tags: [design patterns, software craftsmanship, architecture, dependency injection, inversion of control, unit test]
Comments: true
---
On almost every job interview we can hear this question - "What can you tell me about dependecy injection?". Standard response contains at least one dependency injection framework name and general description with required sentence "it improves testability". Sometimes people say this, without really understandig what it means. How exactly dependency injection improves testability?
<!--more-->
Recently in project I'm working on I got to write some new piece of code. Whole system is, unfortunately, one big ball of mud (in fact it's biggest ball of mud I've seen up to this very day...). I was really excited to finally be able to write something new and not to be forced to deal with really poor legacy code. This was also one of those rare moments when I said to myself "okay, you have quite some time to write this fairly simple code, let's do it TDD way!". I didin't suspect any problems, because what I was writing was a simple procedure - take input data, do some magic and return output. Easy peasy.

I was really suprised that I didn't find test project already existing in solution. However it's no problem, is it? I quickly created it and started to write my test. First bussiness of order - prepare input.

    SimpleDataStructure inputDataStructure = new SimpleDataStructure()
    {
        FirstNecessaryProperty = "Value",
        SecondNecessaryProperty = 15
    }

So far so good! Now make the test fail. Test -> Run selected test. It did fail quite spectacularly. Exception thrown - supposedly I don't have permission to modify `FirstNecessaryProperty`! At this point I was like

![What?! gif](https://media.giphy.com/media/glmRyiSI3v5E4/giphy.gif)

How in the hell I'm not authorized to change property of my object?! I didin't have much experience with this code base, so I posted a question on our team's Slack channel asking if anyone had this type of error before and knows how to deal with it. Their response (or actually the lack of it) was CLEAR signal, that I'm getting myself into something very, very bad. I decided to ignore universe and it's _stupid_ signals and proceed with my test-driven development!

It turned out, that my `SimpleDataStructure` is not as simple as I thought. It was really `VeryBigAutoGeneratedEntityWithTonsOfAttributes`. Somewhere between 3000 and 3500 line of code I found that there is permission checking using some sort of permission provider. After hours of working and my colleague's help I was able to find ou that I can configure permssion provider in App.config using something like `GiveMeAllPermissionsProvider`. Great! I'm good to go!

In the middle of implementing my solution I decided, that I don't like this approach anymore, I can make this code more generic and expand my solution not only to this new functionality, but also to exsiting ones. With a little bit of luck, my code would work even with future funcionalitites as well! Thanks to my unit test I was able to constantly check if I didin't break anything comparing to previous version of my procedure. Luckly everything went smooth so I decided to introduce this new way to existing function.

Problem of not having any tests whatsoever hit me again. How should I know if everything works as before if I can't run any fast veryfication? Testing it through application interface was

1. stupid 
2. time expensive, because I'm not even sure if my local environment works 
3. very stupid. 

I decided to do this the right way - build a scaffolding of unit tests around old solution and when I have tests in place, start refactoring.

Creating data structures was a little complicated, but finally I was able to understand how to build input object. Only few more assertions and... go!

I'm sure, dear reader, that at this point you realized that this is a story of how everything went bad and then went even worse. I guess it will be no surprise, that yet again my test blew up with exception. This time it was a little more enigmatic. 

>Specified method is not supported

Unfortunately this error can occur because of various reasons, so StackOverflow wasn't able to help. I tried asking my team again, but with the same result as before. I really liked the idea of succeeding in my TDD, so I decided to do what developers do best - debug! Working day later I discovered soooo many dependecies inside those simple-looking objects like `SimpleDataStructure`. To initialize all of them I had to add custom configuration sections, new NuGet packages, references to other projects and some fairy dust. Finally I was able to run test like

    AnotherSimpleDataStructure input = new AnotherSimpleDataStructure()
    {
        FirstProperty = "SomeValue",
    }

    FunctionResult result = myFunction.DoWork(input);

    Assert.AreEqual("SomeValue", result.PropertyFirst);

I was never more happy about this kind of test turining green. From this point I was able to almost finish my refactoring!

And this brings us to today. For some bizzare reason my tests were working whole morning, then aroud last pulling code from repository they decided to throw exceptions once again. I was not able to discover why so far. This blog post is both scream for help and psychotherapy for me...

I'm like

![I'm done](https://media.giphy.com/media/QcgkcpzqEfB1C/giphy.gif)

Having this (true) story in mind. Do you really think you can "skip dependency injection for now"? I don't think so. After experiences like this you start to truly realise why whole concept of dependency injection is such a good idea. Why it is so important. And also why intruducing unnecessary dependecy into simple data structure can lead to developing disaster! 

We should really care more about how our classes, project and modules depend of each other. Writing `ServiceLocator.Instance.Resolve<>()` can look like good idea at first. I don't have to change my object's constructor and touch any code that uses it. I'll just sneak into this method and magically create instance of other object. Well... **it does need hundred lines of config**, but I know about this, right? I will NOT forget about it in like two months! What if my code will be used 3 years later after I changed job twice? _Ooups..._

Be explicit. If your object have dependecy I - user of this object's API - want to know about this! I want to be able to provide dumb implementation for unit testing. How should I mock object being resolved somewhere inside by some kind of resolver? Why should I configure whole dependecy injection framework to run simple tests?

**Be explicit.**

I'm sure that you've had experiences like this - let me know what are your remarks on this subject in comments below!