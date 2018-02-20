---
Title:  "Simple mediator pattern implementation in C#"
Published:   2018-02-15
Tags: [c#, design patterns]
---
Recently I watched [fantastic talk by Jimmy Bogard](https://www.youtube.com/watch?v=wTd-VcJCs_M) about how to design your application, so your architecture is as SOLID as your code is. In his talk he says about CQRS and how this pattern allowed him to design his application better.

I have to say - this talk really inspired me. In projects I worked on it was pretty common, that there would be a <code>Repository</code> class, which had (too) many methods. In most cases one method was used in one single place. I really like the idea of bringing CQRS as cure for that.

<!--more-->

Basic idea is this - let's create <code>OperationRequest</code> class for every request in our appliation and <code>OperationResponse</code> for every response. Request represents data which will be used by <code>OperationHandler</code> to perform some task. Response represents result of requested operation. Let's write the code!

This kind of decoupling of objects, that work together, but should not know about eachother can be achieved using [mediator design pattern][mediator]. This pattern assumes, that there is some mediator between objects, so they are referencing this mediator object and not eachother. Thanks to that, objects are coupled only to mediator so their interactions can vary independently. 

To two classes representing request and response we have to add three more - actual mediator, request handler and handler provider, which will be responsible for choosing handler based on request.

Mediator has only one job - take request, dispatch it to appropriate request handler and return response. Interface for mediator looks like this
<pre>
public interface IMediator
{
    TResponse Dispatch<TRequest, TResponse>(TRequest query);
}
</pre>

Handler provider has also simple task - based on provided request get request handler and return it.

<pre>
public interface IHandlerProvider
{
    IRequestHandler<TRequest, TResponse> ProvideHandler<TRequest, TResponse>();
}
</pre>

If we have provider we should declare how those provided handlers will look like. Provider is responsible for handlig query and returning response.

<pre>
public interface IRequestHandler<TRequest, TResponse>
{
    TResponse Handle(TRequest query);
}
</pre>

We just declare an interface. We want client to decide _how_ he wants to do that. [In my sample app on GitHub][github_app] I created <code>DumbHandlerProvider</code> which stores request -> handler map in simple <code>Dictionary</code>, but I also created <code>AutofacHandlerProvider</code> which uses Autofac to resolve all dependecies and to find request handler.

We need just one more interface. It will be implemented by all <code>Request</code> classes and will serve as common type for them, because <code>Request</code> classes are simple data structures, so this interface doesn't declare any methods.

<pre>
public interface IRequest<TResponse>
{
}
</pre>

With all those interfaces we can finally implement them! Let's start with <code>Mediator</code>

<pre>
public class Mediator : IMediator
{
    private IHandlerProvider _handlerProvider { get; set; }

    public Mediator(IHandlerProvider handlerProvider)
    {
        this._handlerProvider = handlerProvider;
    }
    
    public TResponse Dispatch<TRequest, TResponse>(TRequest request)
    {
        var handler = this._handlerProvider.ProvideHandler<TRequest, TResponse>();

        return handler.Handle(request);
    }
}
</pre>

Implementation is really simple - use provided IHandlerProvider to get handler and handle request with specific data.  This object is actually the only thing that we're implementing in, let's say, library part of this solution. The rest of declared interfaces will be implemented by client.

First we declare how our request and response look like. Usually handling request would require, sometimes very complicated, logic. Our sample is a simple HelloWorld application, but we will simulate domain logic. In our request we will pass language parameter. Depending on its value we will return hello world message.

<pre>
public class HelloWorldRequest : IRequest<HelloWorldResponse>
{
    public enum SupportedLanguage
    {
        PL = 0,
        EN = 1
    };

    public SupportedLanguage Language { get; set; }
}
</pre>

Response looks like this:

<pre>
public class HelloWorldResponse
{
    public string HelloWorldText { get; set; }
}
</pre>

As you can see both request and response are not pretty simple. They responsibility is basically only hold pieces of information required for performing a task or for receiving response. 

We have request and response model, so lets create our handler

<pre>
public class HelloWorldRequestHandler : IRequestHandler<HelloWorldRequest, HelloWorldResponse>
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
</pre>

In Handle method we check what language contains HelloWorldRequest.Language and based on this parmeter we're creating response object and returning it to mediator.

For this post I will use my DumbHandlerProvider, but in [mentioned before repository][github_app] you can find provider based on Autofac. Every handler provider has to implement IHandlerProvider interface. Dumb implementation can look like this:

<pre>
public class DumbHandlerProvider : IHandlerProvider
{
    private Dictionary<Type, Type> _handlers = new Dictionary<Type, Type>()
    {
        { typeof(HelloWorldRequest), typeof(HelloWorldRequestHandler) }
    };

    public IRequestHandler<TRequest, TResponse> ProvideHandler<TRequest, TResponse>()
    {
        Type requestType = typeof(TRequest);
        IRequestHandler<TRequest, TResponse> handler = null;

        if (!this._handlers.ContainsKey(requestType))
        {
            throw new ApplicationException($"Handler for query {requestType} is not registered");
                
        }
        Type handlerType = this._handlers[requestType];

        handler = (IRequestHandler<TRequest, TResponse>)Activator.CreateInstance(handlerType);

        return handler;
    }
}
</pre>

In private Dictionary we are holding request -> handler mapping. Based on passed request we get instance of handler and returning it. Using this mechanism is really simple. We create Mediator object passing in its constructor instance of DumbHandlerProvider (or instance of any other class implementing IHandlerProvider interface of course) and we dispatch request to get response.

<pre>
Mediator mediator = new Mediator(new DumbHandlerProvider());

HelloWorldRequest query = new HelloWorldRequest()
{
    Language = HelloWorldRequest.SupportedLanguage.EN,
};

HelloWorldResponse response = mediator.Dispatch<HelloWorldRequest, HelloWorldResponse>(query);

Console.WriteLine(response.HelloWorldText);
Console.ReadLine();
</pre>

As you can see, implementing [mediator design pattern][mediator] can be really simple. This is NOT a producton code - I wrote all this as a coding exercise :-).

[github_app]: https://github.com/trobinpl/mediator
[mediator]: https://sourcemaking.com/design_patterns/mediatorhttps://github.com/trobinpl/mediator