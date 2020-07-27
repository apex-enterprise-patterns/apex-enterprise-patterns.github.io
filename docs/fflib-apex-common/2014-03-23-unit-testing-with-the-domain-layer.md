---
title: Unit Testing with the Domain Layer
parent: Apex Commons
nav_order: 3
---
Writing true Apex [unit tests](http://en.wikipedia.org/wiki/Unit_testing) that are quite granular can be hard in Apex, especially when the application gets more complex, as their is limited mocking support, meaning you have to create all your test data and move it through stages in its life cycle (by calling other methods) to get to the logic your unit test needs to test. Which of course is in itself a valid test approach of definitely needed, this blog is certainly not aiming to detract from those. But these are more of an integration or functional test.&nbsp;[Wikipedia](http://en.wikipedia.org/wiki/Unit_testing) has this to say about unit tests...

_In&nbsp;[computer programming](http://en.wikipedia.org/wiki/Computer_programming "Computer programming"),&nbsp; **unit testing** &nbsp;is a method by which individual units of&nbsp;[source code](http://en.wikipedia.org/wiki/Source_code "Source code"), sets of one or more computer program modules together with associated control data, usage procedures, and operating procedures are tested to determine if they are fit for use.<sup id="cite_ref-kolawa_1-0"><a href="http://en.wikipedia.org/wiki/Unit_testing#cite_note-kolawa-1">[1]</a></sup>&nbsp;Intuitively, one can view a unit as the smallest testable part of an application. In&nbsp;[procedural programming](http://en.wikipedia.org/wiki/Procedural_programming "Procedural programming"), a unit could be an entire module, but it is more commonly an individual function or procedure. In&nbsp;[object-oriented programming](http://en.wikipedia.org/wiki/Object-oriented_programming "Object-oriented programming"), a unit is often an entire interface, such as a class, but could be an individual method.<sup id="cite_ref-2"><a href="http://en.wikipedia.org/wiki/Unit_testing#cite_note-2">[2]</a></sup>&nbsp;Unit tests are short code fragments<sup id="cite_ref-3"><a href="http://en.wikipedia.org/wiki/Unit_testing#cite_note-3">[3]</a></sup>&nbsp;created by programmers or occasionally by&nbsp;[white box testers](http://en.wikipedia.org/wiki/White-box_testing "White-box testing")&nbsp;during the development process._

If as a developer you want to write these kind of low level unit tests around your [domain classes](https://wiki.developerforce.com/page/Apex_Enterprise_Patterns_-_Domain_Layer)&nbsp;(part of the [Apex Enterprise Patterns](https://github.com/financialforcedev/fflib-apex-common)), perhaps trying out a broader number of data input scenarios to your validation or defaulting code its hard using the conventional DML approach approach for the same reasons.&nbsp;Typically the result is you compromise on the tests you really want to write. The other downside is eventually your test class starts to take longer and longer to execute, as the DML overhead is expensive. Thus most developers I’ve spoken to, including myself, start to comment out test methods to speed things up while they work on a specific test method. Or perhaps you are a TDD fan, where incremental dev and unit test writing is important.

[Stephen Willcock](https://twitter.com/stephenwillcock)&nbsp;and I often discuss this balance,&nbsp;he is a big fan of DML’less testing and structuring code for testability, [having presented a few times at Dreamforce on the topic](https://www.youtube.com/watch?v=dWertK6Legc). There is a framework thats been in the&nbsp;[fflib\_SObjectDomain](https://github.com/financialforcedev/fflib-apex-common/blob/master/fflib/src/classes/fflib_SObjectDomain.cls) base class for a while now that presents its own take on this in respect to **Domain layer unit testing**. This allows you to emulate DML statements and use the same base class trigger handler method to invoke your Domain methods in the correct sequence from tests. While also performing more granular assertions without actually doing any DML.

As most of you who are using the patterns know the **Apex Trigger** code is tiny, one line…

```java
fflib_SObjectDomain.triggerHandler(Opportunities.class);  
```

The **mocking approach to DML statements** &nbsp;used by the Domain layer here leverages this line of code directly in your tests (emulating the Apex Trigger invocation directly) without having to execute DML or setup any dependent database state information the record might not need for the given test scenario. But first you must setup your mock database with the records you want to test against.

```java
fflib_SObjectDomain.Test.Database.onInsert(  
  new Opportunity[] { new Opportunity ( Name = 'Test', Type = 'Existing Account' ) } );  
```

The next thing you’ll want to do is use a slightly different convention when setting errors on your records or fields, this allows for you to assert what errors have been raised. This convention also improves from the less than ideal convention of **try/catch** and doing a **.contains('The driods i'm looking for!')** in the exception text for the message you are looking for.&nbsp;So instead of doing this on your **onValidate** …

```java
opp.AccountId.addError(  
 'You must provide an Account for Opportunities for existing Customers.');  
```

You use the **error** method, this registers in a list of errors that can be asserted later in the test. Like the **ApexPages.getMessages()** method, it is request scope so contains all errors raised on all domain objects executed during the test incrementally.

```java
opp.AccountId.addError(  
 error('You must provide an Account for Opportunities for existing Customers.',  
   opp, Opportunity.AccountId) );  
```

The full test then looks like this…

```java
@IsTest  
private static void testInsertValidationFailedWithoutDML()  
{  
 // Insert data into mock database  
 Opportunity opp = new Opportunity ( Name = 'Test', Type = 'Existing Account' );  
 fflib_SObjectDomain.Test.Database.onInsert(new Opportunity[] { opp } );

 // Invoke Trigger handler and thus appropriate domain methods  
 fflib_SObjectDomain.triggerHandler(Opportunities.class);

 // Assert results  
 System.assertEquals(1,  
 fflib_SObjectDomain.Errors.getAll().size());  
 System.assertEquals('You must provide an Account for Opportunities for existing Customers.',  
 fflib_SObjectDomain.Errors.getAll()[0].message);  
 System.assertEquals(Opportunity.AccountId,  
  ((fflib_SObjectDomain.FieldError)fflib_SObjectDomain.Errors.getAll()[0]).field);  
}  
```

**NOTE:** &nbsp;The above error assertion approach still works when your using the classic DML and Apex Trigger approach to invoking your Domain class handlers, just make sure to utilise the **error** method convention as described above.

There are also methods on the **fflib_SObjectDomain.Test.Database** to emulate other DML operations such as **onUpdate** and **onDelete**. As I said at the start of this blog, this is really not ment as an alternative to your normal testing, but might help you get a little more granular on your testing, allowing for more diverse use cases and not to mention speeding up the test execution!

