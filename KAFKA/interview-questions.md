# Kafka Interview Questions and Answers

These are common Kafka interview questions for backend and .NET roles, grouped by difficulty and aligned with official Kafka and Confluent documentation.

## Easy

1. What is Kafka?
   Answer: Kafka is a distributed event streaming platform used to publish, store, and consume streams of records at high throughput.
2. What is a broker in Kafka?
   Answer: A broker is a Kafka server node that stores partitions and handles client read/write requests.
3. What is a topic?
   Answer: A topic is a logical category or stream of events, such as `orders` or `payments`.
4. What is a partition?
   Answer: A partition is an ordered append-only log inside a topic. It is the unit of storage and parallelism.
5. What is an offset?
   Answer: Offset is the position number of a record within a partition.
6. What is a producer?
   Answer: A producer is a client that writes records to Kafka topics.
7. What is a consumer?
   Answer: A consumer is a client that reads records from Kafka topics.
8. What is a consumer group?
   Answer: A consumer group is a set of consumers working together to split topic partitions among themselves.
9. Does Kafka guarantee message order?
   Answer: Kafka guarantees order only within a partition, not across all partitions of a topic.
10. What is the role of a message key?
    Answer: The key influences partition selection and helps keep related events in the same partition for ordering.

## Medium

1. How does Kafka decide which partition gets a message?
   Answer: It can use an explicitly chosen partition, a partitioner based on the message key, or a default partitioning strategy.
2. Why do messages with the same key usually go to the same partition?
   Answer: Because Kafka hashes the key to select the partition, which helps preserve ordering for that key.
3. What is replication factor?
   Answer: Replication factor is the number of copies of each partition stored across brokers.
4. What is the difference between leader and follower replicas?
   Answer: The leader handles reads and writes. Followers replicate data from the leader.
5. What is ISR in Kafka?
   Answer: ISR means In-Sync Replicas, the replicas that are caught up enough to be considered safe for committed data.
6. What does `acks=all` mean?
   Answer: It means the producer waits for acknowledgment from all in-sync replicas before considering the write successful.
7. Why is `min.insync.replicas` important?
   Answer: It defines how many in-sync replicas are required for a successful write when using `acks=all`, improving durability.
8. What is consumer lag?
   Answer: Consumer lag is how far behind a consumer group is compared to the latest available offsets.
9. What is the difference between auto offset commit and manual commit?
   Answer: Auto commit is automatic and simpler, while manual commit gives more control and is safer for precise processing flows.
10. Why can duplicates happen in Kafka systems?
    Answer: Retries, crashes, and commit timing can cause a record to be processed more than once.

## Hard

1. How would you explain Kafka scaling?
   Answer: Kafka scales by adding brokers, increasing partitions, scaling consumer instances, tuning producers, and fixing key distribution problems such as hot partitions.
2. If Kafka is getting too many requests, what would you do first?
   Answer: First identify the bottleneck: producer throughput, broker CPU/network/disk, consumer lag, or hot partition skew. Then scale the correct layer instead of guessing.
3. Why can too many partitions also be a problem?
   Answer: Too many partitions increase metadata overhead, rebalance time, file handles, and operational complexity.
4. How do you handle a hot partition?
   Answer: Improve key design, spread load more evenly, and only keep strict same-key ordering where the business actually needs it.
5. How do you reduce consumer lag?
   Answer: Scale consumers, increase partitions if needed, optimize processing, batch downstream writes, and monitor lag closely.
6. How do you reduce risk of message loss?
   Answer: Use replication, `acks=all`, suitable `min.insync.replicas`, and durable producer/cluster settings.
7. How do you design Kafka for high durability?
   Answer: Use replication factor greater than 1, leader/follower replication, `acks=all`, and strong ISR settings. For many systems, replication factor 3 is common.
8. What is the difference between at-most-once, at-least-once, and exactly-once?
   Answer: At-most-once may lose data but avoids duplicates, at-least-once avoids loss but may duplicate, and exactly-once aims to avoid both under supported scenarios.
9. When is Kafka a poor fit?
   Answer: Kafka is a poor fit when you need request/response RPC, low-volume simple queues with minimal ops overhead, or full global ordering with high parallelism.
10. What is the risk of using a bad key?
    Answer: A bad key can overload one partition, hurt throughput, increase lag, and reduce the benefits of distributed parallelism.

## Very Common Concept Questions

1. Explain broker, partition, offset, and key with one example.
   Answer: Suppose topic `orders` has 3 partitions spread across 3 brokers. If producer sends two messages with key `order-1001`, both usually go to the same partition, for example Partition 1. Inside that partition they may get offsets 15 and 16. Broker means the Kafka server storing partitions. Partition is the ordered log. Offset is the record position inside that partition. Key is the value used to keep related events together.
2. Why is key important in Kafka?
   Answer: Because the key controls partitioning behavior and therefore affects both ordering and load distribution.
3. Why does Kafka only guarantee order within a partition?
   Answer: Because each partition is its own ordered log and different partitions are processed independently for scalability.
4. What happens if the topic has fewer partitions than consumers in a group?
   Answer: Some consumers remain idle because only one consumer in the group can actively consume a given partition at a time.
5. What happens if the producer sends no key?
   Answer: Kafka uses its partitioning strategy without the semantic grouping benefit of a key, so ordering for related entities is not guaranteed across partitions.

## Common Production Problem Questions

1. What are the most common Kafka problems in production?
   Answer: Consumer lag, hot partitions, duplicate processing, weak durability settings, rebalances, large messages, wrong retention, and replica lag.
2. How do you handle duplicate processing?
   Answer: Make consumers idempotent, store processed IDs if necessary, and commit offsets only after successful processing.
3. How do you handle rebalance issues?
   Answer: Keep consumer groups stable, reduce frequent restarts, and make processing predictable and reasonably fast.
4. How do you handle very large messages?
   Answer: Keep messages small when possible, store large payloads externally, and only tune message-size limits carefully when truly necessary.
5. How do you monitor whether Kafka is healthy?
   Answer: Monitor consumer lag, replica lag, ISR shrink, broker CPU, disk, network, request latency, and produce/fetch rates.

## .NET / C# Kafka Questions

1. Which .NET client library is commonly used for Kafka?
   Answer: `Confluent.Kafka` is the commonly used .NET client.
2. How do you produce a keyed message in C#?
   Answer:

```csharp
var result = await producer.ProduceAsync("orders", new Message<string, string>
{
    Key = "order-1001",
    Value = "OrderCreated"
});
```

3. Why is `EnableIdempotence` useful in a producer?
   Answer: It helps reduce duplicate writes during retries and improves delivery safety.
4. When should you manually commit consumer offsets?
   Answer: When you want to commit only after successful processing, especially for important business workflows.
5. What should you log when consuming Kafka messages?
   Answer: Usually topic, partition, offset, key, group, and processing outcome because these are critical for debugging and replay analysis.

## Recommended Answer Style

For Kafka interview answers:

- start with topic, broker, partition, and offset
- explain ordering only within a partition
- mention keys and consumer groups
- for production questions, talk about bottlenecks and tradeoffs, not just definitions
- for scaling questions, mention partitions, brokers, consumers, key distribution, and monitoring

## Sources

Technical alignment and common question coverage were checked against:

- [Apache Kafka - Introduction](https://kafka.apache.org/08/getting-started/introduction/)
- [Confluent - Introduction to Apache Kafka](https://docs.confluent.io/kafka/introduction.html)
- [Confluent - Kafka Producer Design](https://docs.confluent.io/kafka/design/producer-design.html)
- [Confluent - Kafka Consumer Design](https://docs.confluent.io/kafka/design/consumer-design.html)
- [Confluent - Kafka Replication and Committed Messages](https://docs.confluent.io/kafka/design/replication.html)
- [Apache Kafka - Broker Config Reference (`min.insync.replicas`)](https://kafka.apache.org/39/generated/kafka_config.html)
- [Confluent - Choose and Change the Partition Count in Kafka](https://docs.confluent.io/kafka/operations-tools/partition-determination.html)
- [Confluent - Kafka .NET Client](https://docs.confluent.io/kafka-clients/dotnet/1.5/overview.html)
