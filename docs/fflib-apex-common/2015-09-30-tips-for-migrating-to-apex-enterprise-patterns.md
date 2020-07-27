---
title: Tips for Migrating to Apex Enterprise Patterns
parent: Apex Commons
nav_order: 7
---
One of the common questions that gets asked is, "_How do i adopt&nbsp;[Apex Enterprise Patterns](https://github.com/financialforcedev/fflib-apex-common) within an existing code base?_".

Often its also folks who have not owned the code base from the start and are inheriting it. Your also not likely to be&nbsp;blessed with a empty cheque of time to sort the code out before your asked to start adding new features or fixing issues. So how do you&nbsp;find time to crack open an old code base and start introducing [Separation&nbsp;of Concerns](http://wiki.developerforce.com/page/Apex_Enterprise_Patterns_-_Separation_of_Concerns)?

**Tip 1. Pick and choose and go incrementally!** You can pick and choose what layer you want to work on first and incrementally split out further. For example Service layer implementations don't have to use Unit of Work, Domain or Selector initially. Typically I find the Service layer the most value add to get in place first. Going incrementally over a few release iterations perhaps, across the tips below can help you and your team walk before you start to run with the patterns and gain confidence and support across the team in the approach.

**Tip 2. Get a basic Service layer going**. Even if you don't adopt any other aspect, try to look for opportunities to move code from controllers, batch classes etc into a service layer. Remember&nbsp;that technically it is essentially a class ending in Service with some static methods in it, its the meaning and responsibility of it thats important here. If you don't have time to follow all the conventions such as bulkification don't worry, by moving it a Service you've already made a big step! For sure however, don't apply global to your service until your happy with it's methods.

**Tip 3. Sweeping out inlined SOQL.** Again if your not ready to use the base classes in the library here, just start naming classes with the Selector suffix and re-home your code performing SOQL into methods on that class. You won't get all the benefits of consistency, but you will start getting those from reuse and encapsulation. That said the Selector pattern does not really need you to have other things setup so do consider extending the base class where you can, doing for some Selectors and not others is also fine if some are a bigger challenge.

**Tip 4. Moving your Trigger code into Domain class.** The Domain class does support all the Apex Trigger events, and has several overrides to invoke code that applies to multiple events. So you should be able to find a home for your trigger code amongst the various overrides given by the base class. If you have got some recursive stuff going on you'll want to look at the Statefull configuration in the Domain class.

You may want to start with this one over tips 2 and 3 if your code is heavily Apex Trigger driven, it often the best way to show other developers the value as well, as the code starts to become more OO and more factored for typically little risk so long as you have good Apex tests in place. Finally developers familiar with so called Wrapper Patterns elsewhere in the community should warm to this pattern more quickly.

**Tip 5. Adding Application Factory and Mocking support Incrementally.** This is possibly easier to add in areas of the solution&nbsp;where most of the patterns are starting to fall into place so don't rush into adding this to soon. Separation&nbsp;of Concerns is key to making mocking work. So don't try to introduce this to soon if you've not got things factored out well enough. That said, you may find adding support just for mocking the Service layer gives you some initial boosts in writing tests around controllers and batches classes and the like. The Unit of Work does not need to be fully configured either, its fine to have only have a subset of your objects representative of the areas its used in.
