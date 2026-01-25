---
title: Keeping Data True - Your In-Depth Guide to Transaction Processing Systems!
description: Dive deep into Transaction Processing Systems, exploring ACID properties, concurrency control, recovery mechanisms, distributed transactions, and security in electronic commerce.
date: 2023-07-27
draft: false
slug: /pensieve/dbms/transaction-processing-systems
tags:
  - DBMS
  - CS Basics
---

Hey data defenders!  Ever wondered what magic happens when you withdraw cash from an ATM, book a flight online, or make an e-commerce purchase? You expect these operations to complete flawlessly – either fully or not at all – and for your data to remain correct. The systems ensuring this reliability are known as **Transaction Processing Systems (TPS)**.

TPS are the backbone of modern information systems, especially where data integrity and consistency are paramount. They manage and process transactions – sequences of operations that must be executed as a single, indivisible unit. In this post, we'll explore the fundamental principles that make TPS trustworthy, including the famous ACID properties, how they handle multiple users at once, recover from failures, operate across distributed networks, and stay secure. Let's get started!

---
## The ACID Test & Transaction Structures
At the heart of any reliable TPS are the ACID properties, which are like sacred vows for transactions.

* **The Sacred Vows: ACID Properties**

  * **Atomicity**: This means a transaction is "all or nothing." Either all operations within the transaction are completed successfully, and the changes are committed, or if any part fails, the entire transaction is rolled back, leaving the system in its original state. Think of a bank transfer: money must be debited from one account AND credited to another; both must happen, or neither.
  * **Consistency**: A transaction must bring the database from one valid (consistent) state to another valid state. It preserves predefined database rules, such as constraints, triggers, and cascades. For example, if a database rule states that an account balance cannot be negative, a transaction attempting to overdraw would violate consistency (unless allowed by other rules).
  * **Isolation**: When multiple transactions execute concurrently, they should not interfere with each other's operations. The outcome of concurrent execution should be as if the transactions were executed serially (one after another). This prevents issues like dirty reads (reading uncommitted data) or lost updates. The level of isolation can often be configured.
  * **Durability**: Once a transaction has been committed, its effects are permanent and will survive any subsequent system failures, such as power outages or crashes. This is typically achieved by writing changes to non-volatile storage (like a hard disk or SSD) before acknowledging the commit.
* **Transaction Flavors and Forms**:

  * **Flat Transactions**: This is the most common and straightforward model. A flat transaction consists of a sequence of operations that are treated as a single atomic unit. It has a single start point and a single end point (commit or abort).
  * **Providing Structure within a Transaction**:
    * **Savepoints**: Within a flat transaction, a user can define savepoints. If an error occurs after a savepoint, the transaction can be rolled back to that specific savepoint rather than aborting the entire transaction. This allows for more granular error handling within a larger unit of work.
  * **Structuring an Application as Multiple Transactions**: Complex business processes are often too long or involve too many resources to be handled as a single, massive transaction. Instead, they are broken down into a series of smaller, independent flat transactions. For example, booking a holiday might involve separate transactions for flight booking, hotel reservation, and car rental. Ensuring overall consistency across these multiple transactions then becomes an application-level responsibility (often handled by patterns like Sagas in modern distributed systems).

---
## Juggling Acts: Concurrency Control
In real-world systems, many users and applications access data simultaneously. **Concurrency control** mechanisms are essential to allow this parallel execution while upholding the ACID properties, especially Isolation and Consistency.

* **Schedules and Schedule Equivalence**:

  * A **Schedule** (or history) is the chronological order in which the operations of concurrent transactions are interleaved and executed by the system.
  * A **Serial Schedule** is one where transactions are executed one after another, without any interleaving. Serial schedules are inherently correct (they preserve consistency if each individual transaction does) but offer poor performance and throughput.
  * A **Serializable Schedule** is a concurrent schedule whose outcome on the database is equivalent to the outcome of some serial schedule of the same set of transactions. This is the gold standard for correctness in concurrent execution, ensuring that interleaving operations doesn't lead to an inconsistent state. **Conflict serializability** is a common and practical way to achieve serializability by ensuring that the order of any two conflicting operations (operations from different transactions that access the same data item, where at least one is a write) is the same in the concurrent schedule as in some serial schedule.
* **Ensuring Correctness & Cleanliness**:

  * **Recoverability**: A schedule is recoverable if transactions commit only after all transactions whose changes they have read have also committed. This prevents a situation where a transaction (T2) commits based on data written by another transaction (T1) that later aborts. If T2 had committed, its changes would be based on invalid data.
  * **Cascaded Aborts (or Cascading Rollbacks)**: If an uncommitted transaction T1 aborts, any other transaction T2 that read data written by T1 must also be aborted. If a T3 read from T2, it too must abort, and so on. This can lead to a significant amount of wasted work.
  * **Strictness**: A schedule is strict if a transaction can neither read nor write a data item X until the last transaction that wrote X has committed or aborted. Strict schedules avoid cascaded aborts and also simplify the recovery process, as an item's value on disk is never from an uncommitted transaction.
* **Models for Concurrency Control**:

  * **Pessimistic Concurrency Control**: Assumes that conflicts between transactions are likely. It tries to prevent conflicts by acquiring locks on data items before accessing them. If a required lock cannot be obtained, the transaction waits.
  * **Optimistic Concurrency Control**: Assumes that conflicts are rare. Transactions are allowed to proceed without acquiring locks. Before a transaction commits, the system checks if any conflicts have occurred (e.g., if another transaction has modified data it read). If a conflict is detected, the transaction is rolled back and typically restarted.
* **A Strategy for Immediate-Update Pessimistic Concurrency Controls**:

  * Acquire a lock on a data item before reading or writing it.
  * Update the data item in the database buffers immediately (these changes might be flushed to disk later, but they are visible to other transactions that acquire appropriate locks).
  * A common protocol is **Two-Phase Locking (2PL)**:
    1. **Growing Phase**: A transaction can acquire locks but cannot release any locks.
    2. **Shrinking Phase**: A transaction can release locks but cannot acquire any new locks.
       2PL guarantees serializability.
* **Design of an Immediate-Update Pessimistic Concurrency Control**:

  * **Lock Manager**: A component responsible for granting and releasing locks. It maintains a lock table.
  * **Lock Types**:
    * **Shared Locks (Read Locks, S-locks)**: Multiple transactions can hold an S-lock on the same item simultaneously (for reading).
    * **Exclusive Locks (Write Locks, X-locks)**: Only one transaction can hold an X-lock on an item (for writing). An X-lock cannot be acquired if any other lock (S or X) is held on the item.
  * **Lock Granularity**: The size of the data item that a lock protects (e.g., entire database, table, page, row). Finer granularity allows more concurrency but has higher lock management overhead.
  * **Deadlock Detection and Resolution**: Deadlocks can occur when transactions wait for each other to release locks in a circular chain. Systems need to detect deadlocks (e.g., using a waits-for graph) and resolve them (e.g., by aborting one or more transactions).
  * **Strict Two-Phase Locking (Strict 2PL)**: A variation where all locks (S and X) are held until the transaction commits or aborts. This ensures strict schedules, preventing cascaded aborts and simplifying recovery.
* **Beyond Simple Read/Write**:

  * **Objects and Semantic Commutativity**: When dealing with abstract data types (objects with methods), operations might be **semantically commutative** even if they access the same underlying data. For instance, two `Increment()` operations on a counter object are semantically commutative (the final sum is the same regardless of their order), even though both modify the counter. Recognizing semantic commutativity can allow for more concurrency than simple read/write locking.
  * **Atomicity, Recoverability, and Compensating Operations**: For long-running transactions or those structured with internal steps (like in some workflow systems), a full physical rollback might be impractical if a part fails late. A **compensating operation** is a logically defined operation that reverses the effects of a previously committed sub-operation. This is crucial for achieving logical atomicity in models like Sagas.
* **Isolation in Structured Transaction Models**:

  * E.g., **Nested Transactions**: A transaction can be composed of sub-transactions (children). A child can commit or abort, but its commit is relative to its parent. The child's changes only become permanent if the top-level parent transaction commits. Children can run concurrently, and failures within a child don't necessarily abort the parent (the parent can handle the failure). Isolation between siblings and between a child and its parent is maintained.
* **Database Specifics**:

  * **Conflicts in a Relational Database**:
    * **Read-Write (RW) conflict**: T1 reads X, then T2 writes X.
    * **Write-Read (WR) conflict**: T1 writes X, then T2 reads X.
    * **Write-Write (WW) conflict**: T1 writes X, then T2 writes X.
  * **Locking and the SQL Isolation Levels**: SQL standard defines several isolation levels that trade off consistency for concurrency:
    * `READ UNCOMMITTED`: Allows dirty reads (reading uncommitted changes from other transactions), non-repeatable reads, and phantom reads. Lowest isolation, highest concurrency.
    * `READ COMMITTED`: Prevents dirty reads. However, non-repeatable reads (a row read twice in a transaction has different values) and phantom reads (new rows appear in a repeated query) are possible. Often the default.
    * `REPEATABLE READ`: Prevents dirty reads and non-repeatable reads. Phantom reads are still possible (rows matching a query condition might be inserted by another transaction).
    * `SERIALIZABLE`: Highest isolation level. Prevents dirty reads, non-repeatable reads, and phantom reads. Ensures the schedule is serializable.
  * **Granular Locking: Intention Locks and Index Locks**:
    * To manage locks on a hierarchy of data (e.g., database -> table -> page -> row), **Intention Locks** are used. Before placing an S or X lock on a fine-granularity item (like a row), a transaction must acquire an intention lock (IS, IX, or SIX) on its ancestor coarse-granularity items (like the table). This efficiently signals to other transactions the intent to lock at a lower level, preventing conflicting coarse-grained locks.
    * **Index Locks**: Locking parts of an index structure is crucial to prevent phantom reads, especially at the `SERIALIZABLE` isolation level. Techniques like key-range locking can be used.
* **Fine-Tuning and Alternatives**:

  * **Tuning Transactions**: Application developers can improve concurrency by:
    * Keeping transactions short.
    * Accessing data in a consistent order across different transactions to minimize deadlocks.
    * Choosing the lowest acceptable SQL isolation level for a given task.
  * **Multiversion Concurrency Control (MVCC)**: Instead of locking data for readers, MVCC systems maintain multiple versions of data items. When a transaction reads an item, it sees a consistent snapshot of that item as it existed at a certain point in time (often the start of the transaction). Writers create new versions of items. This significantly reduces conflicts between readers and writers, as readers don't block writers and writers don't block readers. Popular in databases like PostgreSQL and Oracle.

---
## Bouncing Back: Recovery Mechanisms
Failures are inevitable. Recovery mechanisms ensure that the database can be restored to a consistent state, upholding Atomicity and Durability.

* **Types of Failures**:

  * **Transaction Failure (Abort)**: A transaction fails to complete due to an internal error, bad input, or deadlock resolution. Its changes must be undone.
  * **System Crash**: The system (hardware or software) fails, causing the contents of volatile memory (RAM) to be lost. Active transactions are interrupted.
  * **Media Failure (Disk Failure)**: The non-volatile storage (disk) where the database resides is damaged or lost. This is the most severe type of failure.
* **Core Recovery Techniques**:

  * **Immediate-Update Systems and Write-Ahead Logs (WAL)**:
    * In immediate-update systems, changes made by a transaction can be written to the database on disk *before* the transaction commits.
    * The **Log** (or journal) is a critical component. It's a sequence of log records stored on stable (non-volatile) storage. Log records describe database operations (e.g., `BEGIN_TRANSACTION`, `WRITE item, old_value, new_value`, `COMMIT`, `ABORT`).
    * **Write-Ahead Logging (WAL) Principle**: This fundamental rule states that before a data item's modification is written to its place on disk (i.e., before the database page is updated), the log record describing that modification must already be written to stable log storage.
    * **Recovery from System Crash**:
      1. The recovery manager scans the log.
      2. It identifies transactions that were active at the time of the crash and those that had committed.
      3. It uses the log to **UNDO** the changes of any transactions that started but did not commit (or explicitly aborted). This involves using the `old_value` from write log records to restore items.
      4. It uses the log to **REDO** the changes of any transactions that committed but whose changes might not have made it to disk due to buffering. This involves using the `new_value` from write log records to reapply changes.
         The order of UNDO/REDO and the specific algorithms (e.g., ARIES) can be complex.
  * **Recovery in Deferred-Update Systems**:
    * In deferred-update systems, changes made by a transaction are only recorded in the log and kept in buffers in memory. They are not written to the database on disk until the transaction commits.
    * **Recovery from System Crash**:
      * No UNDO operations are needed for aborted or incomplete transactions because their changes never reached the disk.
      * Only REDO operations are needed for committed transactions, applying their changes from the log to the database on disk.
        This simplifies recovery but can lead to longer commit times.
  * **Recovery from Media Failure**:
    1. Restore the database from the most recent **archive backup** (a full copy of the database taken at an earlier point).
    2. Use the log files (archived logs and the current online log) to **roll forward** by redoing all committed transactions that occurred since the backup was taken.

---
## Going Global: Distributed Transaction Processing
When transactions need to access and update data across multiple independent databases or systems, often geographically dispersed, we enter the realm of distributed transaction processing.

* **Transaction Processing in a Centralized System vs. Distributed System**:

  * **Centralized**: All data and transaction management reside on a single system. Simpler coordination, concurrency control, and recovery.
  * **Distributed**: Data and processing are spread across multiple autonomous sites connected by a network. Introduces complexities like network latency and unreliability, partial system failures, and the need to coordinate distributed state.
* **The TP Monitor: An Overview**:

  * A **Transaction Processing (TP) Monitor** is software that provides the infrastructure and services to manage transactions in complex, often distributed, application environments.
  * Key functions:
    * Process management (starting server processes).
    * Transaction management (coordinating begin, commit, abort across resources).
    * Inter-process communication (routing requests, handling replies).
    * Name services for locating resources.
    * Enforcing security.
  * Examples from the past include CICS, Tuxedo. Modern application servers (like Java EE servers) and message queue systems often incorporate TP monitor-like capabilities.
* **Ensuring Global Consistency**:

  * **Global Atomicity and the Transaction Manager**: The primary challenge is ensuring that a distributed transaction (one that spans multiple resource managers like databases) either commits at all sites or aborts at all sites. This is global atomicity. A **Transaction Manager (TM)** is responsible for this coordination.
  * **Two-Phase Commit (2PC) Protocol**: The most common protocol for achieving global atomicity.

    1. **Phase 1 (Voting/Prepare Phase)**:
       * The Coordinator (often the TM or the site where the transaction originated) sends a `PREPARE` message to all participating resource managers (Participants).
       * Each Participant determines if it can durably commit its part of the transaction. If yes, it force-writes a `PREPARE` log record (making its state "prepared") and sends a `VOTE_COMMIT` (or "YES") message back to the Coordinator. If no, it sends a `VOTE_ABORT` (or "NO") message (it can abort its part immediately).
    2. **Phase 2 (Decision/Commit Phase)**:
       * If the Coordinator receives `VOTE_COMMIT` from *all* Participants, it force-writes a `GLOBAL_COMMIT` log record and sends a `COMMIT` message to all Participants.
       * If the Coordinator receives *any* `VOTE_ABORT` or a timeout occurs, it force-writes a `GLOBAL_ABORT` log record and sends an `ABORT` message to all Participants.
       * Participants, upon receiving the decision, force-write a `COMMIT` or `ABORT` log record and then perform the action. They then send an acknowledgement to the Coordinator.

    * *Challenges with 2PC*: It's a **blocking protocol**. If the Coordinator fails after initiating Phase 2 but before all Participants receive the decision, Participants that voted YES are blocked (they cannot unilaterally commit or abort) until the Coordinator recovers or a decision can be determined. Similarly, if a Participant fails in the prepared state.
  * **Three-Phase Commit (3PC)**: A more complex protocol designed to be non-blocking under certain failure assumptions (e.g., no network partitions and bounded site failures). It introduces a "pre-commit" phase. While theoretically non-blocking, its complexity and specific failure assumptions have limited its widespread adoption compared to 2PC.
* **Communication in Distributed Transactions**:

  * **Remote Procedure Call (RPC)**: A client makes a call to a procedure that executes on a remote server as if it were a local call. Synchronous by nature.
  * **Peer-to-Peer Communication**: Systems communicate directly with each other as equals, without a strict client-server hierarchy for all interactions.
  * **Event Communication / Message Queuing**: Applications communicate by sending and receiving messages via queues. This is asynchronous and can decouple components, improving resilience to temporary network or component failures. However, integrating message queues into globally atomic transactions adds complexity (e.g., transactional messaging).
* **Storage Architectures**: In distributed systems, data can be:

  * **Partitioned**: The database is split into disjoint segments, each stored at a different site.
  * **Replicated**: Copies of data are stored at multiple sites for availability and performance.
  * **Federated**: Multiple autonomous, heterogeneous databases are integrated under a common schema.
* **Transaction Processing on the Internet**:

  * Challenges: HTTP is inherently stateless. Browser clients are untrusted. Network latency and unreliability are more pronounced. Ensuring security is paramount.
  * Solutions often involve application servers that manage session state, use web services (e.g., REST, SOAP) for communication, and implement robust security measures.
* **Implementing the ACID Properties in Distributed Systems**: Each property faces new challenges:

  * **Atomic Termination**: Handled by distributed commit protocols like 2PC.
  * **Transfer of Coordination**: In case the primary coordinator of a 2PC fails, protocols might exist (often specific to the TP monitor or database system) to elect a new coordinator or for participants to query each other to determine the outcome. This is complex and a weak point of 2PC.
  * **Distributed Deadlock**: Deadlocks can involve a cycle of transactions waiting for locks held by other transactions across multiple sites. Detecting these requires either a centralized global deadlock detector, distributed detection algorithms (where sites exchange waits-for information), or timeout mechanisms.
  * **Global Serialization**: Ensuring that the concurrent execution of distributed transactions is equivalent to some serial execution of those same transactions on a global scale. Distributed concurrency control mechanisms (e.g., distributed 2PL, timestamp ordering) are needed.
* **When Things Get Tricky**:

  * **When Global Atomicity Cannot Be Guaranteed (or is too costly)**: In many modern distributed systems, especially those built with microservices or involving external third-party services, strict 2PC is often considered too slow, too complex, or simply not feasible due to lack of control over all participants. In such cases, alternative patterns are used:
    * **Sagas**: A sequence of local transactions. If any local transaction fails, compensating transactions are executed to undo the work of preceding committed local transactions. Sagas ensure eventual consistency rather than strict atomicity across all steps.
  * **Replicated Databases**: Keeping multiple copies of data consistent is a major challenge.
    * **Eager Replication (Synchronous)**: Updates are propagated to all replicas as part of the committing transaction (e.g., using 2PC with replicas as participants). Strong consistency, but slower writes.
    * **Lazy Replication (Asynchronous)**: Updates are propagated to replicas after the primary transaction commits. Faster writes, but potential for stale reads and eventual consistency.
    * **Quorum-based protocols**: Require reads and writes to involve a certain number (a quorum) of replicas to ensure consistency.
  * **Distributed Transactions in the Real world**: Widely used in core banking systems, airline reservation systems (though often with very sophisticated, specialized TP monitors), and large-scale e-commerce platforms for critical operations. However, there's a growing trend towards eventual consistency models for less critical data or for achieving higher scalability and availability in web-scale applications.

---
## Fort Knox: Security in Transactions ️

Securing transactions and the valuable data they process is non-negotiable.

* **Fundamental Security Services**:

  * **Authentication**: "Who are you?" Verifying the claimed identity of a user, device, or process. Methods include passwords, multi-factor authentication (MFA), security tokens, biometrics, and digital certificates.
  * **Authorization**: "What are you allowed to do?" Once authenticated, authorization mechanisms determine the actions an entity is permitted to perform on specific resources. This is often based on access control lists (ACLs), role-based access control (RBAC), or attribute-based access control (ABAC).
  * **Encryption**: Protecting data confidentiality by transforming it into an unreadable format (ciphertext) that can only be decrypted with a secret key. Applied to data at rest (stored on disk) and data in transit (communicated over networks). Algorithms like AES (Advanced Encryption Standard) are common.
* **Cryptographic Tools**:

  * **Digital Signatures**: Provide:
    * **Authenticity**: Verifies the origin of a message (who sent it).
    * **Integrity**: Ensures the message has not been altered since it was signed.
    * **Non-repudiation**: Prevents the sender from falsely denying they sent the message.
      Created using the sender's private key and verified using their public key (public-key cryptography).
  * **Key Distribution and Authentication**: Securely managing and distributing cryptographic keys is crucial.
    * **Public Key Infrastructure (PKI)**: A system of CAs, RAs, certificates, and policies used to manage public keys and bind them to identities.
    * **Kerberos**: A network authentication protocol that uses symmetric key cryptography and a trusted third party (Key Distribution Center - KDC) to authenticate users to services.
* **Secure Communication**:

  * **Authenticated Remote Procedure Call (RPC)**: Extending RPC mechanisms to include authentication of the caller to the server (and sometimes server to caller) and to ensure the integrity and confidentiality of the parameters and results.
* **Electronic Commerce (E-commerce) Security**: Specific challenges and solutions for online transactions.

  * **The Secure Sockets Layer (SSL) / Transport Layer Security (TLS) Protocol**: The standard for securing web communication (HTTPS). Provides:
    * Encryption of data between browser and server.
    * Server authentication (and optionally client authentication) using digital certificates.
    * Message integrity.
    * **Certificates**: Digital documents issued by trusted **Certificate Authorities (CAs)** that bind a public key to a domain name or organization, verifying the server's identity to the client.
  * **Passport (Historical Context - Single Sign-On - SSO)**: Microsoft Passport was an early attempt at a federated identity/SSO system. While it faced challenges, the concept of SSO is now widely implemented using standards like **SAML (Security Assertion Markup Language)**, **OAuth 2.0** (for delegated authorization), and **OpenID Connect** (for authentication built on OAuth 2.0).
  * **Keeping Credit Card Numbers Private**:
    * **PCI DSS (Payment Card Industry Data Security Standard)**: A set of security standards that organizations handling credit card information must adhere to.
    * **Tokenization**: Replacing sensitive card data with a unique identifier (token) that is not mathematically reversible. The actual card data is stored securely by a payment processor.
    * End-to-end encryption of card details.
  * **The Secure Electronic Transaction (SET) Protocol (Historical Context)**: An early, very comprehensive (and complex) protocol developed by Visa and Mastercard for securing online credit card payments. It aimed to provide confidentiality, integrity, authentication, and non-repudiation for all parties.
    * **Dual Signatures**: A key innovation in SET. A customer would create a dual signature that linked the order information (OI, for the merchant) and the payment instructions (PI, for the bank) in such a way that the merchant could only see the OI and the bank could only see the PI, but both could verify the linkage and authenticity. This provided privacy.
      SET was largely too complex for widespread adoption and was superseded by simpler SSL/TLS-based solutions combined with mechanisms like 3-D Secure (e.g., Verified by Visa, Mastercard SecureCode).
  * **Goods Atomicity, Certified Delivery, and Escrow**: Addressing the trust issues in e-commerce beyond just payment:
    * **Goods Atomicity**: Ensuring that the customer either gets the goods and the merchant gets paid, or neither happens. Difficult to achieve perfectly without trusted intermediaries.
    * **Certified Delivery**: Protocols or services that provide proof that digital or physical goods were delivered to the buyer.
    * **Escrow Services**: A neutral third party holds the buyer's payment until the buyer confirms satisfactory receipt of the goods/services. The escrow then releases payment to the seller.
  * **Electronic Cash (E-cash)**: Systems that attempt to create digital equivalents of physical cash, offering anonymity and direct peer-to-peer payment without intermediaries.
    * **Blind Signatures**: A cryptographic technique invented by David Chaum. It allows a person to get a message (e.g., a digital coin) signed by a bank without the bank learning the content of the message itself. This allows the bank to issue anonymous e-cash that it can later verify as legitimate (and prevent double-spending) without being able to link the spending to the original withdrawer.

---
## Conclusion: The Unsung Heroes of Reliable Computing!
Transaction Processing Systems are the intricate, invisible engines that underpin the reliability and trustworthiness of countless digital interactions that we take for granted every day. From ensuring your bank balance is always correct to making sure your online order goes through smoothly, TPS works diligently in the background.

We've journeyed through:

* The foundational **ACID properties** that are the bedrock of transactional integrity.
* The complex world of **concurrency control**, which allows many users to interact with systems simultaneously without chaos.
* The vital **recovery mechanisms** that ensure data durability and system resilience in the face of failures.
* The challenges and solutions for **distributed transactions** that operate across networks and multiple systems.
* The critical **security measures** needed to protect transactions and sensitive data in an increasingly connected world, especially in electronic commerce.

While often highly technical and operating out of sight, the principles and technologies of Transaction Processing Systems are fundamental to building scalable, reliable, and secure information systems. As our world becomes ever more digital and interconnected, the demand for robust and trustworthy transaction processing will only continue to grow.
