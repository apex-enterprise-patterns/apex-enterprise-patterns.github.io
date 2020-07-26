---
title: Apex Enterprise Patterns - Separation of Concerns
parent: Apex Commons
nav_order: 1
---
Software is often referred to as living thing that changes and evolves over time. As with complex&nbsp;organisms&nbsp;that are expected to endure and evolve over time, it is important to understand what role each part of the organism plays. To keep the organism growing while still remaining strong and support its evolution or even rapid growth through in take of more resources. And so the same is true for complex applications, often&nbsp;categorised&nbsp;as Enterprise level, reflecting the complexity and scale of importance to the 'eco-system' the application lives within. So what can we learn from this when engineering such applications?

### **Separation of Concerns**

Complex code gets out of hand when you don't partition it properly and it becomes heavily intermixed. Making it hard to work from, error prone and worst still hard to learn, only serving to worsen the problem as you bring new developers into the party! Avoiding this to some degree at the basic level is often&nbsp;referred&nbsp;to as 'code re-use'. Which is of course a very good thing as well! The act of creating modules or libraries to share common calculations or processes amongst different parts of your&nbsp;application.

#### Is SOC just a posh word for 'code re-use' then?

If your considering SOC properly, your doing some upfront thinking about the internal plumbing of your application (down to class naming conventions and coding guidelines) that you feel will endure and hopefully be somewhat self describing to others going forward. Rather than the usual adhoc approach to code re-use which sees&nbsp;fragments&nbsp;of code get moved around once two or more areas start to need it. Often just placed in MyUtil classes or some other generic dumping area. Which is fine, and certainly recommended vs copy and paste!

#### So what are the&nbsp;benefits&nbsp;of SOC?

At a high level applications have storage, logic and one or more means to interact with them, be that by humans and/or other applications. Once you outline these, you can start to define layers within your application each with its own set of concerns and responsibilities to the application and other layers. Careful consideration and management of such layers is important to adopting SOC.

- **Evolution.** Over time, as technology, understandings and requirements (both functional and technical) evolve, any one of these layers may need to be significantly extended, reworked or even dropped. Just look at UI technology over the last 10 years as a prime example of this.
- **Impact Management.** Performing&nbsp;work or even dropping one or more of these layers &nbsp;should not&nbsp;unduly&nbsp;impact (if at all) the others, unless the requirements of course lead to that.
- **Roles and Responsibility.** Each layer has its own responsibility and should not drop below or over extend that responsibility. For example dropping one client technology / library in favour of another, should not mean loosing business logic, as this is the responsibility of another layer.&nbsp;If the lines of responsibility get blurred it erodes the purpose and value of SOC.

The typical SOC layers are shown below. On the Force.com platform, there are two distinct approaches to development, both can be used standalone or to complement each together. The Declarative and traditional Coding style approaches, broadly speaking, fit into the standard SOC layers like so...

![]({{ site.baseurl }}/assets/images/screen-shot-2012-11-16-at-14-45-14.png "Screen Shot 2012-11-16 at 14.45.14")

### **Why consider SOC on Force.com?**

One of the key&nbsp;benefits&nbsp;of Force.com, is its declarative development model, the ability to create objects, fields, layouts, validation rules, workflows, formula fields etc without a single line of code. So these for sure should be your first port of call, typically if your app is heavily data centric in its use cases, you'll get by with delivering a large&nbsp;portion&nbsp;of your application this way! So while not code, what you can achieve with&nbsp;declarative&nbsp;development is still very much an architecture layer in your application, one that I will talk about more later.

So if your app is process centric and/or getting pushed to implement more complex calculations, validations or richer UI experiences. You'll be dipping into the land of Apex and thus code.&nbsp;Out of all the logic you invest developer and testing hours in, it is the business logic that you should be most concerned about protecting.&nbsp;Force.com provides many places to place Apex code, Triggers, VF Controllers, Web Services, REST Services, Batch Apex, Email Handlers, the list goes on. We will start to explore some rules for&nbsp;defining&nbsp;what goes into these in the further parts of this series. For now consider the following...

1. If you had to replace or add another UI to your app (e.g. Mobile) how much of that code would you be rewriting / porting that is&nbsp;effectively&nbsp;nothing to do with the UI's responsibility? But more to do with inserting, updating, validating and calculating in your app.
2. If you wanted to provide a public facing API (such as REST) to your logic how would you do this? What parts of your existing code base would you call to implement the API? Action methods in your VF controllers, passing in bits of view state?!?!
3. If you got asked to scale your application logic via Batch Apex as well continue to provide an interactive experience (for smaller volumes) via your existing UI (VF controllers). How would you share logic between the two to ensure the user gets consistant results regardless.
4. Visualforce provides a means for you to partition your code via [MVC](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) (a form of SOC for client development), but simply using Visualforce and Controllers does not&nbsp;guarantee you have implemented it. &nbsp;For example how complex are your action methods in your controllers? Do you have any code does more than just deal with handling content to and from the user?
5. How easily can new developers find their way around your code base? How much time would they need to learn where to put new code and where to find existing behaviour?

Depending on how you have or plan to partition / placed your code you may already be in good shape to tackle some of the above. If not or just curious, hopefully the upcoming articles will help shed a bit of light for further thought.

### **Summary**

This blog entry is the first describing Enterprise Application Architecture patterns, particularly focused on applying them on the Force.com platform. If you attended Dreamforce 2012 this year, you may have caught a presentation on this topic. In the meantime, if you missed the session and fancy a preview of the patterns that help support SOC. You can find the Github repo [here](https://github.com/financialforcedev/df12-apex-enterprise-patterns) along with slides and a recent re-run recording I did. For now, please feel free to comment, make suggestions and requests for topics / use cases you would like to be covered and I'll do my best to include and answer them!

#### LINKS

Here are a few links to other resources, the DF12 session and discussions of late that relate to this post. These will give you some foresight into upcoming topics I'll be discussing here. Enjoy!

- [Separation of Concerns (Wikipedia)](http://en.wikipedia.org/wiki/Separation_of_concerns)
- [Any documentation on writing&nbsp;reusable&nbsp;code?](http://salesforce.stackexchange.com/questions/4423/any-documentation-on-writing-reusable-code)
- [Martin Fowlers description of some of the upcoming patterns I'll be discussing in respect to Force.com.](http://martinfowler.com/eaaCatalog/)
- [Slides from my Dreamforce 2012 presentation](http://www.slideshare.net/afawcett/df12-applying-enterprise-application-design-patterns-on-forcecom?ref=http://andrewfawcett.wordpress.com/)
- [Video recording of a re-run of the session I did at Dreamforce 2012.](https://docs.google.com/open?id=0B6brfGow3cD8UHhzWDF1WENEaXc)
