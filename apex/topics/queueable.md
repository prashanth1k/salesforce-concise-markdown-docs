# Queueable Apex

Queueable Apex allows for asynchronous execution of Apex code, similar to future methods but with enhanced capabilities. It's designed for operations that might take longer to process, need to run in a separate thread, or require more complex data types than future methods allow.

## Core Concepts

- **Purpose:** To run processes asynchronously, offloading work from the main transaction thread. This is useful for long-running operations, callouts after DML, or tasks that need higher governor limits.
- **Interface:** A class becomes Queueable by implementing the `System.Queueable` interface.
- **`execute` Method:** The `Queueable` interface requires implementing one method: `public void execute(QueueableContext context)`. This method contains the logic to be executed asynchronously.
- **Enqueuing:** Jobs are submitted to the Apex job queue using `System.enqueueJob(new MyQueueableClass(...))`.

## Key Features & Benefits

- **Non-Primitive Types:** Unlike `@future` methods, Queueable classes can have member variables of non-primitive data types (e.g., sObjects, custom Apex types). These variables are passed to the `execute` method through the class constructor.
- **Job ID & Monitoring:** `System.enqueueJob()` returns an ID (`AsyncApexJob` ID) which can be used to monitor the job's status and progress, either programmatically (querying `AsyncApexJob`) or via the Salesforce UI (Apex Jobs page).
- **Chaining Jobs:** A Queueable job can enqueue another Queueable job from within its `execute` method. This allows for sequential processing.
  - Only one child job can be chained from a parent job.
  - Stack depth for chained jobs can be configured (default limit of 5 in Developer/Trial Editions, higher in others).
- **Callouts:** Supports HTTP callouts if the Queueable class also implements the `Database.AllowsCallouts` marker interface.

## Implementation Example

```apex
public class MyQueueableJob implements Queueable {
    private List<Account> accountsToProcess;
    private String statusToSet;

    public MyQueueableJob(List<Account> accounts, String status) {
        this.accountsToProcess = accounts;
        this.statusToSet = status;
    }

    public void execute(QueueableContext context) {
        // Processing logic for accounts
        for (Account acc : accountsToProcess) {
            acc.Status__c = this.statusToSet;
            // Potentially make a callout here if Database.AllowsCallouts is implemented
        }
        update accountsToProcess;

        // Example of chaining another job (if needed)
        // ID nextJobId = System.enqueueJob(new AnotherQueueableJob());
    }
}

// How to enqueue:
// List<Account> accs = [SELECT Id FROM Account WHERE Name = 'Test'];
// ID jobId = System.enqueueJob(new MyQueueableJob(accs, 'Processed'));
```

## Testing Queueable Apex

- Wrap the `System.enqueueJob()` call within `Test.startTest()` and `Test.stopTest()`.
- This ensures the asynchronous job executes synchronously within the test method _after_ `Test.stopTest()` is called.
- Assertions can then be made on the results of the job.

```apex
@isTest
static void testMyQueueableJob() {
    // Setup test data
    List<Account> testAccounts = new List<Account>();
    // ... (create and insert test accounts)

    Test.startTest();
    ID jobId = System.enqueueJob(new MyQueueableJob(testAccounts, 'TestProcessed'));
    Test.stopTest(); // Job executes here

    // Verify results
    // List<Account> updatedAccounts = [SELECT Status__c FROM Account WHERE Id IN :testAccounts];
    // System.assertEquals('TestProcessed', updatedAccounts[0].Status__c);
}
```

## Governor Limits

- **Async Apex Limit:** Each queued job counts once against the shared limit for asynchronous Apex method executions (e.g., 250,000 per 24 hours, or user licenses \* 200).
- **Queue Size:** Up to 50 jobs can be added to the queue with `System.enqueueJob` in a single transaction.
- **Stack Depth for Chaining:** Default of 5 in Developer/Trial orgs (initial job + 4 chained). Can be overridden using `AsyncOptions`.
- **Governor Limits within Job:** The `execute` method gets its own set of governor limits, generally higher than synchronous Apex (e.g., heap size, CPU time).

## Duplicate Job Detection

- To prevent enqueuing duplicate jobs with the same logical work, use `QueueableDuplicateSignature.Builder` to create a unique signature for a job.
- Pass this signature via `AsyncOptions` to `System.enqueueJob()`.
- If a job with the same signature is already in the queue, a `DuplicateMessageException` is thrown.

```apex
// Example with duplicate detection
AsyncOptions asyncOpt = new AsyncOptions();
asyncOpt.DuplicateSignature = QueueableDuplicateSignature.Builder()
    .addString('ProcessAccount:' + accountId) // Example signature component
    .build();

try {
    ID jobId = System.enqueueJob(new MyQueueableJob(accounts, 'Processed'), asyncOpt);
} catch (DuplicateMessageException e) {
    System.debug('Job already enqueued: ' + e.getMessage());
}
```

## Transaction Finalizers (`System.Finalizer` interface)

- Attach actions to a Queueable job that execute after the job completes, regardless of success or failure.
- Useful for cleanup, logging, or retry logic.
- A Queueable job and its Finalizer run in separate transactions.
- A Finalizer can re-enqueue a failed Queueable job up to 5 consecutive times.
- Implement `System.Finalizer` and attach using `System.attachFinalizer(new MyFinalizer())` within the Queueable job's `execute` method.

## When to Use Queueable Apex

- When an asynchronous operation needs to pass non-primitive data types (sObjects, custom types) to its execution logic.
- When you need a Job ID to monitor the asynchronous job's progress.
- For sequential asynchronous processing by chaining jobs.
- When operations require higher governor limits than synchronous Apex or `@future` methods.
- For callouts that need to happen after DML operations (ensuring the DML is committed before the callout).
