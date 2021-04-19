# RabbitMQ TOPIC Exchange Node.js Lab

## Prepare project

### Install dependencies

```bash
yarn install
```

### Run RabbitMQ container (docker is required)

Rabbit on 5672 port and management on 8081.

Management user and password are "**guest**" without quotes.

```bash
docker run -d -v ${PWD}/rabbit-db:/var/lib/rabbitmq --hostname yt-rabbit -p 5672:5672 -p 8081:15672 --name yt-rabbit rabbitmq:3-management
```

## Patterns
### Wildcards
* `*` (star) can substitute for exactly one word.
* `#` (hash) can substitute for zero or more words.

### What is a word?
The word is the string between dots: `company.v1.images.crop`

* `word.word.word.word` Each "word" string is a word.

To match `company.v1.images` routing key you can use, for example:
#### Exact match (like a `direct` exchange)
* `company.v1.images`

#### With `*`
* `*.v1.images`
* `*.*.images`
* `*.*.*`
* `company.*.images`
* `company.*.*`
* `company.v1.*`

#### With `#`
* `company.#`
* `#.v1.#`
* `#.images`
* `#` Receives all messages regardless of the routing key.
  (like a `fanout` exchange)

#### You can use both
* `*.v1.#`

## Same Exchange and Match Pattern, different queue

All subscribers receive all messages.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=company.v1.pdf.* QUEUE=first EXCHANGE=my-topic node subscriber/topic-exchange.js

PATTERN=company.v1.# QUEUE=second EXCHANGE=my-topic node subscriber/topic-exchange.js
```

### Publishers

```bash
ROUTING_KEY=company.v1.pdf.generate EXCHANGE=my-topic node publisher/topic-exchange.js
```

## Same Exchange and Match Pattern and queue

All subscribers receive messages using round-robin.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=company.v1.pdf.* QUEUE=first EXCHANGE=my-topic node subscriber/topic-exchange.js

PATTERN=company.v1.# QUEUE=first EXCHANGE=my-topic node subscriber/topic-exchange.js
```

### Publishers

```bash
ROUTING_KEY=company.v1.pdf.generate EXCHANGE=my-topic node publisher/topic-exchange.js
```

## Different Exchange, same Match Pattern and queue

All subscribers receive messages using round-robin.

**This happens because queue is bound to both exchanges.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=company.v1.pdf.* QUEUE=second EXCHANGE=my-topic-2 node subscriber/topic-exchange.js

PATTERN=company.v1.# QUEUE=second EXCHANGE=my-topic node subscriber/topic-exchange.js
```

### Publishers

```bash
ROUTING_KEY=company.v1.pdf.generate EXCHANGE=my-topic node publisher/topic-exchange.js
```

## Keep messages after RabbitMQ restart
### Durability and persistence
It is necessary to set exchange and queue as **durable** and messages
as **persistent** when they are published.
#### Add exchange as "durable"

```js
channel.assertExchange(exchangeName, exchangeType, {
    durable: true
})
```

#### Add queue as "durable"

```js
await channel.assertQueue(queue, {
    durable: true
})
```

#### Add publish "persistent" option as true (default is false)

```js
const sent = channel.publish(
    exchangeName,
    routingKey,
    Buffer.from(JSON.stringify(message)),
    {
        persistent: true
    }
)
```
