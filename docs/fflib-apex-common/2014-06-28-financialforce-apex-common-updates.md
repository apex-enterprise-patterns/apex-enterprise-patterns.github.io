---
title: FinancialForce Apex Common Updates
parent: Apex Commons
nav_order: 4
---
This blog starts a series of updates leading up to&nbsp;[Dreamforce 2014](http://www.salesforce.com/dreamforce/DF14/),&nbsp;where I am pleased to announce that my&nbsp; **Advanced Apex Enterprise Patterns** session has just been selected! In the coming series of blogs I&nbsp;will be covering enhancements&nbsp;to the existing base classes supporting the&nbsp; **Apex Enterprise Patterns** and highlighting some of the more general classes from within the&nbsp;[FinancialForce Apex&nbsp;Common](https://github.com/financialforcedev/fflib-apex-common/blob/master/README.md)&nbsp;repository both&nbsp;reside in.

This time I will be highlighting some excellent contributions from fellow **Force.com MVP** &nbsp;[Chris Peterson](https://twitter.com/ca_peterson) who has recently added more general **utility&nbsp;classes** &nbsp;to support **security** and **dynamic queries.** As well as&nbsp;some improvements from myself to the existing&nbsp; **Domain** and **Selector** classes.

In a following blog I'll be going over in more detail the current&nbsp;results of some [experimental work](https://github.com/financialforcedev/fflib-apex-common/blob/fls-support-experiment/README.md#expirimental-crud-and-fls-support)&nbsp;I've been doing&nbsp;relating to **generic**  **Field Level Security** &nbsp; **enforcement** within the **patterns** base classes, meanwhile enjoy the new **fflib\_SecurityUtils** class..

# **New fflib\_SecurityUtils.cls**

![SecurityUtils]({{site.baseurl}}/assets/images/securityutils.png)

The **Salesforce Security Review** currently requires that both **object level** and **field level security** is enforced by your code. The later of which, field level security has recently stirred up quite a lot of [discussion and concern in the ISV community](https://success.salesforce.com/ideaView?id=08730000000GzMwAAK), more on this in the follow up blog post! In the meantime if you **read the two Salesforce articles** ([here](https://developer.salesforce.com/page/Enforcing_CRUD_and_FLS) and [here](https://developer.salesforce.com/page/Testing_CRUD_and_FLS_Enforcement)) describing the requirement and how to implement it in your **Apex code** ( **Visualforce** offers some built in support for you). If like me, you'll quickly realise the sample code provided is in reality somewhat **verbose** &nbsp;for anything more than a single object or field usage!

Enter the [fflib\_SecurityUtils](https://github.com/financialforcedev/fflib-apex-common/blob/master/fflib/src/classes/fflib_SecurityUtils.cls) class! As you can see from the UML representation shown here it's methods are pretty simple and thus easy to use, an **Apex exception** is thrown if the user does not have access to the given object and/or fields. Here are some examples of it in use, you can choose to check individually or in bulk a set of fields and the object, covering both **CRUD** and **Field Level Security**.

```java

fflib_SecurityUtils.checkObjectIsInsertable(Account.SObjectType);

fflib_SecurityUtils.checkInsert(  
 Account.SObjectType,  
 new List<Schema.SObjectField>{  
 Account.Name,  
 Account.ParentId, } );

fflib_SecurityUtils.checkUpdate(  
 Lead.SObjectType,  
 new List<Schema.SObjectField>{  
 Lead.LastName,  
 Lead.FirstName,  
 Lead.Company } );
```

This class will also help perform **object and field read access** checks before making **SOQL** queries, though you may want to check out the features of the **QueryFactory** class as it also leverages this utility class internally.

# **New fflib\_QueryFactory.cls**

![QueryFactory]({{site.baseurl}}/assets/images/queryfactory.png)

The key purpose of this class is to make building **dynamic SOQL queries safer and more robust** than traditional string&nbsp;concatenation&nbsp;or String.format approaches.&nbsp;It also has an option to automatically **check read security for the objects and fields** given to it.

If your using **Fieldsets** in your **Visualforce** pages you'll know that it's generally up to&nbsp;you to query these fields, as you can see from the UML diagram, the class supports methods that allow you to provide the **Fieldset name** and have it **automatically include those fields** for you in the resulting query.

Chris has done an amazing job with this class not only in its feature and function but in its API design, leveraging the [fluent](http://en.wikipedia.org/wiki/Fluent_interface) model to make coding with it **easy to use** but also **easy to read** and understand. He first presented in at the [FinancialForce DevTalks](http://www.meetup.com/FinancialForce-DevTalks/) event this month, his presentation can be found [here](https://speakerdeck.com/ca_peterson/whats-new-in-apexlib). In the presentation he gives some examples and discusses when you should use it and when not. So if your writing code like this currently...

```java
String soql = ‘SELECT ‘;  
for(Integer i = 0; i< fieldList.size(); i++){  
 soql += fieldList + ‘, ‘;  
}  
soql = soql.substring(0,soql.length()-2);  
soql += conditions != null && conditions.trim().size() > 0 ? ‘ WHERE ‘ +  
 soqlConditions : ‘’;  
soql += limit != null && limit.trim().size() > 0 ? ‘ LIMIT ‘+limit;  
List<Contact> nonBobs = Database.query(soql);  
```

In it's simplest form it's use looks like this..

```java
String soql =  
 new fflib\_QueryFactory(Contact.SObjectType)  
 .selectField(Contact.Name)  
 .setLimit(5)  
 .toSOQL();  
```

With **object and field read&nbsp;security enforcement** and **Fieldset** support would look like this...

```java
String soql =  
 new fflib\_QueryFactory(Contact.SObjectType)  
 .assertIsAccessible().  
 .setEnforceFLS(true).  
 .selectFields(myFields)  
 .selectFieldSet(Contact.fieldSets.ContactPageFieldSet)  
 .setCondition(‘Name != “Bob”’)  
 .toSOQL();  
```

The class is fully commented and the associated test class has further examples, also&nbsp;Chris is keen to work on API for the SOQL where clause in the future, i look forward to seeing it!

# **fflib\_SObjectSelector.cls (Selector Pattern) Updates**  

I have updated this base class used to support the [Selector pattern](https://developer.salesforce.com/page/Apex_Enterprise_Patterns_-_Selector_Layer), to leverage the **fflib\_QueryFactory** , in doing so you now have the option (as a constructor argument) to **enable Field Level Security** for fields selected by the selector. The default constructor and prior constructors are&nbsp;still supported, with the addition of the following that now allows you to control Fieldset, object and field level security enforcement&nbsp;respectively. For example...

```java
OpportunitiesSelector oppsSelector =  
 new OpportunitiesSelector(includeFieldSetFields, enforceObjectSecurity, enforceFLS);  
```

**NOTE:** You can of course implement your own Selector default constructor and enable/disable these features by default within that, if you find yourself constantly passing a certain combination of these configurations parameters.

For custom selector methods you can leverage a **QueryFactory constructed and initialised based on the Selector** information, leaving you to add the additional&nbsp; **where clause** for the particular query logic the method **encapsulates**. Because of this you **no longer need to add assertIsAccessible** as the first line of your custom Selector methods. Prior to adopting QueryFactory a custom selector method might have looked like this...

```java
public Database.QueryLocator queryLocatorReadyToInvoice()  
{  
 assertIsAccessible();  
 return Database.getQueryLocator(  
 String.format('SELECT {0} FROM {1} WHERE InvoicedStatus\_\_c = \'\'Ready\'\' ORDER BY {2}',  
 new List\<String\>{getFieldListString(),getSObjectName(),getOrderBy()}));  
}  
```

The updated&nbsp;patterns sample applications [OpportunitiesSelector](https://github.com/financialforcedev/fflib-apex-common-samplecode/blob/c68d55a46b35b955c8cb343580b7b500d977784c/fflib-sample-code/src/classes/OpportunitiesSelector.cls) example to use the new **newQueryFactory** base class method, because this method **pre-configures the factory** with the selector object, fields, order by and fieldsets (if enabled), the new implementation is **simplified and more focused** on the criteria of the query the method **encapsulates**.

```java
public Database.QueryLocator queryLocatorReadyToInvoice()  
{  
 return Database.getQueryLocator(  
 newQueryFactory().setCondition('InvoicedStatus\_\_c = \'\'Ready\'\'').toSOQL());  
}  
```

This&nbsp;updated&nbsp; **fflib\_SObjectSelector&nbsp;base class is backwards** compatible from the API perspective, so you can choose to continue with the original&nbsp; **String.format** approach or adopt the **newQueryFactory** method accordingly. You can further review the old example [here](https://github.com/financialforcedev/fflib-apex-common-samplecode/blob/c68d55a46b35b955c8cb343580b7b500d977784c/fflib-sample-code/src/classes/OpportunitiesSelector.cls), against the new one [here](https://github.com/financialforcedev/fflib-apex-common-samplecode/blob/master/fflib-sample-code/src/classes/OpportunitiesSelector.cls).

# fflib_SObjectDomain.cls (Domain Pattern) Updates

This base class has to date had minimal functionality in it other than the routing of trigger events to the applicable virtual methods and object security enforcement. To support better configuration of these features and those in the future, i have added a new **Domain class configuration** feature, accessed via a new **Configuration** property.

Despite the focus on enforcing security in the above new features and updates, there are times when you want to enforce&nbsp;this in the calling code and not globally. For this reason the base class can now be configured to disable the object security checking (by default&nbsp;performed during the trigger after event), **leaving it up to the calling code to enforce**. Methods accessed from the new&nbsp; **Configuration** property can be used to control this.

```java
public class ApplicationLogs extends fflib\_SObjectDomain  
{  
 public&nbsp;ApplicationLogs(List<ApplicationLog__c> records)  
 {  
 super(records);  
 Configuration.disableTriggerCRUDSecurity();  
 }  
}  
```

**Domain Trigger State**

I have also been asked to provide a means to maintain member variable **state** between invocation of the trigger handler methods. Currently if you define a **member variable** in your **Domain class** it is **reset** between the **before** and **after trigger phases**. This is due to the default trigger handler recreating your Domain class instance each time.

If you want to **retain** information or records queried in the before handler methods in your class member variables such that it can be reused in the after handler methods, you can now enable this feature using the following configuration. The following illustrates the behaviour.

```java
public class Opportunties extends fflib\_SObjectDomain  
{  
 public String someState;

public TestSObjectStatefulDomain(List\<Opportunity\> sObjectList)  
 {  
 super(sObjectList);  
 Configuration.enableTriggerState();  
 }

public override void onBeforeInsert()  
 {  
 System.assertEquals(null, someState);  
 someState = 'Something';  
 }

public override void onAfterInsert()  
 {  
 System.assertEquals('Something', someState);  
 }  
}  
```

This feature is also aware of **trigger recursion** , meaning if there is such a scenario a new Domain class instance is created (since the records it wraps are new).

