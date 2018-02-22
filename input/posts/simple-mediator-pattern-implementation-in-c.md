---
Title:  "Simple mediator pattern implementation in C#"
Published:   2018-02-15
Tags: [c#, design patterns]
Comments: true
---
Recently I watched [fantastic talk by Jimmy Bogard](https://www.youtube.com/watch?v=wTd-VcJCs_M) about how to design your application, so your architecture is as SOLID as your code is. In his talk he says about CQRS and how this pattern allowed him to design his application better.

I have to say - this talk really inspired me. In projects I worked on, both professionally and as a hobby, it was an obvious to have a <code>Repository</code> class, which had methods like

<pre><code class="cs">public class EntityRepository
{
    public EntityType GetById(int id);
    public EntityType GetByIdWithSomething(int id);
    // and so on...
}
</code></pre>

Size of this file grew along side with project itself. In the end there would be a lot o methods, that were used only in one place in our codebase. Now when I think about that it's clear, that this approach violates many good practices. Most of all - Single Responsibility Principle. I really like how you could improve this poor design with a little of CQRS and [mediator design pattern][mediator]. It's really cool, because using this idea is possible even in legacy project and it will take only a little time to adopt.

<!--more-->

Basic idea is this - let's create <code>OperationRequest</code> class for every request in our appliation and <code>OperationResponse</code> for every response. Request represents data which will be used by <code>OperationHandler</code> to perform some task. Response represents result of requested operation. There are serveral ways to achieve this, but I will present my simple implemenation of [mediator design pattern][mediator] as a one possibility. Basic idea beind this design pattern is to couple interacting objects loosely, so they don't reference each other directly by introducing additional object, that mediates between collaborators.


In my solution mediator has only one job - take request, dispatch it to appropriate request handler and return response. Interface for mediator looks like this
<pre><code class="cs">public interface IMediator
{
    TResponse Dispatch&lt;TRequest, TResponse&gt;(TRequest query);
}
</code></pre>

Handler provider has also simple task - based on provided request get request handler and return it.

<pre><code>public interface IHandlerProvider
{
    IRequestHandler&lt;TRequest, TResponse&gt; ProvideHandler&lt;TRequest, TResponse&gt;();
}
</code></pre>

It's an interface for client to implement, so he can decide _how_ he would like to provide those handlers. [In my sample app on GitHub][github_app] I created <code>DumbHandlerProvider</code> which stores request -> handler map in simple <code>Dictionary</code>, but I also created <code>AutofacHandlerProvider</code> which uses Autofac to resolve all dependecies and to find request handler.

Provider will provide us with specific request handler. It is responsible for handling request in way it should be handled and returning response.

<pre><code>public interface IRequestHandler&lt;TRequest, TResponse&gt;
{
    TResponse Handle(TRequest query);
}
</code></pre>

We need just one more interface. It will be implemented by all <code>Request</code> classes and will serve as common type for them, because <code>Request</code> classes are simple data structures, so this interface doesn't declare any methods. It could be an <code>abstract class</code>, but I just went with interfaces all the way down ;-)

<pre><code>public interface IRequest&lt;TResponse&gt;
{
}
</code></pre>

With all those interfaces we can finally implement them! Let's start with <code>Mediator</code>

<pre><code>public class Mediator : IMediator
{
    private IHandlerProvider _handlerProvider { get; set; }

    public Mediator(IHandlerProvider handlerProvider)
    {
        this._handlerProvider = handlerProvider;
    }
    
    public TResponse Dispatch&lt;TRequest, TResponse&gt;(TRequest request)
    {
        var handler = this._handlerProvider.ProvideHandler&lt;TRequest, TResponse&gt;();

        return handler.Handle(request);
    }
}
</code></pre>

Implementation is really simple - use provided <code>IHandlerProvider</code> to get handler and handle request with specific data.

In real world handling request requires performing some businness logic, but in my HelloWorld application I will assume that my logic is simply chosing language for "Hello world". I'm passing language in request and expect translated "Hello world" in response.

<pre><code>public class HelloWorldRequest : IRequest<HelloWorldResponse>
{
    public enum SupportedLanguage
    {
        PL = 0,
        EN = 1
    };

    public SupportedLanguage Language { get; set; }
}
</code></pre>

Response looks like this:

<pre><code>public class HelloWorldResponse
{
    public string HelloWorldText { get; set; }
}
</code></pre>

As you can see both request and response are not complicated at all. They are just simple data structures. We have request and response model, so lets create our handler.

<pre><code>public class HelloWorldRequestHandler : IRequestHandler&lt;HelloWorldRequest, HelloWorldResponse&gt;
{
    public HelloWorldResponse Handle(HelloWorldRequest helloWorldQuery)
    {
        HelloWorldResponse response = new HelloWorldResponse();
        switch (helloWorldQuery.Language)
        {
            case HelloWorldRequest.SupportedLanguage.PL:
                response.HelloWorldText = "Witaj świecie!";
                break;
            case HelloWorldRequest.SupportedLanguage.EN:
                response.HelloWorldText = "Hello world";
                break;
        }

        return response;
    }
}
</code></pre>

In Handle method we check what language was passed in <code>HelloWorldRequest</code> and based on this parmeter we're creating response object. This object is then returned to mediator.

For this post I will use my <code>DumbHandlerProvider</code> mentioned before. [In my repository I mentioned earlier][github_app] you can find provider based on Autofac. It should give you an idea on how you could implement <code>IHandlerProvider</code> so it suit your needs.

<pre><code>public class DumbHandlerProvider : IHandlerProvider
{
    private Dictionary&lt;Type, Type&gt; _handlers = new Dictionary&lt;Type, Type&gt;()
    {
        { typeof(HelloWorldRequest), typeof(HelloWorldRequestHandler) }
    };

    public IRequestHandler&lt;TRequest, TResponse>&gt; ProvideHandler&lt;TRequest, TResponse&gt;()
    {
        Type requestType = typeof(TRequest);
        IRequestHandler&lt;TRequest, TResponse&gt; handler = null;

        if (!this._handlers.ContainsKey(requestType))
        {
            throw new ApplicationException($"Handler for query {requestType} is not registered");
                
        }
        Type handlerType = this._handlers[requestType];

        handler = (IRequestHandler&lt;TRequest, TResponse&gt;)Activator.CreateInstance(handlerType);

        return handler;
    }
}
</code></pre>

Having provider implemented we can finally wire everything together.

<pre><code>Mediator mediator = new Mediator(new DumbHandlerProvider());

HelloWorldRequest query = new HelloWorldRequest()
{
    Language = HelloWorldRequest.SupportedLanguage.EN,
};

HelloWorldResponse response = mediator.Dispatch&lt;HelloWorldRequest, HelloWorldResponse&gt;(query);

Console.WriteLine(response.HelloWorldText);
Console.ReadLine();
</code></pre>

I must admit I really like this idea of splitting whole request handling process into pieces. It looks much cleaner in actual code, keep objects decoupled and when there is new feature, you just create couple new classes and you're set. Above code is obviously not production ready - it's just an excersise of mine. Actual implementation could be found in [Jimmy's MediatR library][mediatr].

How do you like this approach? Let me know in comments below!

[github_app]: https://github.com/trobinpl/mediator
[mediator]: https://sourcemaking.com/design_patterns/mediator
[mediatr]: https://github.com/jbogard/MediatR