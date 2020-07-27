---
title: Pillars of Enterprise Development
parent: Apex Commons
nav_order: 8
---

Enterprise&nbsp;organisations&nbsp;have complex processes and integration requirements that typically span multiple locations around the world. They&nbsp;seek out the best in class applications that support their needs today and in the future. The ability to adapt an application to their practices, terminology, and integrations with other existing applications or processes is key to them. They invest as much in your application as they do in you as the vendor capable of delivering an application strategy that will grow with them.

### **Motivation and background to the book Lightning Platform Enterprise Architecture**

The Salesforce community is diverse, consisting of package&nbsp;developers, in house developers and consultants, each with varying degrees of technical knowledge. While Salesforce documentation can be found to address the needs of each of these types of developers, it is often more of a reference style in nature and can be hard to contextualise. Meaning a clear path to an architecture for enterprise developers to refer can be hard to find and peace together.

I wanted the book to act as flow&nbsp;for all that a developer needs to get the best out of the platform while laying down a strong foundation of development practices and patterns, to allow their application to scale and evolve with the same rapid pace of the platform itself (see some examples of where this has been achieved below). &nbsp;Existing enterprise Java or .Net developers considering the platform will find some well known enterprise patterns. There has been a significant increase in architecture and best practice questions over the last 2-3 years on the Salesforce forums such as [StackExchange](http://salesforce.stackexchange.com/) and [Salesforce Community Answers](https://success.salesforce.com/).

[![]({{ site.baseurl }}/assets/images/book.png)](https://www.packtpub.com/programming/lightning-platform-enterprise-architecture-third-edition){:target="_blank"}

### **Pillars of Enterprise Development**

When planning the outline for the book and thinking about Enterprise development on the platform in general, i had three core beliefs in mind that i wanted to seed within the book.

- **Embrace the whole Platform** , The first tenant of the Force.com platform is to combine the power of the declarative programming style with the traditional source code based programming style. Doing so ensures not only the developer is as effective as possible, focusing on coding only where needed, but also the resulting application has strong ‘native’ feel to it, giving its end users access to the platforms rich set of customization, configuration and integration capabilities that enterprise customers demand. I wanted the reader to&nbsp;have a key awareness of the benefits of being ‘native’ on the platform, to keep it in mind always and realize the benefits this combined development approach can be bring to end users.

- **Build Strong Foundations** , Enterprise applications are expected to serve their customers for many years to come, as customers build solutions and processes around them, which become a critical part of their businesses. As applications grow in complexity the code base especially needs to not buckle under the pressure, be that the addition features or general maintenance of existing features, made by existing or new developer resources. A strong foundation will ensure that the code base endures this type of change with minimal impact on the rest of the system and the users. I wanted the&nbsp;reader to get&nbsp;a strong sense of the meaning of [Separation of Concerns](https://developer.salesforce.com/page/Apex_Enterprise_Patterns_-_Separation_of_Concerns)&nbsp;and how it applies to enterprise applications built on the Force.com platform.

  - Lightning Experience was obviously not around at the time and Lightning Components only in Pilot, now they are both GA, but are&nbsp;[Services](https://developer.salesforce.com/page/Apex_Enterprise_Patterns_-_Service_Layer) still as applicable to&nbsp;Lighting controllers? You bet!

- **Your Application as Platform** , Enterprise customers demand high levels of integration, customization and integration from your application, as they either repurpose or consume the application within a larger business process or integration. So i wanted the reader to gain an understanding of the Force.com platform features they can leverage to ensure that these aspects are considered from an architecture perspective and thus baked into each new application function, such that the resulting application becomes in essence a part of the platform itself.

  - Lightning Process Builder and Flow was but a gleam in someones eye before these patterns arrived on the scene, but&nbsp;can&nbsp;[Services](https://developer.salesforce.com/page/Apex_Enterprise_Patterns_-_Service_Layer) be exposed via [Invocable Methods](http://andyinthecloud.com/category/invocable-methods/) to this tool? You bet!

