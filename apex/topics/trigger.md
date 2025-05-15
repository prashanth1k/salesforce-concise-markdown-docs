# Apex Triggers

Apex triggers enable you to perform custom actions before or after Data Manipulation Language (DML) events (insert, update, delete, undelete) occur on Salesforce records. Triggers are stored as metadata associated with a specific sObject.

## Core Concepts

*   **Purpose:** To automate custom logic in response to data changes. This can include validation, field updates on related records, creating new records, or initiating asynchronous processes.
*   **Events:** Triggers fire on one or more DML events:
    *   `before insert`
    *   `before update`
    *   `before delete`
    *   `after insert`
    *   `after update`
    *   `after delete`
    *   `after undelete`
*   **Context Variables (`Trigger` class):** Static variables available within the trigger's execution context to access runtime information and the records being processed.
    *   `Trigger.new`: A list of the new versions of sObject records (available in insert, update, undelete triggers). Records in `Trigger.new` can be modified *only* in `before` triggers.
    *   `Trigger.old`: A list of the old versions of sObject records (available in update, delete triggers). Read-only.
    *   `Trigger.newMap`: A map of ID to the new versions of sObject records (available in `before update`, `after insert`, `after update`, `after undelete`).
    *   `Trigger.oldMap`: A map of ID to the old versions of sObject records (available in `update`, `delete` triggers).
    *   `Trigger.isExecuting`: `true` if the current context is a trigger.
    *   `Trigger.isInsert`, `Trigger.isUpdate`, `Trigger.isDelete`, `Trigger.isUndelete`: Booleans indicating the DML operation.
    *   `Trigger.isBefore`, `Trigger.isAfter`: Booleans indicating if the trigger is running before or after the DML operation.
    *   `Trigger.size`: The total number of records in the trigger invocation.
    *   `Trigger.operationType`: An enum (`System.TriggerOperation`) indicating the DML operation type.
*   **Bulkification:** Triggers **must** be designed to handle up to 200 records at a time. DML operations and SOQL queries should be performed on collections of records, not within loops iterating over individual records.

## Trigger Syntax

```apex
trigger TriggerName on sObjectName (trigger_events) {
    // Apex code to execute
}
// Example:
trigger AccountBeforeInsertOrUpdate on Account (before insert, before update) {
    for (Account acc : Trigger.new) {
        if (String.isBlank(acc.Description)) {
            acc.Description = 'Default description set by trigger.';
        }
    }
}
```
*   `TriggerName`: The name of the trigger.
*   `sObjectName`: The API name of the sObject the trigger is associated with (e.g., `Account`, `MyCustomObject__c`).
*   `trigger_events`: A comma-separated list of one or more DML events.

## Key Characteristics & Best Practices

1.  **One Trigger Per Object (Recommended):** While technically possible to have multiple triggers per object, it's a best practice to have only one. Use a handler class pattern to manage logic for different DML events and contexts within that single trigger. This improves maintainability and control over the order of execution.
2.  **Logic-less Triggers:** Keep triggers themselves minimal. Delegate complex logic to handler classes (Apex classes).
    ```apex
    // Example: Delegating to a handler
    trigger AccountTrigger on Account (before insert, after update) {
        if (Trigger.isBefore && Trigger.isInsert) {
            AccountTriggerHandler.handleBeforeInsert(Trigger.new);
        }
        if (Trigger.isAfter && Trigger.isUpdate) {
            AccountTriggerHandler.handleAfterUpdate(Trigger.new, Trigger.oldMap);
        }
    }
    ```
3.  **Bulkify Your Code:**
    *   Never place SOQL queries or DML statements inside `for` loops that iterate over `Trigger.new` or `Trigger.old`.
    *   Collect IDs or records in Sets/Lists first, then perform a single SOQL query or DML operation on the collection.
    *   Use Maps (e.g., `Trigger.newMap`, `Trigger.oldMap`, or custom maps keyed by ID) for efficient record lookup and comparison.
4.  **Context-Specific Logic:** Use `Trigger` context variables (e.g., `Trigger.isInsert`, `Trigger.isBefore`) to control which blocks of code execute.
5.  **Preventing DML Operations:** In `before` triggers, use `sObject.addError('Error message')` or `sObjectField.addError('Error message')` to prevent the DML operation and display an error to the user.
6.  **Callouts from Triggers:**
    *   Direct callouts (HTTP or Web Service) are **not allowed** from triggers because they would hold the database transaction open.
    *   To make a callout, invoke an asynchronous Apex method (e.g., an `@future(callout=true)` method or a Queueable job) from the trigger. Pass necessary data (like record IDs) as primitive parameters.
7.  **Recursion Control:** Triggers can cause other records to be updated, which in turn might fire more triggers.
    *   Salesforce has built-in limits to prevent infinite recursion.
    *   Use static Boolean variables in a helper class to prevent a trigger from running more than once in the same transaction if necessary.
    ```apex
    // In a helper class:
    // public static Boolean hasRun = false;

    // In the trigger:
    // if (!MyHelperClass.hasRun) {
    //     MyHelperClass.hasRun = true;
    //     // ... trigger logic ...
    // }
    ```
8.  **Order of Execution:** Salesforce has a defined order of execution for save operations, including validation rules, before triggers, system validation, after triggers, assignment rules, workflow rules, etc. Understand this order to predict trigger behavior.
9.  **Error Handling:** Use `try-catch` blocks for robust error handling, especially around DML operations or complex logic.
10. **Testing:**
    *   Triggers must have test coverage (at least 75% overall Apex coverage, but aim for higher on critical logic).
    *   Test bulk scenarios (e.g., with 200 records).
    *   Test positive and negative cases.
    *   Verify expected outcomes using `System.assertEquals()` and other assertion methods.
    *   Use `Test.startTest()` and `Test.stopTest()` to get a fresh set of governor limits for the code being tested.

## Trigger Context Variable Usage Examples

**Modifying records in `before` triggers:**
```apex
trigger AccountValidation on Account (before insert, before update) {
    for (Account acc : Trigger.new) {
        if (acc.Industry == 'Technology' && String.isBlank(acc.Sic)) {
            acc.Sic = 'TECH-DEFAULT'; // Modifying Trigger.new record directly
        }
    }
}
```

**Accessing old values in `update` or `delete` triggers:**
```apex
trigger AccountFieldChangeLogger on Account (after update) {
    for (Account newAcc : Trigger.new) {
        Account oldAcc = Trigger.oldMap.get(newAcc.Id);
        if (newAcc.AnnualRevenue != oldAcc.AnnualRevenue) {
            // Log the change
            System.debug('Revenue changed for Account ' + newAcc.Id +
                         ' from ' + oldAcc.AnnualRevenue + ' to ' + newAcc.AnnualRevenue);
        }
    }
}
```

**Querying related records (bulkified):**
```apex
trigger OpportunityUpdatesOnAccount on Account (after update) {
    Set<Id> accountIds = Trigger.newMap.keySet();
    List<Opportunity> oppsToUpdate = new List<Opportunity>();

    // Query related Opportunities once
    for (Opportunity opp : [SELECT Id, AccountId, Description
                            FROM Opportunity
                            WHERE AccountId IN :accountIds]) {
        Account updatedAccount = Trigger.newMap.get(opp.AccountId);
        opp.Description = 'Account updated on: ' + System.today() +
                          '. New Account Site: ' + updatedAccount.Site;
        oppsToUpdate.add(opp);
    }

    if (!oppsToUpdate.isEmpty()) {
        update oppsToUpdate;
    }
}
```

