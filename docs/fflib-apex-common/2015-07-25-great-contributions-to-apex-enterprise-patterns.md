---
title: Great Contributions to Apex Enterprise Patterns!
parent: Apex Commons
nav_order: 6
---
Just one week before the start of [Dreamforce 2015](http://www.salesforce.com/dreamforce/DF15/), the [Apex Enterprise Pattern](https://github.com/financialforcedev/fflib-apex-common) library will be **2 years old** since it was first published. My community time is increasingly busy these days not only answering questions around this and other GitHub repositories, but also reviewing [Pull Requests](https://help.github.com/articles/using-pull-requests/). Both are&nbsp;a sign of a healthy open source project&nbsp;for sure! In this short blog i just wanted to call out some of the new features added by the community. So lets get started...

- **Unit of Work, register Method flexibility.&nbsp;&nbsp;**  
The initial implementation of Unit Of Work used a List to contain the records registered with it. This ment that in some cases if the code path registered the same record twice an error would occur. This change means you don't have to worry about this and if complex code paths happen to register the same record again, it will be handled without error.Thanks to:&nbsp;[Thomas Fuda](https://github.com/tfuda) for this enhancement!

- **Unit of Work, customisable DML implementation via IDML interface.&nbsp;**  
This improvement allows you to implement the IDML interface to implement your own calls to the platforms DML methods. The use case that prompted this enhancement was to allow for fine grained control using the Database methods that permit options such as all or nothing control (the default being all).Thanks to:&nbsp;[David Esposito](https://github.com/daveespo) for this enhancement!

- **Unit of Work and Application Factory, new newInstance(Set\<SObjectType\> types) method**  
This enhancement provides the ability to leverage the factory but have it provide Unit of Work instances configured using a specific set of SObjectType's and not the default one. In cases where you have only a few objects to register, perhaps dynamically or those different from the default Application set for a specific use case. Please read the comments for this method for more details.Thanks to:&nbsp;[John Davis](https://github.com/jondavis9898) for this enhancement!

- **Unit of Work, Eventing API**  
New virtual methods have been added to the Unit of Work class, allowing those that want to subclass it, to hook special handling code to be executed during commitWork. This allows you to extend in very custom way your application or services own work to be done at various stages, start, end and during the DML operations. For example common validation that can only occur once everything has been written but not before the request ends.Thanks to:&nbsp;[John Davis](https://github.com/jondavis9898) for this enhancement!

- **Unit of Work, Bulkified Register Methods**  
Its now possible to register lists of SObject's with the Unit of Work in one method call rather than one per call. While the Unit of Work has always been internally bulkified, this enhancement helps callers who are dealing with lists or maps interact with the Unit Of Work more easily.Thanks to:&nbsp;[MayTheSForceBeWithYou](https://github.com/MayTheSForceBeWithYou)&nbsp;(now a FinancialForce employee) for this enhancement!

- **Selector, Better default Order By handling**  
Not all Standard objects have a Name field, this excellent enhancement helped ensure that if this was the case the base class would look for the next best thing to sequence your records against by default. Alex has also made numerous other tweaks to the Selector layer in addition to this btw!Thanks to:&nbsp;[Alex Tennant](https://github.com/adtennant) for this enhancement!

In addition to the above there has been numerous behind the scenes improvements to library as well, making it more stable, support various non standard aspects of standard objects and such like.&nbsp;I based the above on GitHub's report of commits to the repository [here](https://github.com/financialforcedev/fflib-apex-common/commits/master?page=1).

In addition to code changes, there are also some great discussions and ideas in the pipeline...

- **Enterprise Patterns 'Light'**  
This discussion from&nbsp;[john-m](https://github.com/john-m), was prompted by&nbsp;Force.com MVP&nbsp;[Matt Lacey](https://twitter.com/LaceySnr), who is [thinking about the footprint of the library](http://www.laceysnr.com/2015/03/the-enterprise-patterns-on-diet.html) and making it more accessible to smaller consulting based code bases. Join the conversation [here](https://github.com/financialforcedev/fflib-apex-common/issues/36)!

- **Unit of Work&nbsp;and Cyclic Dependencies**  
This limitation doesn't come up often, but for complex situations is causing fans of the UOW&nbsp;some pain having to leave it behind when it does. This is the long standing [request](https://github.com/financialforcedev/fflib-apex-common/issues/1) by [Seth Stone](https://github.com/sethstone), but has seen a few attempts since via [Alex Tennant](https://github.com/financialforcedev/fflib-apex-common/pull/23) and more recently some upcoming efforts by [john-m in the pipeline](https://github.com/financialforcedev/fflib-apex-common/issues/59). Watch this space!

- **Helper methods to fflib\_SObjectDomain for Changed Fields**  
Another Force.com MVP,&nbsp;[Daniel Hoechst](https://github.com/dhoechst), suggested this feature, inspired from those present in other trigger frameworks, join the conversation [here](https://github.com/financialforcedev/fflib-apex-common/issues/52).

- **Support for undelete Trigger events to Domain layer**  
This idea raised by our good friends over at [BigAssForce](https://appexchange.salesforce.com/listingDetail?listingId=a0N3000000B51gaEAB&tab=r), is seeing some recent interest for contribution by [Autobat](https://github.com/Autobat), see [here](https://github.com/financialforcedev/fflib-apex-common/issues/18) for more discussion.

Thank you one and all for being social and contributing to a growing community around this library!

