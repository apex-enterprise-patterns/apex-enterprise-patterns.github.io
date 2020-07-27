---
title: Unit Testing, Apex Enterprise Patterns and ApexMocks - Part 1
parent: Apex Mocks
nav_order: 1
---
As [Paul Hardaker](https://twitter.com/comic96) ([ApexMocks](https://github.com/financialforcedev/fflib-apex-mocks/blob/master/README.md) author) once pointed out, technically the reality is Apex developers often only end up writing only integration tests.

Lets review Wikipedia's definition of [unit tests](http://en.wikipedia.org/wiki/Unit_testing)...

_Intuitively, one can view a unit as the **smallest testable** part of an application. In procedural programming, a unit could be an entire module, but it is more commonly an **individual function or procedure**. In object-oriented programming, a unit is often an entire **interface, such as a class** , but could be **an individual method**. Unit tests are **short code fragments** created by programmers or occasionally by white box testers during the development process_

Does this describe an **Apex Test** you have written recently?

Lets review what Apex tests typically require us to perform...

- **Setup of application data** for every test method
- Executes the **more code than we care about testing** at the time
- Tests often **not very varied enough** , as they can take a long time to run!

Does the following Wikipedia snippet describing [integration tests](http://en.wikipedia.org/wiki/Integration_testing) more&nbsp;accurately describe this?

_ **Integration testing** (sometimes called integration and testing, abbreviated I&T) is the phase in software testing in which **individual software modules are combined and tested as a group**. It occurs **after unit testing** and before validation testing. Integration testing takes as its input modules that have been unit tested, groups them in larger aggregates, **applies tests defined in an integration test plan** to those aggregates, and delivers as its **output the integrated system ready for system testing** _

The challenge with writing true unit tests in Apex can also leave those wishing to follow practices like&nbsp;[TDD](http://en.wikipedia.org/wiki/Test-driven_development) struggling due to the&nbsp;lack of [dependency injection](http://jessealtman.com/2014/03/dependency-injection-in-apex/) and mocking support in the Apex runtime. We start to desire&nbsp;mocking support such as what we find for example in Java's&nbsp;[Mockito](https://code.google.com/p/mockito/)&nbsp;(the inspiration behind ApexMocks).

The lines between unit vs integration testing and which&nbsp;we should use and when can&nbsp;get blurred since Force.com does need Apex tests to invoke&nbsp;Apex Triggers for coverage (requiring actual test integration with the database) and if your using Workflows a lot you may want the behaviour of these reflected in your tests. So one&nbsp;cannot completely move away from writing integration tests of course. But is there a better way for us to regain some of the benefits other platforms enjoy in this area for the times we feel it would benefit us?

**Problems writing Unit Tests for complex code bases...**

![Integration Testing]({{ site.baseurl }}/assets/images/integration-testing.png)


The problem is a true unit tests aim to test a small unit of the code, typically&nbsp;a specific method. However if this method ends up querying&nbsp;the database we need to have inserted those records prior to calling the method and then assert the records afterwards.&nbsp;If your familiar with Apex Enterprise Patterns, you'll recognise the following separation&nbsp;of concerns in this diagram which shows clearly what code might be executed in&nbsp;a controller test for example.

For complex applications this approach per test can&nbsp;be come quite an overhead&nbsp;before you even get to call your controller method and assert the results! Lets face it, as we have to wait longer and longer for such tests, this inhibits our desire to write further more complex tests that may more thoroughly test the code with different data combinations and use cases.

**What if we could emulate the database layer somehow?**

Well those of you familiar with [Apex Enterprise Patterns](https://github.com/financialforcedev/fflib-apex-common/blob/master/README.md#application-enterprise-patterns-on-forcecom) will know its big on separation of concerns. Thus aspects such as querying the database and updating it are encapsulated away in so called Selectors and the Unit Of Work. Just prior to Dreamforce 2014, the patterns introduced the [**Application class**](http://andyinthecloud.com/2014/08/26/preview-of-advanced-apex-enterprise-patterns-session/), this provides a single application wide means to access the **Service** , **Domain** , **Selector** and **Unit Of Work** implementations&nbsp;as apposed to directly instantiating&nbsp;them.

In this two part blog series, we are focusing on the role of the **Application** class and its **setMock** methods. These methods, modelled after the platforms **Test.setMock** method (for mocking HTTP comms), provide a means to mock the core architectural layers of an application which is based on the Apex Enterprise Patterns. By allowing mocking in these areas, we can see that we can write unit tests that focus only on the behaviour of the controller, service or domain class we&nbsp;are testing.

![Unit Testing]({{ site.baseurl }}/assets/images/unit-testing.png)

**Preparing your Service, Domain and Selector classes for mocking**

[Apex Interfaces](https://www.salesforce.com/us/developer/docs/apexcode/Content/apex_classes_interfaces.htm) are key to implementing mocking. You must define these in order to allow the mocking framework to substitute dynamically different implementations. The patterns library also provides base interfaces that reflect the base class methods for the Selector and Domain layers. The [sample application contains a full example of these interfaces](https://github.com/financialforcedev/fflib-apex-common-samplecode) and how they are applied.

```java

// Service layer interface

public interface IOpportunitiesService  
{  
  void applyDiscounts(Set<ID> opportunityIds, Decimal discountPercentage);
  Set<Id> createInvoices(Set<ID> opportunityIds, Decimal discountPercentage);
  Id submitInvoicingJob();  
}

// Domain layer interface

public interface IOpportunities extends fflib_ISObjectDomain  
{  
 void applyDiscount(Decimal discountPercentage, fflib_ISObjectUnitOfWork uow);  
}

// Selector layer interface

public interface IOpportunitiesSelector extends fflib_ISObjectSelector  
{  
 List<Opportunity> selectByIdWithProducts(Set<ID> idSet);  
}  
```

First up apply the Domain class interfaces as follows...

```java

// Implementing Domain layer interface

public class Opportunities extends fflib\_SObjectDomain  
 implements IOpportunities {

// Rest of the class  
}  
```

Next is the Service class, since the service layer remains stateless and global, i prefer to retain the static method style. Since you cannot apply interfaces to static methods, i use the following convention, though I've seen others with inner classes. First create a new class something like OpportunitiesServiceImpl, copy the implementation of the existing service into it and remove the static modifier from the method signatures before apply the interface. The original service class then becomes a stub for the service entry point.

```java

// Implementing Service layer interface

public class OpportunitiesServiceImpl implements IOpportunitiesService  
{  
 public void applyDiscounts(Set\<ID\> opportunityIds, Decimal discountPercentage)  
 {  
 // Rest of the method...  
 }

public Set\<Id\> createInvoices(Set\<ID\> opportunityIds, Decimal discountPercentage)  
 {  
 // Rest of the method...  
 }

public Id submitInvoicingJob()  
 {  
 // Rest of the method...  
 }  
}
```

```java
// Service layer stub
global with sharing class OpportunitiesService  
{  
 global static void applyDiscounts(Set\<ID\> opportunityIds, Decimal discountPercentage)  
 {  
  service().applyDiscounts(opportunityIds, discountPercentage);  
 }

 global static Set\<Id\> createInvoices(Set\<ID\> opportunityIds, Decimal discountPercentage)  
 {  
  return service().createInvoices(opportunityIds, discountPercentage);  
 }

 global static Id submitInvoicingJob()  
 {  
  return service().submitInvoicingJob();  
 }

 private static IOpportunitiesService service()  
 {  
  return new OpportunitiesServiceImpl();  
 }  
}  
```

Finally the Selector class, like the Domain class is a simple matter of applying the interface.

```java
public class OpportunitiesSelector extends fflib\_SObjectSelector  
 implements IOpportunitiesSelector  
{  
 // Rest of the class  
}  
```

**Implementing Application.cls**

Once you have defined and implemented your interfaces you need to ensure there is a means to switch at runtime the different implementations of them, between the real implementation and a the mock implementation as required within a test context. To do this a factory pattern is applied for calling logic to obtain the appropriate instance. Define the Application class as follows, using the factory classes provided in the library. Also note that the Unit Of Work is defined here in a single maintainable place.

```java
public class Application  
{  
 // Configure and create the UnitOfWorkFactory for this Application  
 public static final fflib_Application.UnitOfWorkFactory UnitOfWork =  
  new fflib_Application.UnitOfWorkFactory(  
  new List<SObjectType> {  
    Invoice__c.SObjectType,  
    InvoiceLine__c.SObjectType,  
    Opportunity.SObjectType,  
    Product2.SObjectType,  
    PricebookEntry.SObjectType,  
    OpportunityLineItem.SObjectType });

  // Configure and create the ServiceFactory for this Application  
  public static final fflib\_Application.ServiceFactory Service =  
    new fflib_Application.ServiceFactory(  
    new Map<Type, Type> {  
      IOpportunitiesService.class => OpportunitiesServiceImpl.class,  
      IInvoicingService.class => InvoicingServiceImpl.class });

  // Configure and create the SelectorFactory for this Application  
  public static final fflib\_Application.SelectorFactory Selector =  
    new fflib_Application.SelectorFactory(  
    new Map<SObjectType, Type> {  
    Opportunity.SObjectType => OpportunitiesSelector.class,  
    OpportunityLineItem.SObjectType => OpportunityLineItemsSelector.class,  
    PricebookEntry.SObjectType => PricebookEntriesSelector.class,  
    Pricebook2.SObjectType => PricebooksSelector.class,  
    Product2.SObjectType => ProductsSelector.class,  
    User.sObjectType => UsersSelector.class });

// Configure and create the DomainFactory for this Application  
 public static final fflib_Application.DomainFactory Domain =  
 new fflib_Application.DomainFactory(  
    Application.Selector,  
    new Map<SObjectType, Type> {  
      Opportunity.SObjectType => Opportunities.Constructor.class,  
      OpportunityLineItem.SObjectType => OpportunityLineItems.Constructor.class,  
      Account.SObjectType => Accounts.Constructor.class,  
      DeveloperWorkItem__c.SObjectType => DeveloperWorkItems.class });  
}  
```

**Using Application.cls**

If your adapting an existing code base, be sure to leverage the Application class factory methods in your application code, seek out code which is explicitly instantiating the classes of your Domain, Selector and Unit Of Work usage. Note you don't need to worry about Service class references, since this is now just a stub entry point.

The following code shows how to wrap the Application factory methods using convenience methods that can help avoid repeated casting to the interfaces, it's up to you if you adopt these or not, the effect is the same regardless. Though the modification the service method shown above is required.

```java

// Service class Application factory usage
global with sharing class OpportunitiesService  
{  
 private static IOpportunitiesService service()  
 {  
  return (IOpportunitiesService) Application.Service.newInstance(IOpportunitiesService.class);  
 }  
}

// Domain class Application factory helper
public class Opportunities extends fflib_SObjectDomain  
 implements IOpportunities  
{  
 public static IOpportunities newInstance(List<Opportunity> sObjectList)  
 {  
  return (IOpportunities) Application.Domain.newInstance(sObjectList);  
 }  
}

// Selector class Application factory helper
public with sharing class OpportunitiesSelector extends fflib_SObjectSelector  
 implements IOpportunitiesSelector  
{  
 public static IOpportunitiesSelector newInstance()  
 {  
  return (IOpportunitiesSelector) Application.Selector.newInstance(Opportunity.SObjectType);  
 }  
}  
```

With these methods in place reference them and those on the Application class as shown in the following example.

```java
public class OpportunitiesServiceImpl  
 implements IOpportunitiesService  
{  
 public void applyDiscounts(Set<ID> opportunityIds, Decimal discountPercentage)  
 {  
  // Create unit of work to capture work and commit it under one transaction  
  fflib_ISObjectUnitOfWork uow = Application.UnitOfWork.newInstance();

  // Query Opportunities  
  List<Opportunity> oppRecords =  
  OpportunitiesSelector.newInstance().selectByIdWithProducts(opportunityIds);

  // Apply discount via Opportunties domain class behaviour  
  IOpportunities opps = Opportunities.newInstance(oppRecords);  
  opps.applyDiscount(discountPercentage, uow);

  // Commit updates to opportunities  
  uow.commitWork();  
 }  
}  
```

The Selector factory does carry some useful generic helpers, these will internally utilise the Selector classes as defined on the Application class definition above.

```java
List<Opportunity> opps =  
 (List<Opportunity>) Application.Selector.selectById(myOppIds);
List<Account> accts =  
 (List<Account>) Application.Selector.selectByRelationship(opps, Account.OpportunityId);
```

**Summary and Part Two**

In this blog we've looked at how to defined and apply interfaces between your service, domain, selector and unit of work dependencies. Using a factory pattern through the indirection of the **Application class** we have implemented an **&nbsp;injection framework** within the definition of these enterprise application&nbsp;separation&nbsp;of concerns.

I've seen dependency injection done via&nbsp;constructor injection, my personal preference is to use the approach shown in this blog. My motivation for this lies with the fact that these pattern layers are well enough known throughout the application code base and the Application class supports other facilities&nbsp;such as polymorphic instantiation of domain classes and helper methods as shown above on the Selector factory.

In the second part of this series we will look at how to write true unit tests for your controller, service and domain classes, leveraging the amazing [ApexMocks library](https://github.com/financialforcedev/fflib-apex-mocks/blob/master/README.md). If in the meantime you wan to get a glimpse of what this might look like take a wonder through the Apex Enterprise Patterns sample application tests [here](https://github.com/apex-enterprise-patterns/fflib-apex-common-samplecode/blob/master/sfdx-source/apex-common-samplecode/test/classes/service/OpportunitiesServiceTest.cls#L38) and [here](https://github.com/apex-enterprise-patterns/fflib-apex-common-samplecode/blob/master/sfdx-source/apex-common-samplecode/test/classes/controllers/OpportunityApplyDiscountControllerTest.cls#L30).

```java
// Provide a mock instance of a Unit of Work  
Application.UnitOfWork.setMock(uowMock);

// Provide a mock instance of a Domain class  
Application.Domain.setMock(domainMock);

// Provide a mock instance of a Selector class  
Application.Selector.setMock(selectorMock);

// Provide a mock instance of a Service class  
Application.Service.setMock(IOpportunitiesService.class, mockService);
```

