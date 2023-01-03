+++
title = "Are they ever going to make ObjectDataSource worthwhile?"
date = 2006-03-03T13:44:00.000-06:00
updated = 2006-03-10T20:12:56.850-06:00
draft = false
url = '/2006/03/are-they-ever-going-to-make.html'
tags = []
+++

The current "stateless" design of `ObjectDataSource` when using the `DataObjectTypeName` is not viable for use in modern Ajax-driven applications. Since the `ObjectDataSource` creates a new object on every postback, there is no way to persist between call-backs any changes made by other previous postbacks without committing the infomation to the datastore.

Previously proposed "solutions" like not using `DataObjectTypes` are unacceptible due to the coupling of the view to the needed Insert/Update/Delete methods.

Microsoft should allow `ObjectDataSource` to have "acquire" symantics when handling postbacks, so that it can get the object instance being manipulated from the session store (whereever that is). Then `ObjectDataSource`/`GridView`/etc. can set the just properties bound on that specific view.

The easiest implementation would be to call an _Acquire_ method (parameterized with the declare key values) and use that object. This would be an overridable method that could in the base implementation simply do the current behavior of an ex-nilo call to the `DataObjectType`'s default constructor. In derived classes, we can offer whatever symantics we need (including cache, session, ORM, or web-service based object acquisition).

Of course we could do this ourselves using a derived `ObjectDataSource` and `ObjectDataSourceView`, except they didn't do the implementation of that correctly in the framework. `ObjectDataSource` never calls the derived-classes `GetView(String)` method. If they simply fixed that so that `ObjectDataSource`'s `GetView()` method called `GetView(String.Empty)`, I could do it myself.

If you agree, feel free to vote on [MSDN Product Feedback Center](http://lab.msdn.microsoft.com/productfeedback/viewfeedback.aspx?feedbackid=903a430d-d671-4e3e-9967-678c193fa349)

---

### Comments

#### You're right! I vote for it too. I also spent seve…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-03-09T03:30:00.000-06:00">Mar 4, 2006</time>

You're right! I vote for it too. I also spent several hours trying to override ODS's GetView method (and access that private field \_view using reflection)! Horrible!

---

#### Take a look at this to extend the ObjectDataSource…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2006-03-10T20:12:00.000-06:00">Mar 5, 2006</time>

Take a look at this to [extend the ObjectDataSource](http://www.manuelabadia.com/blog/PermaLink,guid,32e83915-a503-403e-97c7-e20dcf2e0b7e.aspx)

---
