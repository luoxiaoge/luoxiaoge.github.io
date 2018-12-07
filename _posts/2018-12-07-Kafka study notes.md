## Kafka study notes

> I've been asked to evaluate RabbitMQ instead of Kafka but found it hard to find a reason that it's doing something better than Kafka. Does anyone knows if is really better in throughput, durability, latency, or ease-of-use?

### difference between Kafka and RabbitMq

RabbitMQ is a solid, general purpose **message broker** that supports several protocols such as AMQP, MQTT, STOMP etc. It can handle high-throughput and common use cases for it is to handle background jobs or as message broker between microservices. Kafka is a message bus optimized for **high-ingress data streams** and replay

Kafka can be seen as a **durable message broker** where applications can process and re-process streamed data on disk. Kafka has a very simple routing approach. RabbitMQ has better option if you need to route your messages in complex ways to your consumers. Use Kafka if you need to supporting batch consumers that could be offline, or consumers that want messages at low latency. 

RabbitMQ will keep all states about consumed/acknowledged/unacknowledged messages while Kafka doesn't, it assumes the consumer keep tracks of what's been consumed and not. RabbitMQ's queues are fastest when they're empty, while Kafka retain large amounts of data with very little overhead - Kafka is designed for holding and distributing large volumes of messages. (If you plan to have very long queues in RabbitMQ you could have a look at [lazy queues](https://www.rabbitmq.com/lazy-queues.html).)

Kafka is built from the ground up with horizontal scaling (scale by adding more machines) in mind, while RabbitMQ is mostly designed for vertical scaling (scale by adding more power ).

RabbitMQ has a user-friendly interface that let you monitor and handle your RabbitMQ server from a web browser. Among other things queues, connections, channels, exchanges, users and user permissions can be handled - created, deleted and listed in the browser and you can monitor message rates and send/receive messages manually. Kafka manager is yet not as developed as RabbitMQ Management interface. I would say that it's easier/gets faster to get a good understanding about RabbitMQ

[Is there any reason to use RabbitMQ over Kafka?](https://stackoverflow.com/questions/42151544/is-there-any-reason-to-use-rabbitmq-over-kafka)

[Comparison: Apache Kafka VS RabbitMQ](https://www.cloudkarafka.com/blog/2016-12-05-apachekafka-vs-rabbitmq.html)