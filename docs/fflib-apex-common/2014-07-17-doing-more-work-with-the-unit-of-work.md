---
title: Doing more work with the Unit Of Work
parent: Apex Commons
nav_order: 5
---
In a previous post I [introduced the Unit Of Work class](http://andyinthecloud.com/2013/06/09/managing-your-dml-and-transactions-with-a-unit-of-work/), which is part of the [Apex Enterprise Patterns series](https://github.com/financialforcedev/fflib-apex-common#this-library). As the original post describes it helps you **simplify your code** when it comes to performing **multiple DML statements over related objects** , as well as providing a **bulkification** and **transaction**  **management**. In this post i would like to share information on a **new feature** that adds some **extensibility** to the [fflib_SObjectUnitOfWork](https://github.com/financialforcedev/fflib-apex-common/blob/master/fflib/src/classes/fflib_SObjectUnitOfWork.cls) class.

This new feature addresses use cases where the work you want it to do, does not fit with the use of the existing **registerDirty** , **registerNew** or **registerDeleted** methods. But you do want that work to only be performed **during the commitWork** method, along with other registered work and within the **same transaction** it manages.

Some examples of such situations are...

- **Sending Emails**  
You want to register the **sending an email** once the work has completed and have that email only be sent if the entire unit of work completes.
- **Upsert**  
You want to perform an **upsert** operation along with other insert, update and delete operations performed by the existing functionality.
- **Database class methods**  
You want to utilise the [**Database class methods**](https://www.salesforce.com/us/developer/docs/apexcode/Content/apex_methods_system_database.htm) for additional functionality and/or granularity on some database operations, e.g. emptyRecycleBin, convertToLead, or perform an insert with allOrNothing set to false.
- **Self referencing objects**  
You want to perform **DML work relating to self referencing objects** , something which is currently not supported by the registerNew and registerRelationship methods.
- **Nested Unit Of Works**  
Though generally not recommended, if you do happen to have created another fflib\_SObjectUnitOfWork instance, you might want to **nest another unit of work instance** &nbsp; **commitWork** call with another.

In these cases the class now contains a **new** [Apex Interface](https://www.salesforce.com/us/developer/docs/apexcode/Content/apex_classes_interfaces.htm), called [IDoWork](https://github.com/financialforcedev/fflib-apex-common/blob/master/fflib/src/classes/fflib_SObjectUnitOfWork.cls#L73), it is an incredibly simple interface!

```java
/**  
 * Interface describes work to be performed during the commitWork method  
 **/
public interface IDoWork  
{  
 void doWork();  
}  
```

To use this interface you implement it and register an instance of the implementing class through the new **registerWork** method. When the&nbsp; **commitWork** method is called it will callback on the **doWork** method for each registered implementations once it has completed the work given to it by the existing methods. In other words after all the DML on the dirty, new or deleted records have been processed.

A [SendEmailWork](https://github.com/financialforcedev/fflib-apex-common/blob/master/fflib/src/classes/fflib_SObjectUnitOfWork.cls#L262) implementation of this interface actually resides internally within the fflib\_SObjectUnitOfWork class and is used to support another new method called, **registerEmail**. I added this to experiment with the feature during development but felt its worth leaving in as an added bonus feature if your writing code thats doing DML and sending emails!

The following example integrates the upsert&nbsp;method on the Database class (which needs a concreate SObject list), but you can **really do anything you want** in the **doWork** method!

```java
 public class UpsertUnitOfWorkHelper implements fflib_SObjectUnitOfWork.IDoWork  
 {  
  public Database.UpsertResult[] Results {get; private set;}
  private List<Account> m_records;

  public UpsertUnitOfWorkHelper()  
  {  
    m_records = new List<Account>();  
  }

  public void registerAccountUpsert(Account record)  
  {  
    m_records.add(record);  
  }

  public void doWork()  
  {  
    Results = Database.upsert(m_records, false);  
  }  
}  
```

The following is an example of this in use...

```java
  // Create Unit Of Work as normal  
  fflib_SObjectUnitOfWork uow =  
  new fflib_SObjectUnitOfWork(  
  new Schema.SObjectType[] {  
  Product2.SObjectType,  
  PricebookEntry.SObjectType,  
  Opportunity.SObjectType,  
  OpportunityLineItem.SObjectType });

  // Register some custom work  
  UpsertUnitOfWorkHelper myUpsertWork = new UpsertUnitOfWorkHelper();  
  uow.registerWork(myUpsertWork);

  // Do standard work via...

  // uow.registerNew(...)

  // uow.registerDeleted(...)

  // uow.registerDirty(...)

  // Register some custom work via...  
  myUpsertWork.registerAccountUpsert(myAccountRecordToUpsert);

  // Commit work as normal  
  uow.commitWork();

  // Optionally, determine the results of the custom work...  
  List<Database.UpsertResult> myUpsertResults = myUpsertWork.Results;  
```

In summary, the **unit of work helps co-ordinate** many things we do that in essence are part of a **transaction of work** we want to happen as a **single unit**. This enhancement helps **integrate other "work" items** your code might have been performing **outside the unit of work into that single unit of work** and transaction. It also helps **bridge some gaps** in the unit of work that exist today such as **self referencing objects**.

Please let me know your thoughts on this new feature and how you see yourself using it!

