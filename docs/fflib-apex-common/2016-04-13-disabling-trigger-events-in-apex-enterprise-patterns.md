---
title: Disabling Trigger Events in Apex Enterprise Patterns
parent: Apex Commons
nav_order: 10
---
I'm proud to host my first **guest blogger**!&nbsp;**[Chris Mail](https://twitter.com/Autobat)** or **[Autobat](https://github.com/Autobat)** as he is known on GitHub. Take it away Chris....

**How to put the safety on...**

Being an **architect** in a **professional services organisation** is a funny game. Each project is either a shiny new Salesforce instance without a fingerprint on it or an unknown vault of code and configuration that we must navigate through.

I have been using the fflib pattern now for some time, and more of our teams are adopting it for our programs of work. My latest addition is something that an architect might wonder why we need; the ability to **turn off triggers** via a simple interface on all domains.

In an ever growing complex environment, perhaps multiple projects over time delivering iterative enhancements I was noticing a common piece of code being developed within the Domain layer. It looked something along the lines of this:

```java
public override void onAfterInsert()  
{  
 // if this is set we are already in a loop and want to exit!  
 if (bProhibitAfterInsertTrigger)  
 {  
  return;  
 }  
 // down here we do something, maybe insert an Account!  
}  
```

While small and inconspicuous it allowed our code base to become inconsistent as there was **no control over the exposure of these controlling flags** and worse, we were repeating ourselves in every domain!

The solution was simple, a **fluent style API** within **fflib\_SObjectDomain**. Any code can now simply set the control flags for any domain class:

```java
fflib_SObjectDomain.getTriggerEvent(YourDomain.class).disableAll(); // dont fire anything  
fflib_SObjectDomain.getTriggerEvent(YourDomain.class).disableAllBefore();  
fflib_SObjectDomain.getTriggerEvent(YourDomain.class).disableAllAfter();

fflib_SObjectDomain.getTriggerEvent(YourDomain.class).disableBeforeInsert();  
fflib_SObjectDomain.getTriggerEvent(YourDomain.class).disableBeforeUpdate();  
fflib_SObjectDomain.getTriggerEvent(YourDomain.class).disableBeforeDelete();

fflib_SObjectDomain.getTriggerEvent(YourDomain.class).disableAfterInsert();  
fflib_SObjectDomain.getTriggerEvent(YourDomain.class).disableAfterUpdate();  
fflib_SObjectDomain.getTriggerEvent(YourDomain.class).disableAfterDelete();  
fflib_SObjectDomain.getTriggerEvent(YourDomain.class).disableAfterUndelete();  
```

To enable, just call the inverse e.g. . **enableAfterInsert** (); etc.

While not every code base will need to use these flags, they allow you to control quickly and easily your trigger execution with a single line of code that all your development team can reuse and follow.

