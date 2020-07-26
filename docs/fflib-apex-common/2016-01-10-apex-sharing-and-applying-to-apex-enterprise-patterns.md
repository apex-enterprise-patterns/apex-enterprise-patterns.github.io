---
title: Apex Sharing and applying to Apex Enterprise Patterns
parent: Apex Commons
nav_order: 9
---
**[Apex Sharing](https://developer.salesforce.com/docs/atlas.en-us.198.0.apexcode.meta/apexcode/apex_classes_keywords_sharing.htm)** can be a bit of mystery to new developers as well as seasoned ones from other platforms. This blog is not for those wanting to understand sharing as such, there are plenty of [excellent articles](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_bulk_sharing_understanding.htm)&nbsp;and [Salesforce docs](https://developer.salesforce.com/docs/atlas.en-us.198.0.apexcode.meta/apexcode/apex_classes_keywords_sharing.htm) on that. Here&nbsp;i wanted to talk about how first I came to understand it and how it fits into **[Apex Enterprise Patterns](https://github.com/financialforcedev/fflib-apex-common#application-enterprise-patterns-on-forcecom)**.

I recall one really basic thing that took me by surprise was the name, Sharing? Of course&nbsp;this is&nbsp;an end user based way of describing what as an engineer, I effectively understood&nbsp;as **row level security**. I was also blown away to know that this applied in and outside of code, for example when reporting is used, very cool! Row level security is certainly for me a more accurate way to describe it and certainly helps when i have been talking to others new to the platform but have experience on other platforms.

The second thing that i learn is that&nbsp;in order&nbsp;to control it, it is required&nbsp;to be&nbsp;considered&nbsp;in&nbsp;the way one&nbsp;**[annotates code at design time](https://developer.salesforce.com/docs/atlas.en-us.198.0.apexcode.meta/apexcode/apex_classes_keywords_sharing.htm)**. And&nbsp;less&nbsp;so about a&nbsp;default&nbsp;runtime or a configured at runtime&nbsp;context. Since&nbsp;sharing is not enabled by default in Apex (except for Anonymous Apex contexts), it needs to be **enabled via opt-in** by the developer. Salesforce helps remind us of this through tools like the [Salesforce Security Scanner](http://security.force.com/security/tools/forcecom/scanner) and [best practices here](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_security_sharing_rules.htm), well worth a read.

You may have noticed that the&nbsp; **Apex Enterprise Patterns** &nbsp;classes providing implementations of your&nbsp; **Service** layer always have **with sharing** specified. This sets the default context for **all code,** in the **&nbsp;Domain, Selector** &nbsp;or&nbsp;other classes that are&nbsp; **executed from then on to run in this mode**. Such classes do not need to and should not generally need to qualify any **with sharing** or **without sharing** keywords either.

```java
global with sharing class OpportunitiesService  
{  
 global static void applyDiscounts(\<ID\> opportunityIds, Decimal discountPercentage)  
 {  
 // This code and any it calls runs as 'with sharing'  
 }  
}  
```

So what happens if you really **want to run without sharing** ([great article here on reasons for this](https://developer.salesforce.com/page/Without_Sharing))? Do you apply it to your Domain or Selector class definition? Well actually neither, since not all the code in these classes may warrant sharing being disabled for example. What i prefer to do is keep the execution of running in this mode **as short and contained as possible** , to avoid any other inadvertent execution of other code running in this mode.

The basic approach is to leverage an **inner class** that contains just the code that needs to **run without sharing**. Typically this code would run in the **Selector** layer, though can be used elsewhere inside a service method implementation or domain class method. The point is its scoped to a method or specific execution path.

```java
public class OpportunitiesSelector extends fflib_SObjectSelector  
{  
  public List<Opportunity> selectById(Set<Id> idSet) {  
  // This method simply runs in the sharing context of the caller  
  // ...  
  return opportunities;  
  }

  public List\<OpportunityInfo\> selectOpportunityInfo(Set\<Id\> idSet) {  
    // Explicitly run the query in a 'without sharing' context  
    return new SelectOpportunityInfo().selectOpportunityInfo(this, idSet);  
  }

  private without sharing class SelectOpportunityInfo {  

    public List<OpportunitiesSelector.OpportunityInfo> selectOpportunityInfo(OpportunitiesSelector selector, Set<Id> idSet) {  
      // Execute the query as normal  
      // ...  
      return opportunityInfos;  
    }  
  }  
}  
```

So do we still need it specify **with sharing** elsewhere? Well yes, controllers for sure is still good practice, indeed Selectors can end up being called from these. I personally also consider any class that is invoked as an Apex entry point, such as **Invocable Methods** , **Batch Apex** , **Scheduled Apex** etc in this category.

If your following a **service orientated design** most of these entry points delegate to the Service layer, so it feels like your doubling up at times, but thats no bad thing when security is concerned. Finally keep in mind, if you choose to expose your Service layer as an **API** , it feels equally important to ensure the default sharing mode is enabled&nbsp;regardless of what mode the caller is running in.

The general approach here, is enable sharing, then make the code, developer and business/solution analyst justify why it needs to be switched off for a system level operation that requires it. If you put aside the Apex Enterprise Patterns, this is in fact not that different from the **general guideline of having with sharing on all your controllers** , the main difference here is by putting it on your service layer, your ensuring **not just your controller entry points are covered**.

