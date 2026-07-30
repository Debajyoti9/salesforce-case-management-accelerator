# Governor Limits in Salesforce Apex

## What are Governor Limits?

Salesforce uses a **multitenant architecture**, where a single Salesforce infrastructure is shared by multiple customers (called **tenants** or **organizations**). Since all organizations share the same computing resources, Salesforce enforces **Governor Limits** to ensure that no single organization consumes excessive resources and affects the performance of others.

### Understanding Multitenancy

Think of Salesforce as an **apartment building**.

* 🏢 **Building** → Salesforce Servers
* 🏠 **Apartment** → One Salesforce Organization (Org)
* ⚡ **Shared Resources** → CPU, Memory, Database, Storage, Network

If one apartment uses all the electricity or water, the remaining apartments will experience problems.

Similarly, if one Salesforce organization consumes excessive CPU time, memory, or database resources, it can negatively impact other organizations running on the same Salesforce infrastructure.

To prevent this, Salesforce introduces **Governor Limits**.

---

## Why are Governor Limits Important?

Governor Limits help Salesforce:

* Ensure fair resource allocation among all organizations.
* Prevent inefficient or poorly written Apex code from monopolizing shared resources.
* Maintain platform stability and performance.
* Prevent infinite loops and runaway processes.
* Encourage developers to write efficient, scalable, and bulkified Apex code.

If Apex code exceeds a governor limit, Salesforce throws an **uncatchable `System.LimitException`**, and the entire transaction is rolled back.

---

## How Governor Limits Work

Whenever Apex code executes, the **Apex Runtime Engine** continuously tracks the resources consumed by the transaction.

Some of the monitored resources include:

* SOQL Queries
* SOSL Queries
* DML Statements
* CPU Time
* Heap Memory
* HTTP Callouts
* Email Invocations
* Queueable Jobs
* Future Methods

If any governor limit is exceeded, Salesforce immediately stops the transaction and throws a runtime exception.

---

## Governor Limits Reset

Governor limits are calculated **per Apex transaction**.

For **Batch Apex**, governor limits are **reset for every execution of the `execute()` method**, allowing Salesforce to process large datasets in manageable batches.

---

# Common Governor Limits

| Governor Limit                                    |  Synchronous Apex  |         Asynchronous Apex        |
| :------------------------------------------------ | :----------------: | :------------------------------: |
| SOQL Queries                                      |         100        |                200               |
| Records Retrieved by SOQL                         |       50,000       |              50,000              |
| Records Retrieved by `Database.getQueryLocator()` |       10,000       |              10,000              |
| SOSL Queries                                      |         20         |                20                |
| Records Retrieved by a Single SOSL Query          |        2,000       |               2,000              |
| DML Statements                                    |         150        |                150               |
| Records Processed by DML                          |       10,000       |              10,000              |
| Trigger Recursion Stack Depth                     |         16         |                16                |
| HTTP/Web Service Callouts                         |         100        |                100               |
| Maximum Callout Timeout                           |     120 Seconds    |            120 Seconds           |
| Future Method Calls                               |         50         | 0 (Batch/Future), 50 (Queueable) |
| Queueable Jobs (`System.enqueueJob`)              |         50         |                 1                |
| `Messaging.sendEmail()` Calls                     |         10         |                10                |
| Heap Size                                         |        6 MB        |               12 MB              |
| CPU Time                                          | 10,000 ms (10 sec) |        60,000 ms (60 sec)        |
| Maximum Apex Transaction Time                     |     10 Minutes     |            10 Minutes            |
| Push Notification Method Calls                    |         10         |                10                |
| Push Notifications per Call                       |        2,000       |               2,000              |
| `EventBus.publish()` Calls                        |         150        |                150               |
| Rows Across Apex Cursors                          |     50 Million     |            50 Million            |
| Apex Cursors per Day                              |       10,000       |              10,000              |
| Cursor Fetch Calls                                |         100        |                100               |
| Cursor Rows (24 Hours)                            |     100 Million    |            100 Million           |
| Rows Across Pagination Cursors                    |       100,000      |              100,000             |
| Pagination Cursor Instances per Transaction       |         50         |                50                |
| Pagination Cursor Instances per 24 Hours          |       200,000      |              200,000             |
| Rows Retrieved per Pagination Cursor Page         |        2,000       |               2,000              |

---

# Example: Exceeding SOQL Limit

### ❌ Bad Practice

Query inside a loop:

```apex
for (Account acc : accounts) {
    Contact con = [
        SELECT Id
        FROM Contact
        WHERE AccountId = :acc.Id
    ];
}
```

If there are more than **100 Accounts**, Salesforce executes more than **100 SOQL queries**, resulting in:

```text
System.LimitException: Too many SOQL queries: 101
```

---

### ✅ Best Practice

Move the query outside the loop.

```apex
List<Contact> contacts = [
    SELECT Id, AccountId
    FROM Contact
    WHERE AccountId IN :accountIds
];

Map<Id, List<Contact>> contactsByAccount = new Map<Id, List<Contact>>();
```

This approach is **bulkified**, more efficient, and avoids governor limit exceptions.

---

# Best Practices to Avoid Governor Limits

* ✅ Never write SOQL queries inside loops.
* ✅ Never perform DML operations inside loops.
* ✅ Bulkify your Apex code.
* ✅ Use Collections (`List`, `Set`, `Map`) efficiently.
* ✅ Use Batch Apex for processing large datasets.
* ✅ Use Queueable Apex or Future Methods for asynchronous processing.
* ✅ Optimize SOQL queries with selective filters.
* ✅ Monitor governor limits using the `Limits` class.
* ✅ Avoid unnecessary trigger recursion.

---

# Monitoring Governor Limits

The `Limits` class allows you to monitor resource consumption during Apex execution.

```apex
System.debug('SOQL Queries Used: ' + Limits.getQueries());
System.debug('SOQL Query Limit: ' + Limits.getLimitQueries());

System.debug('DML Statements Used: ' + Limits.getDmlStatements());
System.debug('DML Statement Limit: ' + Limits.getLimitDmlStatements());

System.debug('CPU Time Used: ' + Limits.getCpuTime());
System.debug('CPU Time Limit: ' + Limits.getLimitCpuTime());

System.debug('Heap Size Used: ' + Limits.getHeapSize());
System.debug('Heap Size Limit: ' + Limits.getLimitHeapSize());
```

---

# Interview Question

### What are Governor Limits?

> Governor Limits are Salesforce-enforced runtime limits that restrict the amount of resources an Apex transaction can consume. Since Salesforce runs on a multitenant architecture where multiple organizations share the same infrastructure, these limits ensure fair resource allocation, maintain platform performance, and prevent inefficient or runaway code from affecting other customers. If an Apex transaction exceeds any governor limit, Salesforce throws an uncatchable `System.LimitException`, and the entire transaction is rolled back.

---

# Key Takeaways

* Salesforce is a **multitenant platform**.
* Governor Limits protect shared resources.
* Limits are enforced by the **Apex Runtime Engine**.
* Governor limits apply **per transaction**.
* Batch Apex gets fresh limits for each `execute()` invocation.
* Exceeding a governor limit throws an **uncatchable `System.LimitException`**.
* Writing **bulkified and optimized Apex code** is the best way to stay within governor limits.

# 🧠 How to Remember Salesforce Governor Limits

One of the biggest mistakes beginners make is trying to memorize **every governor limit**.

**Don't do that.**

Even experienced Salesforce developers don't remember all of them. They memorize the **most commonly used limits** and refer to the documentation when needed.

---

# 🎯 Step 1: Memorize the Golden Numbers

These **9 limits** cover around **90% of interview questions** and day-to-day development.

| Governor Limit          |          Value |
| ----------------------- | -------------: |
| SOQL Queries            |        **100** |
| DML Statements          |        **150** |
| SOQL Records            |     **50,000** |
| DML Records             |     **10,000** |
| Callouts                |        **100** |
| CPU Time                | **10 Seconds** |
| Heap Size               |       **6 MB** |
| Trigger Recursion Depth |         **16** |
| Email Invocations       |         **10** |

---

# 🧩 Step 2: Remember by Category

Instead of memorizing individual limits, group them.

## 📖 Database Limits

```text
SOQL Queries      → 100
SOSL Queries      → 20
SOQL Records      → 50,000
```

---

## ✍️ DML Limits

```text
DML Statements    → 150
DML Records       → 10,000
```

---

## 🌐 Integration Limits

```text
Callouts          → 100
Timeout           → 120 Seconds
```

---

## ⚙️ System Limits

```text
CPU Time          → 10 Seconds
Heap Size         → 6 MB
Execution Time    → 10 Minutes
```

---

## 📧 Miscellaneous

```text
Emails            → 10
Trigger Depth     → 16
```

---

# 🎭 Step 3: Learn with a Story

Imagine you're writing Apex code.

> 📖 I ask **100** questions (SOQL)

> ✍️ Then I make **150** database changes (DML)

> 📚 I can read **50K** records

> 📝 I can modify **10K** records

> 🌐 I call **100** external APIs

> ⏱️ Salesforce gives me **10 seconds** to finish

> 💾 I get only **6 MB** of memory

> 🔁 My trigger can recurse only **16** times

> 📧 Finally, I can send only **10 emails**

Stories are much easier for the brain to remember than isolated numbers.

---

# 🔢 Step 4: Remember the Magic Numbers

Instead of remembering 30+ limits, remember only these numbers.

```text
6
10
16
50
100
150
```

Everything else is built around these numbers.

| Number  | Think About                          |
| ------- | ------------------------------------ |
| **6**   | Heap Size                            |
| **10**  | CPU Seconds, Emails, 10K DML Records |
| **16**  | Trigger Recursion                    |
| **50**  | 50K SOQL Records                     |
| **100** | SOQL Queries, Callouts               |
| **150** | DML Statements                       |

---

# 🚀 Step 5: Async Apex Trick

Remember one simple rule:

> **Asynchronous Apex gets more resources.**

| Limit        | Synchronous | Asynchronous |
| ------------ | ----------: | -----------: |
| SOQL Queries |         100 |          200 |
| Heap Size    |        6 MB |        12 MB |
| CPU Time     |  10 Seconds |   60 Seconds |

You don't need to memorize every async limit.

Just remember:

> **Async = Bigger Limits**

---

# 📝 Step 6: Daily Revision (5 Minutes)

Test yourself every day.

**Q:** Maximum SOQL Queries?

> **100**

---

**Q:** Maximum DML Statements?

> **150**

---

**Q:** Maximum CPU Time?

> **10 Seconds**

---

**Q:** Maximum Heap Size?

> **6 MB**

---

**Q:** Maximum SOQL Records?

> **50,000**

---

**Q:** Maximum Trigger Depth?

> **16**

---

Repeat this for a week and these numbers will become second nature.

---

# 🎤 Interview Tip

If an interviewer asks:

> **"Do you remember all governor limits?"**

A good answer is:

> "I remember the commonly used limits such as SOQL queries, DML statements, CPU time, heap size, callouts, and record limits because I use them frequently. For less common platform-specific limits, I refer to the Salesforce documentation when needed."

This shows practical experience instead of rote memorization.

---

# ⭐ Golden Numbers Cheat Sheet

```text
SOQL Queries         → 100
DML Statements       → 150
SOQL Records         → 50K
DML Records          → 10K
Callouts             → 100
CPU Time             → 10 sec
Heap Size            → 6 MB
Trigger Depth        → 16
Email Invocations    → 10
```

## 🎯 Remember This Sequence

```text
100 → 150 → 50K → 10K → 100 → 10s → 6MB → 16 → 10
```

If you can recall this sequence in under **10 seconds**, you're well prepared for most Salesforce developer interviews on governor limits.

