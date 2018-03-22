---
Title:  "Chain of responsibility vs. message queues"
Published:   2018-02-24
Tags: [design patterns, software craftsmanship]
Comments: true
---
Often when I read about using design patterns I see adapter, factory method or singleton. Less often someone would use decorator or strategy. I have the feeling that some design patterns are super popular and others are almost not used at all! One of those forgiven patterns is chain of responsibility, which I want to talk about in this post. When I think about this pattern message queues also come in mind, so I will put chain of responsibility in this context.

Chain of responsibility is a behavioral design pattern. It's used to decouple message sender and receiver. Additionally one request can be handled by one of multiple handlers or by many handlers one after another. This concept is really similar to queues. In next paragraphs I will provide some sample implementation of this pattern and also talk about when it can be a good idea and when we should stay with our beloved RabbitMQ or MSMQ or whatever MQ system there is :-).

Structure of chain of responsibility is based on linked list. Base idea is this: we have 