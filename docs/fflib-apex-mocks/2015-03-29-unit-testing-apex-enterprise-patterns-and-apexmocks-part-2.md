---
title: Unit Testing, Apex Enterprise Patterns and ApexMocks – Part 2
parent: Apex Mocks
nav_order: 2
---
In&nbsp;[Part 1](http://andyinthecloud.com/2015/03/22/unit-testing-with-apex-enterprise-patterns-and-apexmocks-part-1/)&nbsp;of this blog series i introduced a new means&nbsp;of applying true unit testing to Apex&nbsp;code&nbsp;leveraging the&nbsp;[Apex Enterprise Patterns](https://github.com/financialforcedev/fflib-apex-common). Covering the differences between true unit testing vs integration testing and how the lines can get a little blurred when writing Apex test methods.

If your following along&nbsp;you should be all set to start writing [true unit tests](http://en.wikipedia.org/wiki/Unit_testing) against your **controller** , **service** and **domain** classes.&nbsp;Leveraging the inbuilt dependency injection framework provided by the **Application** class introduced in the last blog. By injecting mock implementations of service, domain, selector and unit of work classes accordingly.

**What are Mock classes and why do i need them?**

Depending on the type of&nbsp; **class your unit testing** &nbsp;you'll need to mock different dependencies&nbsp;so that you don't have to worry about the data setup of those classes while your busy putting your hard work in to testing your specific class.

![Unit Testing]({{ site.baseurl }}/assets/images/unit-testing.png)

_In [object-oriented programming](http://en.wikipedia.org/wiki/Object-oriented_programming "Object-oriented programming"), **mock objects** are simulated objects that mimic the behavior of real objects in controlled ways. A programmer typically creates a mock object to test the behavior of some other object, in much the same way that a car designer uses a [crash test dummy](http://en.wikipedia.org/wiki/Crash_test_dummy "Crash test dummy") to [simulate](http://en.wikipedia.org/wiki/Simulation "Simulation") the dynamic behavior of a human in vehicle impacts. [Wikipedia](http://en.wikipedia.org/wiki/Mock_object)._

In this blog we are going to focus on an example **unit test method for a Service** , which requires that we **mock** the **unit of work** , **selector** and **domain** classes it depends on (unit tests for these classes will of course be written as well). Lets take a look first at the overall test method then break it down bit by bit.&nbsp;The following test method makes no SOQL queries or DML to accomplish its goal of testing the service layer method.

```java
 @IsTest  
 private static void callingServiceShouldCallSelectorApplyDiscountInDomainAndCommit()  
 {  
 // Create mocks  
 fflib_ApexMocks mocks = new fflib_ApexMocks();  
 fflib_ISObjectUnitOfWork uowMock = new fflib_SObjectMocks.SObjectUnitOfWork(mocks);  
 IOpportunities domainMock = new Mocks.Opportunities(mocks);  
 IOpportunitiesSelector selectorMock = new Mocks.OpportunitiesSelector(mocks);

 // Given  
 mocks.startStubbing();  
 List<Opportunity> testOppsList = new List<Opportunity> {  
 new Opportunity(  
 Id = fflib_IDGenerator.generate(Opportunity.SObjectType),  
 Name = 'Test Opportunity',  
 StageName = 'Open',  
 Amount = 1000,  
 CloseDate = System.today()) };  
 Set<Id> testOppsSet = new Map<Id, Opportunity>(testOppsList).keySet();  
 mocks.when(domainMock.sObjectType()).thenReturn(Opportunity.SObjectType);  
 mocks.when(selectorMock.sObjectType()).thenReturn(Opportunity.SObjectType);  
 mocks.when(selectorMock.selectByIdWithProducts(testOppsSet)).thenReturn(testOppsList);  
 mocks.stopStubbing();  
 Decimal discountPercent = 10;  
 Application.UnitOfWork.setMock(uowMock);  
 Application.Domain.setMock(domainMock);  
 Application.Selector.setMock(selectorMock);

 // When  
 OpportunitiesService.applyDiscounts(testOppsSet, discountPercent);

 // Then  
 ((IOpportunitiesSelector)  
 mocks.verify(selectorMock)).selectByIdWithProducts(testOppsSet);  
 ((IOpportunities)  
 mocks.verify(domainMock)).applyDiscount(discountPercent, uowMock);  
 ((fflib\_ISObjectUnitOfWork)  
 mocks.verify(uowMock, 1)).commitWork();  
 }  
```

First of all, you'll notice the test method name is a little longer than you might be used to, also the general layout&nbsp;of the test splits code into **Given** , **When** and **Then** blocks. These conventions help add some documentation, readability and consistency to test methods, as well as helping you focus on what it is your testing and assuming to happen. The convention is one defined by [Martin Fowler](http://martinfowler.com/), you can read more about [GivenWhenThen here](http://martinfowler.com/bliki/GivenWhenThen.html). The test method name itself, stems from a desire to express the behaviour the test is confirming.

**Generating and using Mock Classes**

**UPDATE:** Since the Apex Stub API was released you do not need this, see [here](https://www.salesforce.com/video/297000/)!

The Java based&nbsp;Mockito framework leverages the Java runtimes capability to dynamically create mock implementations. However&nbsp;the&nbsp;Apex runtime does not have any support for this. Instead ApexMocks uses [source code generation](http://en.wikipedia.org/wiki/Automatic_programming#Source_code_generation) to generate the mock classes it requires based on the interfaces you defined in my earlier post.

The patterns library also comes with its own **mock implementation of the Unit of Work** for you to use, as well as some base mock classes for your selectors and domain mocks (made know to the tool below). The following code at the top of the test method creates the necessary mock instances that will be configured and injected into the execution.

```java
// Create mocks  
fflib_ApexMocks mocks = new fflib_ApexMocks();  
fflib_ISObjectUnitOfWork uowMock = new fflib_SObjectMocks.SObjectUnitOfWork(mocks);  
IOpportunities domainMock = new Mocks.Opportunities(mocks);  
IOpportunitiesSelector selectorMock = new Mocks.OpportunitiesSelector(mocks);  
```

To generate the **Mocks** class used above use&nbsp;the&nbsp; **ApexMocks Generator** , you can run it via the [Ant tool](http://ant.apache.org/). The **apex-mocks-generator-3.1.2.jar** file can be downloaded from the ApexMocks repo [here](https://github.com/financialforcedev/fflib-apex-mocks).

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project name="Apex Commons Sample Application" default="generate.mocks" basedir=".">

<target name="generate.mocks">  
 <java classname="com.financialforce.apexmocks.ApexMockGenerator"/>  
 <classpath>  
 <pathelement location="${basedir}/bin/apex-mocks-generator-3.1.2.jar"/>  
 </classpath>  
 <arg value="${basedir}/fflib-sample-code/src/classes"/>  
 <arg value="${basedir}/interfacemocks.properties"/>  
 <arg value="Mocks"/>  
 <arg value="${basedir}/fflib-sample-code/src/classes"/>  
 </java>  
 </target>

</project>  
```

You can configure the output of the tool using a properties file (you can find more information [here](https://code4cloud.wordpress.com/2014/06/27/new-improved-apex-mocks-generator/)).

```java
IOpportunities=Opportunities:fflib_SObjectMocks.SObjectDomain  
IOpportunitiesSelector=OpportunitiesSelector:fflib_SObjectMocks.SObjectSelector  
IOpportunitiesService=OpportunitiesService  
```

The generated mock classes are contained&nbsp;as inner classes in the Mocks.cls class and also implement the interfaces you define, just as the real classes do. You can choose to add the above Ant tool call into your build scripts&nbsp;or just simply retain the class in your org refreshing it by re-run the tool whenever your interfaces change.

```java
/* Generated by apex-mocks-generator version 3.1.2 */  
@isTest  
public class Mocks  
{  
 public class OpportunitiesService  
 implements IOpportunitiesService  
 {  
 // Mock implementations of the interface methods...  
 }

public class OpportunitiesSelector extends fflib_SObjectMocks.SObjectSelector  
 implements IOpportunitiesSelector  
 {  
 // Mock implementations of the interface methods...  
 }

public class Opportunities extends fflib_SObjectMocks.SObjectDomain  
 implements IOpportunities  
 {  
 // Mock implementations of the interface methods...  
 }  
}  
```

**Mocking method responses**

Mock classes are dumb by default, so of course you cannot inject them into the upcoming code execution and expect them to work. You have to&nbsp;tell them how to respond when called. They will however record for you when their methods have been called for you to check or assert later. Using the framework you can tell a mock method what to return or exceptions to throw when the class your testing calls it.

So in effect you can teach them to **emulate** their real counter parts.&nbsp;For example when a Service method calls a Selector method it can return some in memory records as apposed to having to have them setup on the database. Or when the unit of work is used it will record method invocations as apposed to writing to the database.

Here is an example of configuring&nbsp;a **Selector mock method** to return test record data. Note that you also need to inform the Selector mock what type of SObject it relates to, this is also the case when mocking the Domain layer. Finally be sure to call startStubbing and stopStubbing between your mock configuration code. You can read much more about the [ApexMocks API here](https://github.com/financialforcedev/fflib-apex-mocks/blob/master/README.md), which resembles&nbsp;the Java Mockito API as well.

```java
// Given  
mocks.startStubbing();  
List<Opportunity> testOppsList = new List<Opportunity> {  
 new Opportunity(  
 Id = fflib_IDGenerator.generate(Opportunity.SObjectType),  
  Name = 'Test Opportunity',  
  StageName = 'Open',  
  Amount = 1000,  
  CloseDate = System.today()) };  
 Set<Id> testOppsSet = new Map<Id, Opportunity>(testOppsList).keySet();  
 mocks.when(domainMock.sObjectType()).thenReturn(Opportunity.SObjectType);  
 mocks.when(selectorMock.sObjectType()).thenReturn(Opportunity.SObjectType);  
 mocks.when(selectorMock.selectByIdWithProducts(testOppsSet)).thenReturn(testOppsList);  
 mocks.stopStubbing();  
```

**TIP:** If you want to [mock sub-select queries](http://andyinthecloud.com/2014/12/03/mocking-soql-sub-select-query-results/) returned from a selector take a look at this.

**Injecting your mock implementations**

Finally before you call the method your wanting to test, ensure you have injected the mock implementations. So that the calls to the **Application** class factory methods will return your mock instances over the real implementations.

```java
Application.UnitOfWork.setMock(uowMock);  
Application.Domain.setMock(domainMock);  
Application.Selector.setMock(selectorMock);  
```

**Testing your method and asserting the results**

Calling your method to test is a straight forward as you would expect. If it returns values or modifies parameters you can assert those values. However the ApexMocks framework also allows you to add further behavioural assertions that add further confidence the code your testing is working the way it should. In this case we are wanting to assert or **verify** (to using mocking speak) the correct information was passed onto the domain and selector classes.

```java
// When  
OpportunitiesService.applyDiscounts(testOppsSet, discountPercent);

// Then  
((IOpportunitiesSelector)  
 mocks.verify(selectorMock)).selectByIdWithProducts(testOppsSet);  
((IOpportunities)  
 mocks.verify(domainMock)).applyDiscount(discountPercent, uowMock);  
((fflib_ISObjectUnitOfWork)  
 mocks.verify(uowMock, 1)).commitWork();  
```

**TIP:** &nbsp;You can verify method calls have been made and also how many times. For example checking a method is only called a specific number of times can help add some level of performance and optimisation checking into your tests.

**Summary**

The full API for ApecMocks is outside the scope of this blog series, and frankly [Paul Hardaker](https://twitter.com/comic96) and [Jessie Altman](http://jessealtman.com/) have done a much better job, take a look at the [full list of documentation links here](https://github.com/financialforcedev/fflib-apex-mocks/blob/master/README.md#documentation). Finally keep in mind my comments at the start of this series, this is not to be seen as a total alternative to traditional Apex test method writing. Merely another option to consider when your wanting a more focused means to test specific methods in more varied ways without incurring the development and execution costs of having to setup all of your applications data in each test method.

