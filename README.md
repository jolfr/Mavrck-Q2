# Mavrck Architecture Exercise

## Prompt

### Assume we have two continuous data streams

- Stream A

  - schema : { unique_identifier, number, text }
  - velocity : 100k records per day
  - cardinality : bounded at around 3M ( so, in 30 days we update all records )

- Stream B
  - schema : { unique_identifier, Stream_A_unique_identifier, number, text }
  - these are essentially child objects of Stream A
  - velocity : 1M records per day
  - cardinality : unbounded ~ although, of the 1M / day, 80% are overlapping

### Requirements

- support realtime, or near-realtime event notifications ( looking for some signal in the text )
- support rolling aggregates over a period of time
  - number of Schema B items in the past N hours
  - number of Schema B w/ condition in past N hours
- support lookup of Stream A assets by by identifier
- support lookup of Stream B assets by Stream_A identifiers
- support analytics projects for Stream A/B ( ie : custom analytics for "data-science" explorations that don't bring down the system )

## *Approach:*

### Functional Components

- Ingestion Layer - ingests data streams from external sources
- Real-Time Processing Layer - signal event notifications
- Storage Layer - document storage due to semi-struc nature of data
- Async Analytics Layer - queryable interface to suport lookup/aggregations/analytics

![Stream Process Diagram](https://github.com/jolfr/Mavrck-Q2/blob/main/stream_processing.png)

### Services

**NOTE:** All services with the exception of Slack (which interacts through an SDK) are procured from Amazon Web Services.

 -[Kinesis Data Streams](https://aws.amazon.com/kinesis/data-streams/) _(Ingestion, Real-Time Proc, Storage)_ Highly scalable streaming service, the ingestion layer uses generic instances, but ulike the two other offerings I have used in this architecture, it is not serverless so will require defintion of instances and scaling rules. Kinesis Firehose is more specialized, performing direct streaming to the DB instance(s) once Data Stream has delegated to it. It fits well for Schema A, but not for Schema B due to the need to check for overlaps before writing to the DB. This will have to be a custom implementation of the more generic instance. Signal Proc is handled by Kinesis Analytics, which will require implementation of code but it is made easier by AWS' SDKs. 

 -[DynamoDB](https://aws.amazon.com/dynamodb/) _(Storage)_ Document based storage made sense for this use case, as the data is semi-structured text. A more traditiional alternative would've been to set up a relational database with pointers to various S3 blobs. This seperation of the data can now be avoided if one so chooses.

 -[Lambda Functions](https://aws.amazon.com/lambda/) _(Async Analytics)_ The majority of the code work would happen implementing these functions. Data can be queried through DynamoDb's SDK, or directly from the Ingestion Layer's memory
