---
title: FinancialForce Apex Common Community Updates
parent: Apex Commons
nav_order: 11
---
![]({{ site.baseurl }}/assets/images/contributions-e1549239445378.png?w=300)

This short blog highlights a batch of new features recently merged to the [FinancialForce Apex Common library aka fflib](https://github.com/financialforcedev/fflib-apex-common#this-library). In addition to the various Dreamforce and blog resources linked from the repo, fans of&nbsp;[Trailhead](https://trailhead.salesforce.com/en/home)&nbsp;can also find modules relating to the library [here](https://trailhead.salesforce.com/en/content/learn/modules/apex_patterns_sl) and [here](https://trailhead.salesforce.com/en/content/learn/modules/apex_patterns_dsl). But please read this blog first before heading out to the trails to hunt down badges!&nbsp;It's really pleasing to see it continue to get great contributions so here goes...

**Added methods for detecting changed records with given fields in the Domain layer (fflib\_SObjectDomain)**

First up is a great new optimization feature for your Domain class methods from **Nathan Pepper** aka [MayTheSForceBeWithYou](https://github.com/MayTheSForceBeWithYou)&nbsp;based on a suggestion by [Daniel Hoechst](https://github.com/dhoechst). Where applicable its a good optimization practice to considering comparing the old and new values of fields relating to processing you are doing in your Domain methods to avoid unnecessary overheads. The new **fflib\_SObjectDomain.getChangedRecords** method can be used as an alternative to the Records property to just the records that have changed based on the field list passed to the method.

```java
// Returns a list of Account where the Name or AnnaulRevenue has changed  
List<Account> accounts =  
 (List<Account>) getChangedRecords(  
 new List<SObjectField> { Account.Name, Account.AnnualRevenue });  
```

**Supporting EventBus.publish(list\<SObject\>) in Unit of Work (fflib\_SObjectUnitOfWork)**

[Platform Events](https://developer.salesforce.com/docs/atlas.en-us.platform_events.meta/platform_events/platform_events_intro_emp.htm) are becoming ever popular in many situations. If you regard them as logically part of the unit of work your code is performing, this enhancement from&nbsp;[Chris Mail](https://github.com/Autobat)&nbsp;is for you! You can now register platform events to be sent based on various scenarios. Chris has also provided bulkified versions of the following methods, nice!

```java
 /**  
  * Register a newly created SObject (Platform Event) instance to be published when commitWork is called  
  *  
  * @param record A newly created SObject (Platform Event) instance to be inserted during commitWork  
  **/  
 void registerPublishBeforeTransaction(SObject record);  
 /**  
  * Register a newly created SObject (Platform Event) instance to be published when commitWork has successfully  
  * completed  
  *  
  * @param record A newly created SObject (Platform Event) instance to be inserted during commitWork  
  **/  
 void registerPublishAfterSuccessTransaction(SObject record);  
 /**  
  * Register a newly created SObject (Platform Event) instance to be published when commitWork has caused an error  
  *  
  * @param record A newly created SObject (Platform Event) instance to be inserted during commitWork  
  **/  
 void registerPublishAfterFailureTransaction(SObject record);  
```

**Add custom DML for Application.UnitOfWork.newInstance call (fflib\_Application)**

It's been possible for a while now to override the default means by which the **fflib\_SObjectUnitOfWork.commitWork** method performs DML operations (for example if you wanted to do some additional pre/post processing or logging). However, if you have been using the [Application class pattern to access your UOW](https://andyinthecloud.com/2015/03/22/unit-testing-with-apex-enterprise-patterns-and-apexmocks-part-1/) (shorthand and helps with mocking) then this has not been possible. Thanks to [William Velzeboer](https://github.com/wimvelzeboer) you can now get the best of both worlds!

```java
fflib_SObjectUnitOfWork.IDML myDML = new MyCustomDMLImpl();  
fflib_ISObjectUnitOfWork uow = Application.UnitOfWork.newIntance(myDML);  
```

**Added methods to Unit of Work to be able to register record for upsert (fflib\_SObjectUnitOfWork)**

[Unit Of Work](https://andyinthecloud.com/2014/07/17/doing-more-work-with-the-unit-of-work/) is a very popular class and receives yet another enhancement in this batch from&nbsp;[Yury Bondarau](https://github.com/yurybond). These two methods allow you to register records that will either be inserted or updated as automatically determined by the records having an Id populated or not, aka a UOW upsert.

```java
 /**  
  * Register a new or existing record to be inserted or updated during the commitWork method  
  *  
  * @param record An new or existing record  
  **/  
 void registerUpsert(SObject record);  
 /**  
  * Register a list of mix of new and existing records to be upserted during the commitWork method  
  *  
  * @param records A list of mix of existing and new records  
  **/  
 void registerUpsert(List&lt;SObject&gt; records);  
 /**  
  * Register an existing record to be deleted during the commitWork method  
  *  
  * @param record An existing record  
  **/  
```

**Alleviates unit-test exception when Org's email service is limited**

Finally, long term mega fan of the library [John Storey](https://github.com/stohn777) comes in with an ingenious fix to an Apex test failure which&nbsp;occurs when the org's email deliverability's 'Access Level' setting is not 'All Email'. John leveraged an extensibility feature in the Unit Of Work to avoid the test being dependent on this org config all&nbsp; **while not losing any code coverage** , sweet!

Last but not least, thank you&nbsp;[Christian Coleman](https://github.com/christiancoleman) for fixing those annoying typos in the docs! :-)

