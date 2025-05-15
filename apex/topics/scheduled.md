# Scheduled Apex

Scheduled Apex allows you to execute Apex classes at specified times or on a recurring schedule. This is ideal for routine maintenance tasks, daily/weekly processing, or any Apex logic that needs to run automatically without user intervention.

## Core Concepts

*   **Purpose:** To run Apex code at predetermined future times or on a regular schedule (e.g., nightly, weekly).
*   **Interface:** A class becomes schedulable by implementing the `System.Schedulable` interface.
*   **`execute` Method:** The `Schedulable` interface requires implementing one method: `global void execute(SchedulableContext sc)`. This method contains the logic that will be executed when the job runs.
    *   The `SchedulableContext sc` parameter provides context about the scheduled job, such as its ID.
*   **Scheduling:**
    *   **Programmatically:** Use `System.schedule(jobName, cronExpression, schedulableClassInstance)` to schedule a job.
    *   **UI:** Schedule jobs via the Salesforce UI (Setup > Apex Classes > Schedule Apex).
*   **CRON Expression:** A string that defines the schedule (e.g., "0 0 22 ? \* MON-FRI" for 10 PM every weekday).
*   **Job Monitoring:** Scheduled jobs can be monitored via the "Scheduled Jobs" and "Apex Jobs" pages in Salesforce Setup, or by querying the `CronTrigger` and `CronJobDetail` sObjects.

## Implementation Example

```apex
// Schedulable class example
global class MyScheduledJob implements Schedulable {
    global void execute(SchedulableContext sc) {
        // Logic to be executed on schedule
        // For example, call a batch Apex job:
        // MyBatchableClass batchJob = new MyBatchableClass();
        // Database.executeBatch(batchJob);

        System.debug('Scheduled Apex job executed. Job ID: ' + sc.getTriggerId());
    }
}

// How to schedule programmatically (e.g., in Developer Console)
// MyScheduledJob jobInstance = new MyScheduledJob();
// String jobName = 'My Daily Maintenance Job';
// String cronExpression = '0 0 1 * * ?'; // Run daily at 1 AM
// String jobId = System.schedule(jobName, cronExpression, jobInstance);
// System.debug('Scheduled job ID: ' + jobId);
```

## CRON Expression Syntax

The CRON expression is a string with 7 space-separated fields:

`Seconds Minutes Hours Day_of_month Month Day_of_week Optional_year`

| Field           | Allowed Values      | Special Characters |
| :-------------- | :------------------ | :----------------- |
| Seconds         | 0-59                | , - \* /           |
| Minutes         | 0-59                | , - \* /           |
| Hours           | 0-23                | , - \* /           |
| Day_of_month    | 1-31                | , - \* ? / L W     |
| Month           | 1-12 or JAN-DEC     | , - \* /           |
| Day_of_week     | 1-7 or SUN-SAT      | , - \* ? / L #     |
| Optional_year   | empty or 1970-2099  | , - \* /           |

**Special Characters:**

*   `*` (asterisk): "all values" (e.g., `*` in Hours field means "every hour").
*   `?` (question mark): "no specific value" (useful for Day_of_month or Day_of_week when the other is specified).
*   `-` (hyphen): Specifies a range (e.g., `10-12` in Hours field means 10 AM, 11 AM, and 12 PM).
*   `,` (comma): Specifies additional values (e.g., `MON,WED,FRI` in Day_of_week field).
*   `/` (slash): Specifies increments (e.g., `0/15` in Seconds field means 0, 15, 30, and 45).
*   `L`: "last". In Day_of_month, it's the last day of the month. In Day_of_week, it's the last day of the week (Saturday). When used with another day (e.g., `FRI L` or `5L`), it means "the last Friday of the month."
*   `W`: "weekday". `15W` means the nearest weekday to the 15th of the month.
*   `#`: "nth". `FRI#3` or `5#3` means the third Friday of the month.

**Examples:**

*   `0 0 22 ? * MON-FRI`: 10 PM every Monday through Friday.
*   `0 0 1 * * ?`: 1 AM every day.
*   `0 0 0 15 * ?`: Midnight on the 15th day of every month.

## Key Considerations

*   **System Context:** Scheduled Apex runs in system context, meaning it bypasses user permissions and field-level security. Sharing rules are enforced based on the class's `with sharing` or `without sharing` keywords.
*   **Governor Limits:** Synchronous Apex governor limits apply to scheduled jobs (not asynchronous limits, even though it runs asynchronously relative to user interaction).
*   **Scheduling Limits:**
    *   Maximum 100 scheduled Apex jobs at one time.
    *   Maximum 250,000 asynchronous Apex executions (Batch, Future, Queueable, Scheduled) per 24-hour period, or user licenses \* 200, whichever is greater.
*   **No Callouts (Directly):** Synchronous Web service callouts are not supported directly from Scheduled Apex. To make callouts:
    *   Use Queueable Apex (implementing `Database.AllowsCallouts`) and schedule the Queueable class.
    *   Call an `@future(callout=true)` method from the scheduled class.
    *   Schedule a Batch Apex job (which can make callouts) from the scheduled class.
*   **Concurrency:** While jobs are scheduled for specific times, actual execution can be delayed based on service availability.
*   **State:** The Schedulable class instance is instantiated when scheduled. If the class has member variables, their state at the time of scheduling persists across executions unless `Database.Stateful` is used (relevant if the scheduled job itself is a Batch Apex job that also implements `Schedulable`). Use `transient` keyword for variables that shouldn't persist.
*   **Testing:**
    *   Wrap the `System.schedule()` call and any subsequent DML/queries related to its execution within `Test.startTest()` and `Test.stopTest()`.
    *   This ensures the scheduled job executes synchronously within the test *after* `Test.stopTest()` is called.
    *   Query `CronTrigger` and `CronJobDetail` sObjects to verify scheduling and job details.

## Scheduling Batch Apex

A common pattern is to schedule a Batch Apex job.

1.  **Directly using `System.scheduleBatch`:**
    ```apex
    // Example: Schedule a batch job to run once, 60 minutes from now
    // MyBatchClassName batchInstance = new MyBatchClassName();
    // String jobId = System.scheduleBatch(batchInstance, 'My Batch Job Name', 60);
    ```
    This method takes the batch class instance, a job name, and the delay in minutes. An optional `scope` (batch size) can also be provided.

2.  **Using a Schedulable class to invoke Batch Apex:**
    The `execute` method of your Schedulable class can instantiate and execute a Batch Apex job. This is useful for recurring batch jobs.
    ```apex
    // In MyScheduledJob.cls execute method:
    // MyBatchableClass batchJob = new MyBatchableClass();
    // Database.executeBatch(batchJob);
    ```

## Aborting Scheduled Jobs

Use `System.abortJob(jobId)` to stop a scheduled job from running. The `jobId` is the ID of the `CronTrigger` record.

