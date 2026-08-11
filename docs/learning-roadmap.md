# System Design Learning Roadmap

This document maps your existing system design notes to a structured learning path.

---

### Building Blocks

These are the fundamental components used to construct large-scale systems.

#### 9. Databases

- **Your Notes**: [Databases](./services/databases.md)
- **Topics Covered**: Types of Databases, Data Replication, Data Partitioning, Trade-Offs.

#### 10. Key-Value Store

- **Your Notes**: [Dynamic Configuration Service Design](./dynamic-config-system-design.md) (uses a Key-Value pattern with MySQL/Redis)
- **Topics Covered**: Scalability, Replication, Caching, Fault Tolerance.

#### 16. Distributed Cache

- **Your Notes**: [Redis Cache](./services/redis-cache.md)
- **Topics Covered**: Caching patterns (profile, feed), data structures, distributed locks, persistence, and HA.

#### 17. Distributed Messaging Queue

- **Your Notes**: [Kafka Message Queue](./services/kafka-message-queue.md)
- **Topics Covered**: Core concepts (Producers, Topics, Consumers), configuration, and use cases for high-throughput ingestion.

#### 18. Pub-Sub

- **Your Notes**:
    - [Event-Based Triggers](./services/event-based-triggers.md)
    - [Kafka Message Queue](./services/kafka-message-queue.md)
- **Topics Covered**: Event-driven architecture, decoupling services, and real-time notifications.

#### 22. Distributed Logging

- **Your Notes**: [Centralized Logging System Design](./centralized-logging-system-design.md)
- **Topics Covered**: Log ingestion, processing pipelines, storage (Elasticsearch), and querying.

#### Other Building Blocks

- **Kafka Streams**: Your notes on [Kafka Streams](./services/kafka-streams.md) cover real-time stream processing, which is a powerful building block for deduplication, aggregation, and enrichment.
- **Connection Pooling**: Your notes on [Connection Pool Management](./services/connection-pool.md) are fundamental for building efficient, high-throughput services.

---

### System Design Problems

These are practical applications of the building blocks to solve real-world problems.

#### 27. Design Quora / 32. Design Newsfeed System

- **Your Notes**: [Google News System Design](./google-news-system-design.md)
- **Concepts Applied**: This design is highly relevant as it covers article ingestion, clustering, classification, and serving personalized feeds, which are core to both Quora and Newsfeed systems.

#### 30. Design Uber

- **Your Notes**:
    - [Hotel Booking System Design](./hotel-booking-system-design.md)
    - [Fraudulent Credit Card Detection System Design](./fraudulent-card-detection-design.md)
- **Concepts Applied**: Your hotel booking design covers reservations and inventory management, similar to ride booking. The fraud detection design is a key component of any transactional platform like Uber.

---

## Full Learning Roadmap (with your files integrated)

1.  **Introduction**
2.  **System Design Interviews**
3.  **Preliminary System Design Concepts**
4.  **Non-Functional System Characteristics**
5.  **Back-of-the-Envelope Calculations**
6.  **Building Blocks**
7.  **Domain Name System**
8.  **Load Balancers**
9.  **Databases** -> [databases.md](./services/databases.md)
10. **Key-Value Store** -> See [dynamic-config-system-design.md](./dynamic-config-system-design.md)
11. **Content Delivery Network (CDN)**
12. **Sequencer**
13. **Distributed Monitoring**
14. **Monitor Server-Side Errors**
15. **Monitor Client-Side Errors**
16. **Distributed Cache** -> [redis-cache.md](./services/redis-cache.md)
17. **Distributed Messaging Queue** -> [kafka-message-queue.md](./services/kafka-message-queue.md)
18. **Pub-Sub** -> [event-based-triggers.md](./services/event-based-triggers.md)
19. **Rate Limiter**
20. **Blob Store**
21. **Distributed Search**
22. **Distributed Logging** -> [centralized-logging-system-design.md](./centralized-logging-system-design.md)
23. **Distributed Task Scheduler**
24. **Sharded Counters**
25. **Concluding the Building Blocks Discussion**
26. **Design YouTube**
27. **Design Quora** -> See [google-news-system-design.md](./google-news-system-design.md)
28. **Design Google Maps**
29. **Design a Proximity Service/Yelp**
30. **Design Uber** -> See [hotel-booking-system-design.md](./hotel-booking-system-design.md) and [fraudulent-card-detection-design.md](./fraudulent-card-detection-design.md)
31. **Design Twitter**
32. **Design Newsfeed System** -> See [google-news-system-design.md](./google-news-system-design.md)
33. **Design Instagram**
34. **Design a URL Shortening Service/TinyURL**
35. **Design a Web Crawler**
36. **Design WhatsApp**
37. **Design Typeahead Suggestion**
38. **Design a Collaborative Document Editing Service/Google Docs**
39. **Design a Deployment System**
40. **Design a Payment System** -> See [fraudulent-card-detection-design.md](./fraudulent-card-detection-design.md)
41. **Design a ChatGPT System**
42. **Design a Data Infrastructure System**
43. **LLM-Powered Customer Support Bot System Design**
44. **AI-Powered Code Assistant System Design**
45. **Lessons from System Failures**
46. **Concluding Remarks**
47. **Free System Design Lessons**
48. **System Design Case Studies**