---
title: New Features - July 2015
parent: Apex Commons
nav_order: 101
---
This short post calls out some features added by the community.

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

Thank you one and all for being social and contributing to a growing community around this library!

