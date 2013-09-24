---
layout: post
title: "RabbitMQ a Quick and Dirty Introduction"
date: 2013-09-24 06:44
comments: true
categories: rabbitmq erlang amqp
---

RabbitMQ has become one of the most popular solutions for application messaging. Owned and maintained by VMWare, its
proven track record, stability, and speed has made it the poster child of what a powerful message bus should be. This
introduction should be viewed as a boiled down crash course in RabbitMQ.

The Quick
---------

### AMQP ###

RabbitMQ is a broker for the AMQP (Advanced Message Queuing Protocol) v0.9.1. On the face of it, the AMQP protocol is
pretty simple. If you are used to working with email, the concepts in AMQP will be nothing new. It can be viewed as a
series of mail boxes, mail exchanges, and addresses working together to get the job done. My purpose is not to
go into great detail about AMQP itself or its history, I will leave that to your own research or perhaps a later post.
Rather this post is intended to give the developer enough information that they can navigate their way around or at
least have a fairly good understanding of how it all works together to get the job done.

### RabbitMQ ###

RabbitMQ is the goto broker for AMQP messaging. With the financial backbone of EMC and VMWare under the guise of
GoPivotal, this will probably be the case for many years to come. It is written in Erlang, a functional language
known for its ability for concurrent processing, and backed by Mnesia, Erlang's powerful persistence database.

Currently RabbitMQ is used by some of the biggest names in the tech world including:

* VMWare/EMC (obviously)
* AT&T Interactive
* NASA
* Digg
* BBC
* Nokia
* Heroku


The Dirty
---------

### Channels ###

When working with RabbitMQ (or AMQP for that matter) one of the first things you will run into is the Channel. A
channel is essentially a virtual connection allowing separate threads to maintain their own point of communication while
still using a single TCP connection.

### Exchanges ###

The Exchange can be viewed as the backbone of the AMQP protocol. It acts much as an individual mail server determining
which mailboxes (queues) to deliver messages to. These come in several different flavors covering almost all of your
delivery needs:

* **Direct** - The most commonly used exchange type. Direct exchanges are designed for fast and, you guessed it, direct
  delivery to queues using a *key*, or effectively the Queue's address on that exchange.
* **Topic** - The Topic Exchange is similar to the direct exchange with the exception that queues bound to this exchange
  can listen to variable keys. For example, in a direct exchange if you wanted a queue to listen every message coming
  into *'foo.bar.\*'( you would have to bind that queue for every possible key. A topic exchange instead allows wildcard
  subscriptions, so you can literally bind to 'foo.bar.*' and it will consume everything from *'foo.bar.baz'* to
  *'foo.bar.why.does.everyone.use.foo.bar'*.
* **Fanout** - In the Fanout Exchange, the message keys are completely ignored. Instead this exchange delivers to every
  queue bound to it.
* **Headers** - This is possibly the least used, but most powerful type of Exchange. Instead of using the message key
  it actually looks at the message headers to determine where it is routed to.

### Queues ###

The proverbial mailbox. Queues are pretty self descriptive. The only things fo real note are the options. When queues
are declared they can be passed a number of attributes that determine how the queue behaves:

* **Durable** - Messages in a durable queue will survive restart.
* **Exclusive** - The queue is dedicated to a single connection. When that connection is lost, the queue will be
  removed.
* **AutoDelete** - Similar to Exclusive queues, AutoDelete will cause the queue to be deleted when the last subscriber
  disappears. Unlike Exclusive queues, this allows multiple subscribers.

### Bindings ###

Bindings are the glue that holds queue to exchange. When a queue is declared it can be bound to multiple exchanges,
each binding defining its own rules of how messages are to be delivered to the queue. For example, you may bind a
single queue to a Topic exchange with key *'foo.bar.\*'* and to a Fanout Exchange. The queue will receive messages
directed to it on the Topic Exchange and all messages from the Fanout Exchange both.

### Consumers ###

While not technically part of the broker, consumers still bare mentioning. The most useful point here is that consumers
can subscribe to messages transactionally. This allows the subscriber to either acknowledge or reject a message based
on whether or not it was able to process it. Combine this with RabbitMQ's *Dead Lettering* capabilities and you have
the makings of a extraordinarily fault tolerant messaging system.

Differences Between RabbitMQ and other AMQP Brokers
---------------------------------------------------

While most of the above details the more the AMQP protocol in general, I would be remiss if I didn't point out some
slight differences RabbitMQ has with the AMQP 0.9.1 protocol proper.

RabbitMQ comes with a number of protocol extensions giving the developer a bit more power than they would have
otherwise:

* **Confirms** The ability for the **publisher** to confirm message delivery.
* **Exchange to Exchange Bindings** RabbitMQ provides the ability to bind exchanges to each other.
* **Message TTL** - RabbitMQ implements a TTL header that defines how long a message is allowed to live in a queue.
  When this time expires, it is removed from the queue.
* **Dead Lettering** - This is, in my opinion one of the best extensions RabbitMQ provides. In the case that a message
  is considered *dead*, you can define an exchange to transfer that message to for handling. Using this in combination
  with Message TTL, one can implement a timed retry of messages or even an automatic consumer delay.


For those of you who have only passively heard about RabbitMQ, hopefully you find this post useful. My goal is to later
go into more details about implementing and using Rabbit's various features, so stay tuned.
