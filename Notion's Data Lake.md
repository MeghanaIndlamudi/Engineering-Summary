# Storing 200 Billion Entities: Notion’s Data Lake Project

This project documents the architectural evolution of Notion’s data infrastructure as it scaled from 20 billion blocks in 2021 to over 200 billion blocks by 2024. The increased data volume led Notion to re-architect its data storage and processing solutions to handle hundreds of terabytes efficiently while supporting new product features and analytics requirements.

## Project Overview

Notion’s original data architecture used a sharded Postgres database alongside Fivetran and Snowflake for data processing.

However, the rapid growth and update-heavy nature of Notion’s data posed several challenges:

- **Operational Difficulty**: Managing 480 Fivetran connectors to handle sharded data was complex and time-consuming.
- **Speed and Cost Inefficiency**: Snowflake was slow and costly for Notion's update-heavy workload.
- **Complex Requirements**: Critical features like AI and Search required advanced tree traversal and permission calculations that Snowflake’s SQL interface struggled to support.

To address these issues, Notion developed an in-house data lake using a modern tech stack designed for scalability and efficiency.

## New Data Lake Architecture

The new architecture, implemented using AWS S3 and Apache Hudi, incorporates the following key components:

- **AWS S3**: Used as the primary storage layer for both raw and processed data.
- **Kafka**: Handles real-time data ingestion via Debezium CDC connectors to stream incremental changes from Postgres to the data lake.
- **Apache Hudi**: Optimized for update-heavy workloads, Hudi is used to process and store data in S3 with UPSERT and COPY_ON_WRITE features.
- **Apache Spark**: Employed for distributed data processing and denormalization, with PySpark for lighter tasks and Scala Spark for intensive computations like tree traversal.

### Data Ingestion and Processing Pipeline

1. **Change Data Capture (CDC)**: Debezium captures incremental changes in Postgres and streams them to Kafka topics, where each table is assigned a topic.
2. **Incremental Ingestion**: Kafka messages are consumed by Apache Hudi, which ingests only updated data, maintaining data freshness while minimizing cost and system load.
3. **Post-Processing**: Raw data is stored in S3 as a single source of truth. Transformation, denormalization, and permission data construction (e.g., tree traversal) are performed in Spark before saving processed data for analytics and reporting.

## Key Optimizations

- **Partitioning and Sorting**: Data is partitioned by Postgres shard scheme, and sorted by the event timestamp to improve query efficiency.
- **Bloom Filter Indexes**: Used to optimize data retrieval in Hudi tables.
- **Multi-Threading and Parallel Processing**: Large shards are managed through parallel processing, enhancing performance for high-demand operations.

## Results and Impact

The transition to this new data lake architecture provided significant benefits:

- **Cost Savings**: Over $1 million annually in operational savings since 2022.
- **Ingestion Speed**: Reduced ingestion time from more than a day to minutes or hours.
- **Scalability**: Supported Notion's data doubling rate every 6–12 months.
- **Reliability**: Enabled reliable support for Notion's AI features launched in 2023–2024.

## Tech Stack

- **Storage**: AWS S3
- **Data Processing**: Apache Hudi, Apache Spark (PySpark and Scala Spark)
- **Data Ingestion**: Kafka, Debezium CDC
- **Database**: Postgres (sharded)

## Conclusion

Notion’s data lake project showcases the power of a modern data stack in handling large-scale, update-heavy data while enabling cost-effective, high-performance analytics. By leveraging AWS S3, Kafka, Apache Hudi, and Spark, Notion created a robust and scalable data lake that supports both analytics and critical product features.

---

For further details on Notion’s data infrastructure and design choices, please refer to the official [blog](https://blog.bytebytego.com/p/storing-200-billion-entities-notions).
