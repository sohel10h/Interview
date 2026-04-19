# Kafka Brief for C# and .NET Core

## What Kafka Is

Apache Kafka is a distributed event streaming platform used for high-throughput, fault-tolerant messaging, event-driven systems, and real-time pipelines.

Simple interview line:

"Kafka is a distributed append-only log system where producers write events to topics and consumers read them independently."

In .NET interviews, Kafka usually comes up in:

- microservices communication
- event-driven architecture
- asynchronous processing
- audit/event logs
- real-time analytics pipelines

## Core Building Blocks

## Broker

A broker is a Kafka server.

A Kafka cluster has one or more brokers. Brokers store topic partitions, serve reads and writes, and replicate data to other brokers.

Interview line:

"Broker means one Kafka server node in the cluster."

## Topic

A topic is a logical stream of events, like:

- `orders`
- `payments`
- `user-events`

Producers write to topics. Consumers read from topics.

## Partition

A topic is split into partitions.

Each partition is an ordered append-only log. Partitions are the unit of:

- parallelism
- ordering
- storage distribution

Important:

- Kafka guarantees order only inside a partition
- Kafka does not guarantee global order across all partitions of a topic

## Offset

Each message inside a partition has an offset.

Offset is the sequential position of the record within that partition.

Example:

- Partition 0:
  - offset 0 -> OrderCreated
  - offset 1 -> OrderPaid
  - offset 2 -> OrderShipped

Interview line:

"Offset is like the row number of a record inside one partition."

## Key

The key decides which partition a message goes to, unless you explicitly set the partition.

Messages with the same key usually go to the same partition. That is why the key is important for:

- ordering
- grouping related events
- avoiding out-of-order updates for the same entity

Example:

- key = `customer-101`
- all events for `customer-101` go to the same partition
- consumer sees those events in order inside that partition

## Broker, Partition, Offset, Key Example

Assume topic `orders` has 3 partitions:

- Partition 0 on Broker 1
- Partition 1 on Broker 2
- Partition 2 on Broker 3

Producer sends:

1. key=`order-1001`, value=`OrderCreated`
2. key=`order-1001`, value=`OrderPaid`
3. key=`order-2001`, value=`OrderCreated`

Possible result:

- `order-1001` -> Partition 1, offset 15
- `order-1001` -> Partition 1, offset 16
- `order-2001` -> Partition 2, offset 42

What this means:

- same key `order-1001` goes to the same partition
- its events stay ordered in that partition
- each record gets its own offset
- different keys may go to different partitions for parallelism

## Producer, Consumer, Consumer Group

## Producer

A producer publishes events to Kafka topics.

Producer decides partitioning using:

- explicit partition
- message key
- default partitioning strategy

## Consumer

A consumer reads records from Kafka.

Consumer controls where it reads by offset.

## Consumer Group

Consumers in the same group split partitions among themselves.

Important rule:

- in one consumer group, one partition is consumed by at most one consumer at a time

Example:

- topic has 4 partitions
- consumer group has 2 consumers
- each consumer may get 2 partitions

If the group has 6 consumers and topic has 4 partitions:

- 4 consumers work
- 2 consumers stay idle

## Replication, Leader, Follower, ISR

Kafka uses replication for fault tolerance.

Each partition has:

- one leader
- zero or more follower replicas

All reads and writes go to the leader.

Followers replicate from the leader.

ISR means In-Sync Replicas, which are replicas caught up enough to be considered safe.

Very important interview setting:

- replication factor = 3
- producer `acks=all`
- `min.insync.replicas=2`

Why this matters:

- you get stronger durability
- producer fails fast if enough replicas are not available

## Retention and Log Compaction

Kafka is not just a queue. It is a durable log.

Messages are retained based on:

- time
- size
- compaction policy

Log compaction keeps the latest value for a given key, which is useful for:

- user profile state
- latest account balance
- current config/state topics

## Delivery Semantics

Kafka discussions often include:

- at-most-once
- at-least-once
- exactly-once

Short version:

- at-most-once: no duplicates, but may lose messages
- at-least-once: no loss preferred, but duplicates can happen
- exactly-once: strongest guarantee, but more complexity

In most business systems, at-least-once plus idempotent consumers is common.

## Problem Solving Example 1: Produce Keyed Messages in .NET

Common .NET package:

```bash
dotnet add package Confluent.Kafka
```

Producer example:

```csharp
using Confluent.Kafka;

var config = new ProducerConfig
{
    BootstrapServers = "localhost:9092",
    Acks = Acks.All,
    EnableIdempotence = true
};

using var producer = new ProducerBuilder<string, string>(config).Build();

var message = new Message<string, string>
{
    Key = "order-1001",
    Value = "OrderCreated"
};

var result = await producer.ProduceAsync("orders", message);

Console.WriteLine(
    $"Topic={result.Topic}, Partition={result.Partition}, Offset={result.Offset}");
```

What this shows:

- message key
- durable producer setup
- broker returns partition and offset

## Problem Solving Example 2: Consume Messages in .NET

```csharp
using Confluent.Kafka;

var config = new ConsumerConfig
{
    BootstrapServers = "localhost:9092",
    GroupId = "order-service",
    AutoOffsetReset = AutoOffsetReset.Earliest,
    EnableAutoCommit = false
};

using var consumer = new ConsumerBuilder<string, string>(config).Build();
consumer.Subscribe("orders");

while (true)
{
    var result = consumer.Consume();

    Console.WriteLine(
        $"Key={result.Message.Key}, Value={result.Message.Value}, Partition={result.Partition}, Offset={result.Offset}");

    consumer.Commit(result);
}
```

Interview point:

- consumer reads from a partition at a specific offset
- manual commit gives more control
- commit should happen after successful processing

## How to Scale Kafka

This is one of the most common practical interview questions.

If Kafka gets too many requests, do not just say "add more servers." Explain where the bottleneck is.

## 1. Scale Brokers

Add more brokers when:

- storage is full
- network is saturated
- broker CPU is high
- partition leaders are overloaded

This distributes partitions and traffic across more machines.

## 2. Increase Partition Count

Partitions are the unit of parallelism.

More partitions allow:

- more producer parallel writes
- more consumer parallel reads
- more consumers in a group

But too many partitions also increase:

- metadata overhead
- open files
- rebalance cost
- operational complexity

Important caution:

Increasing partitions later can change key-to-partition mapping and affect ordering assumptions for keys.

## 3. Scale Consumers

If consumers cannot keep up:

- increase consumer instances in the same group
- make sure the topic has enough partitions

Rule:

- max active consumers in one group is effectively bounded by partition count

## 4. Tune Producers

High-throughput producer tuning usually involves:

- `linger.ms`
- `batch.size`
- compression
- `acks`
- idempotence

Bigger batches and compression can improve throughput.

## 5. Spread the Key Better

Bad keys can create a hot partition.

Example of a bad design:

- all messages use key=`system`

That pushes most traffic into one partition.

Better:

- use entity-based keys like `orderId`, `customerId`, or another balanced key

## 6. Use Strong Topic Design

Split very different workloads into separate topics when needed.

Do not put unrelated high-volume and low-volume workloads into one topic just because both are "events".

## 7. Tune the Cluster

Depending on the real bottleneck, you may need to review:

- disk throughput
- broker CPU
- network bandwidth
- replication settings
- message size
- partition leader distribution

## Example: Kafka Getting Too Many Requests

Good interview answer:

"First I would check whether the bottleneck is producer throughput, broker CPU/network/disk, or consumer lag. If producer or broker throughput is the issue, I would add brokers, rebalance partitions, and increase partitions if more parallelism is needed. If the bottleneck is a hot partition, I would review the message key strategy. If consumers are behind, I would scale the consumer group and optimize processing time. I would also tune batching, compression, and idempotent producer settings."

## Common Kafka Problems and How to Overcome Them

## 1. Consumer Lag

Problem:

- producers write faster than consumers can process

Why it happens:

- slow business logic
- too few consumers
- too few partitions
- downstream DB/API is slow

How to overcome:

- scale consumers
- increase partitions if needed
- optimize consumer processing
- batch downstream writes
- monitor lag continuously

## 2. Hot Partition

Problem:

- one partition gets most traffic

Why it happens:

- poor key design

How to overcome:

- choose a better key
- shard the key if business ordering allows
- review partition distribution

## 3. Duplicate Processing

Problem:

- consumer may process a record twice

Why it happens:

- crash after processing but before offset commit
- retries

How to overcome:

- make consumers idempotent
- store processed message ids if needed
- commit offsets after successful processing
- use idempotent producer where appropriate

## 4. Message Loss Risk

Problem:

- low durability settings

How to overcome:

- use replication
- use `acks=all`
- set `min.insync.replicas`
- avoid unsafe producer settings for critical topics

## 5. Rebalance Pauses

Problem:

- consumers pause during rebalancing

How to overcome:

- keep consumer group stable
- avoid frequent consumer restarts
- tune polling/processing behavior
- keep processing time reasonable

## 6. Large Messages

Problem:

- large payloads hurt throughput and can hit size limits

How to overcome:

- keep messages small
- send object storage pointer instead of huge payload
- tune broker/producer/consumer size configs only if truly required

## 7. Wrong Retention Settings

Problem:

- data disappears earlier than expected or storage grows too much

How to overcome:

- define retention by business need
- monitor storage growth
- use compaction only when the topic semantics fit

## 8. Broker Failure / Replica Lag

Problem:

- followers fall behind, ISR shrinks, leaders become stressed

How to overcome:

- monitor ISR and replica lag
- ensure enough disk/network capacity
- tune replication and infrastructure
- keep replication factor appropriate

## Other Kafka Concepts You Should Know

## ZooKeeper vs KRaft

Older Kafka clusters used ZooKeeper for metadata management.

Modern Kafka uses KRaft mode, where Kafka handles metadata itself without ZooKeeper.

Interview note:

- mention KRaft as the modern direction
- but many companies still run older setups

## Kafka Connect

Kafka Connect is used to move data between Kafka and external systems like:

- databases
- S3
- Elasticsearch

Use it when you want integration without writing full custom pipeline code.

## Kafka Streams

Kafka Streams is a stream-processing library for:

- filtering
- joining
- windowing
- aggregation

Good interview point:

- Kafka itself is messaging/storage
- Kafka Streams is stream processing on top of Kafka

## Schema Registry and Avro/Protobuf/JSON Schema

In real systems, message schemas matter.

Common production approach:

- schema-managed events
- versioned contracts
- compatibility rules

This avoids breaking consumers when message formats evolve.

## What to Monitor

Very important operational metrics:

- consumer lag
- broker CPU
- disk usage
- network throughput
- request latency
- replica lag
- ISR shrink events
- produce/fetch request rates

## Quick Interview Summary

If asked for a short answer:

"Kafka is a distributed event streaming platform built around topics, partitions, brokers, and offsets. Producers write to topic partitions, consumers read from them using offsets, and consumer groups provide horizontal scaling. Order is guaranteed only within a partition, which is why message keys matter. To scale Kafka, I look at partitions, brokers, consumer lag, key distribution, batching, and replication settings rather than just adding hardware blindly."

## References

- [Apache Kafka - Introduction](https://kafka.apache.org/08/getting-started/introduction/)
- [Confluent - Introduction to Apache Kafka](https://docs.confluent.io/kafka/introduction.html)
- [Confluent - Kafka Producer Design](https://docs.confluent.io/kafka/design/producer-design.html)
- [Confluent - Kafka Consumer Design](https://docs.confluent.io/kafka/design/consumer-design.html)
- [Confluent - Kafka Replication and Committed Messages](https://docs.confluent.io/kafka/design/replication.html)
- [Apache Kafka - Broker Config Reference (`min.insync.replicas`)](https://kafka.apache.org/39/generated/kafka_config.html)
- [Confluent - Choose and Change the Partition Count in Kafka](https://docs.confluent.io/kafka/operations-tools/partition-determination.html)
- [Confluent - Kafka .NET Client](https://docs.confluent.io/kafka-clients/dotnet/1.5/overview.html)
