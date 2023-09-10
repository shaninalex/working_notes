---
title: "Rabbitmq Concepts"
date: 2023-09-10T10:15:53+03:00
draft: true
tags: ["RabbitMQ"]
---


Before we dive deep into RabbitMQ concepts, let's get this app in our computer.
The best way, in my opinion, is to use docker. I do not like to write long
commands in the terminal so we will use **docker compose** file. Create folder
project folder and create this file:

```
# ./docker-compose.yml
version: "3.7"

services:
  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: rabbitmq_learn
    ports:
      - 5672:5672
      - 15672:15672

```

Notice that I used `management-alpine` version that include web management console
in it, available in port `15672`. And port `5672` is the port where we can connect
to the system via [AMQP](https://www.rabbitmq.com/protocol.html) (Advanced Message 
Queuing Protocol). 
Buy the way - RabbitMQ supports other protocols like Streaming Text Oriented 
Messaging Protocol (STOPM), Message, Queue Telemtry Transport (MQTT) and others

Start the app by running:

```bash
$ docker compose up -d
```

Let this app up and running in the background while you can continue reading.

# AMQP Queues

Queues are a collection of entities maintained in a sequence. Sequence ( queue )
can by modified by adding entities to the beginnign or to the end, removing from
the beginning or from the end of the queue.

### Queues-metrics

**Queue size** - number of messages in the queue,

**Queue age** - age of the oldest message in the queue.

If you have 1 million messages and the oldest one is 5 seconds - it’s okay, and 
your system handle queue easely, bacause it means that your system handle 200 000
messages per second. But if your queue have 10 messages and the oldest one is 5 
minutes - do some thing cuz’ it bearely handle any messages in the queue.

### Use cases

With queue architecture you can create modular system written in different 
laguages and technologies, you can estimate system performance and increase fault 
tolerance level. Support horizontal scaling, autoscaling. Increase system inbound 
traffic. Buffer system’s outboutnd bytes.

## AMQP Message

RabbitMQ contains this parts:

- **header** - metadata, key/value pairs - defined by AMQP specificatoin
- **properties** - metadata, key/value pairs - Application specific information holder
- **body** - payload is sequance of bytes ( byte[] )

## AMQP Connections and Channels

![](/static/images/rabbitmq-concepts/channels_connections.png)

Inside connections you can open multiple channels. Connection is an TCP/IP 
connection between the client and the broker. Channel is an virtual connection
inside the phisical TCP/IP connection. Why do we need multiple channels inside
connections instead just use multiple connections? Because establishing TCP/IP
connection can take a while and it can make communication slow. But using channels
inside already established connection will be much faster. Read more in [official
documentation](https://www.rabbitmq.com/connections.html).

## Queuing model

![](/static/images/rabbitmq-concepts/queuing-model.png)
 
Producer produce the message, message handlend by RabbitMQ broker and than 
consumed by consumer. Once message is consumed it is no longer available in 
RabbitMQ ( basicaly it do not store consumed messages since it just a broker )


## RabbitMQ Web Admin

Okay. Our app is definitly runnning and you can check this by enter in web admin
by url: `http://localhost:15672`. Default credentials
are `guest:guest`. And you will see this page:

![](/static/images/rabbitmq-concepts/rabbitmq-admin.png)

> About users and authorizations you can read [official documentation](https://www.rabbitmq.com/access-control.html#loopback-users)

I will not writing details about web interface because it's not the topic of this
article. But this is the place where you can practice.


# Core concepts

- **Producer** - emits messages to exchange.
- **Consumer** - receives messages from the queue.
- **Queue** - Keeps/Stores messages *( In RabbitMQ **producer** never published 
directly in the **queue**. It Has to use **exchange** )*.
- **Exchange** - is bound with **queue** by `binding key`. It dicides in which 
**queue** message should be placed. It compare `routing key` with `binding key`.

This is communication chart:

![](/static/images/rabbitmq-concepts/core-concepts.png)

Producer is sending message with `routing-key` "foo". Routing key is everything 
string value. You can think about it like about categories or tags in blogs. 
Exchange receive message and compares it with existing bindings (`binding-keys`).
If there is any queue with same binding key, than message is persisted in bound 
queue. Than RabbitMQ deliver message to the consumer. If there are no binding 
keys with given routing-key - publisher recives an error.

There are many exchange types:
- **nameless** - default
- **fanout**
- **direct**
- **topic**
- **headers**

> In `Exchanges` tab in web admin panel you can find predefined Rabbitmq exchanges. 
You can't modify them.

Nameless exchange ( default ) - compares routing key with queue name. Allows 
sending messages directly to the queue ( technicaly it sends messages throug the
exchange into the queue, but it just feel like it does). 

### fanout

It routes all received messages to all queues that are bound to it. Ignores `routing
key`. *It's like a fan that spread messages all around to the all queues*.

### direct

It send message to the queue that match `routing-key`. **Nameless** exchage is the 
**direct** exchange

### topic

It routes a received message to queues where binding key defined by a pattern. For
example binding key `*.log.error` routing key `application1.log.error`. If any
binding key match routing key - producer will receive the error.

### headers

It's simular the topic exchange but it compares against `any` or `all` message
headers ( header x-match determinees the behavior ).


# Queue concepts

located on single node where it was declared and referenced by unique name. 1 
queue = 1 Erlang process. Queue is an ordered collection of messages. They are 
handled by FIFO priciple (except prioritized queues). Internal queues are 
prefixed by `amq`.

### Main Queue Properties

- `QUEUE_NAME` - human readable name 
- `durable: bool` - survived brocker restart
- `exclusive: bool` - used for only one connection and will autodelete when 
connection closes
- `autoDelete: bool` - autodelete queue when all consumers disconected
- `classic or quorum` - increases availability
- `priority` - queues are FIFO by default, but you can change it by using this 
property
- `Expiration time (TTL)` - remove not consumed messages after some time. 

> All properties listed in [official documentation](https://www.rabbitmq.com/queues.html#properties)

> Thread about [naming convention](https://groups.google.com/g/rabbitmq-discuss/c/Jp49IRe693o)
as best practices to not mess around with names of exchanges, routing keys, queues etc...

---

# Used information

- Basicaly this post is my conspect of Udemy course. If you want to know more 
about RabbitMQ - buy [this greate cource](https://www.udemy.com/course/rabbitmq-in-practice/).
- [Official documentation](https://www.rabbitmq.com/)