

### **ACID Properties in Databases**

The ACID properties are a set of principles that ensure reliable processing of database transactions, even in the face of errors or failures. Each property is critical for maintaining the integrity of data in databases, especially in transactional systems.

---

### **1. Atomicity**

- **Definition**: Ensures that a transaction is treated as a single, indivisible unit of work. Either all operations in the transaction are completed successfully, or none are.
- **Example**:
    - A bank transfer includes debiting one account and crediting another. If the credit operation fails, the debit must be rolled back to maintain consistency.
- **Implementation**:
    - Relying on logs to record changes so incomplete transactions can be undone.

---

### **2. Consistency**

- **Definition**: Guarantees that a transaction transitions the database from one valid state to another, adhering to all defined rules (constraints, triggers, etc.).
- **Example**:
    - Ensuring that the total number of seats sold does not exceed the total number available, even during concurrent transactions.
- **Key Aspect**: Rules and constraints must be checked and enforced after every transaction.

---

### **3. Isolation**

- **Definition**: Ensures that the operations of one transaction are not visible to other transactions until the transaction is committed.
- **Example**:
    - If two users simultaneously try to book the last seat in a theater, isolation ensures that only one succeeds, preventing conflicts.
- **Isolation Levels**:
    - **Read Uncommitted**: Transactions can read uncommitted data from others (may cause "dirty reads").
    - **Read Committed**: Only committed data is visible to transactions.
    - **Repeatable Read**: Ensures the same data is read consistently during a transaction.
    - **Serializable**: Fully isolates transactions, appearing as if they execute sequentially.

---

### **4. Durability**

- **Definition**: Once a transaction is committed, its effects are permanent and will survive system failures (e.g., power outages).
- **Example**:
    - If a customer places an order and the transaction is marked as committed, the database guarantees that the order will not disappear, even if the server crashes immediately afterward.
- **Implementation**:
    - Transactions are often written to persistent storage (e.g., logs, SSDs) before marking them as complete.

---

### **Importance of ACID**

- **Reliability**: Ensures correct data even during unexpected failures.
- **Concurrency Control**: Safeguards data integrity in multi-user environments.
- **Critical Systems**: Widely used in systems where data integrity is crucial, such as banking, inventory management, and e-commerce.




### **Locks in Databases**

Locks are mechanisms used by database management systems (DBMS) to manage access to data, ensuring consistency and integrity during concurrent transactions. They are an essential part of **concurrency control** and help prevent issues like data corruption, inconsistencies, and anomalies such as **dirty reads**, **phantom reads**, and **lost updates**.

---

### **Types of Locks**

#### **1. Shared Lock (S)**

- **Definition**: Allows multiple transactions to read a resource concurrently, but no transaction can write to it.
- **Use Case**:
    - Reading a record while ensuring it’s not modified until the read operation is complete.
- **Example**:
    - Multiple users querying product inventory without making changes.

---

#### **2. Exclusive Lock (X)**

- **Definition**: Ensures that only one transaction can read or write to a resource. Prevents other transactions from accessing the locked resource.
- **Use Case**:
    - Writing or modifying a record.
- **Example**:
    - A transaction updating account balances locks the records exclusively to avoid interference.

---

#### **3. Intent Locks**

![[Pasted image 20241207012319.png]]
- **Definition**: Indicates the intention of a transaction to acquire a lock on a resource. Commonly used in hierarchical locking to prevent conflicts.
- **Types**:
    - **Intent Shared (IS)**: Intends to acquire a shared lock.
    - **Intent Exclusive (IX)**: Intends to acquire an exclusive lock.
- **Use Case**:
    - Prevents deadlocks in table-level and row-level locking hierarchies.

![[Pasted image 20241207012451.png]]





### **Deadlocks in Databases**

A **deadlock** occurs in a database when two or more transactions are waiting for resources held by each other, forming a cycle of dependency that prevents any of them from proceeding. Deadlocks are a major challenge in transactional systems and must be effectively managed to maintain system performance and reliability.

---

### **How Deadlocks Occur**

1. **Transaction A** acquires a lock on **Resource 1** and requests a lock on **Resource 2**, which is held by **Transaction B**.
2. **Transaction B** has a lock on **Resource 2** and requests a lock on **Resource 1**, which is held by **Transaction A**.
3. Neither transaction can proceed because they are waiting for each other, resulting in a deadlock.

---

### **Common Scenarios of Deadlocks**

1. **Concurrent Transactions**:
    
    - Multiple transactions attempting to lock rows in a different order.
2. **Long-Lived Locks**:
    
    - Transactions that hold locks for an extended period, increasing the chances of deadlocks.
3. **Exclusive and Shared Locks**:
    
    - Transactions requesting incompatible locks on the same resource (e.g., one transaction has a shared lock, and another requests an exclusive lock).

---

### **Deadlock Detection and Prevention**

#### **1. Deadlock Detection**

- The DBMS periodically checks for cycles in the **wait-for graph**, which represents transactions waiting for resources.
- If a cycle is detected, one of the transactions is chosen as the "victim" and rolled back to resolve the deadlock.

#### **2. Deadlock Prevention**

- **Resource Ordering**:
    - Enforce a consistent order of resource acquisition across transactions.
- **Timeouts**:
    - Abort transactions if they hold locks for too long.
- **Wait-Die / Wound-Wait Schemes**:
    - Decide which transaction proceeds based on their priorities:
        - **Wait-Die**: Older transactions wait for younger ones; younger transactions are aborted.
        - **Wound-Wait**: Older transactions preempt younger ones, forcing the younger to abort.

---

### **Impacts of Deadlocks**

- **Performance Degradation**:
    - Rolling back transactions wastes resources and increases latency.
- **Reduced Throughput**:
    - Deadlocks block resources, delaying other transactions.
- **User Experience**:
    - Applications waiting on deadlocked queries may appear unresponsive.

---

### **Deadlock Avoidance Techniques**

#### **1. Minimize Lock Scope**

- Lock only necessary rows or tables and avoid broad locks (e.g., table-level locks).

#### **2. Keep Transactions Short**

- Reduce the likelihood of overlap by minimizing transaction duration.

#### **3. Index Usage**

- Proper indexing can reduce the number of rows locked by targeting specific rows.

#### **4. Use Optimistic Concurrency Control**

- Avoid locks entirely by using mechanisms like versioning to ensure consistency.

#### **5. Priority-based Locking**

- Assign priorities to transactions, allowing higher-priority transactions to preempt lower-priority ones.

---

### **Example of a Deadlock in SQL**

#### Scenario:

- **Transaction A**:
    
    ```sql
    BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
    ```
    
- **Transaction B**:
    
    ```sql
    BEGIN;
    UPDATE accounts SET balance = balance + 50 WHERE account_id = 2;
    UPDATE accounts SET balance = balance - 50 WHERE account_id = 1;
    ```
    

#### Result:

- Transaction A locks `account_id = 1` and waits for `account_id = 2`.
- Transaction B locks `account_id = 2` and waits for `account_id = 1`.
- A deadlock occurs.

#### Resolution:

The database's deadlock detector identifies the cycle and aborts one of the transactions (usually the one with fewer changes or lower priority).

---

### **How Deadlocks are Handled in Common DBMS**

#### **1. PostgreSQL**:

- Detects deadlocks via the wait-for graph and aborts one transaction.
- Allows configuration of lock timeouts.

#### **2. MySQL (InnoDB)**:

- Implements deadlock detection and automatically rolls back the "victim" transaction.
- Provides an error (`ER_LOCK_DEADLOCK`) for handling in the application layer.

#### **3. SQL Server**:

- Uses a lock manager to detect deadlocks.
- Offers tools like **Trace Flags** to log deadlocks for troubleshooting.

---

### **Best Practices to Avoid Deadlocks**

1. **Acquire Locks in a Consistent Order**:
    
    - Design transactions to lock resources in the same order across all transactions.
2. **Use Smaller Transactions**:
    
    - Break large transactions into smaller units of work.
3. **Monitor and Optimize Queries**:
    
    - Identify and resolve slow queries that hold locks longer than necessary.
4. **Enable Proper Indexing**:
    
    - Reduce the number of rows involved in a lock by targeting specific data.
5. **Set Lock Timeouts**:
    
    - Avoid indefinite waiting by setting maximum wait times for locks.

