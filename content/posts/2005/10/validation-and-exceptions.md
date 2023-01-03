+++
title = "Validation and exceptions."
date = 2005-10-17T10:30:00.000-05:00
updated = 2007-11-19T00:20:22.807-06:00
draft = false
url = '/2005/10/validation-and-exceptions.html'
tags = [".Net","exception","best practice"]
+++

I was reading a blog entry (that I can't seem to find) [here](http://codebetter.com/blogs/brendan.tompkins/archive/2005/10/14/133152.aspx) about exception strategies with regard to web service calls.

I'll repeat the most important things first:
Only throw exceptions under **exceptional** circumstances!
Do **NOT** use exceptions for flow-control".

Okay, on to the real question, how to handle validation logic. What I usually do for business objects is to have a `ArrayList Validate(bool throwError)` method that accepts a boolean flag to determine if exceptions should be thrown _(hold on there, Nelly! I'll explain)_ and returns a list of business rule violations. What I do is have client or service code call Validate passing `false` for the `throwError` parameter. In this case, the Validate method returns an ArrayList of enumerated values of business rules. Since they are enumerated values, you can easily `switch` on the enumeration returned and act accordingly. Additionally the value can very easily be localized by using a resource lookup to derive the error message given to the user while still logging the exact error.

Now, if someone calls through to the `Save` method even in the presence of violations (or because they didn't call `Validate`), then I want to throw an exception, so I internally call `Validate` inside the `Save` method and pass `true` for the `throwError` parameter. When that error is thrown, the `Exception.Data` is first filled with all the enumerated business rule violations. This strategy insures that validation messages are available when desired via the `Validate` method, and that attempts to save bad data are stopped via an exception no matter what business object is being saved and no matter what the call depth. Lastly, the reuse of the `Validate` method and using an enumeration to encode the business rules allows code to handle problems and error messages to easily be localized. If anyone is interested in some code examples, tag up this post. Here's a quick sample:

```csharp
enum BusinessRules
{
   UserNameRequired
   , UserNameMinimumLength
}

class BusinessObject
{
   public string UserName;

   public ArrayList Validate(bool throwError)
   {
      ArrayList errors = new ArrayList();

      if (this.UserName == null  this.UserName.Length == 0)
      {
         errors.Add(BusinessRules.UserNameRequired);
      }

      if (this.UserName.Length < ex =" new" errors =" businessObject.Validate(false);"> 0)
      {
      StringBuilder statusMessage = new StringBuilder();

      foreach (BusinessRule error in errors)
      {
         if (error == BusinessRules.UserNameMinimumLength)
         {
            // add stuff to make it long enough as an example of handling the error
            businessObject.UserName += " terse little bugger";
         }
         else
         {
            statusMessage.AppendFormat(GetResource(error), this.UserName);
         }
      }

      view.SaveButton.Enabled = false;
   }
   else
   {
      statusMessage = "Looking good!";
      view.SaveButton.Enabled = false;
   }

   UpdateStatus(statusMessage);
}

```

---

### Comments

#### I juist don't get why a lot of people are trying t…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2005-11-28T18:10:00.000-06:00">Nov 1, 2005</time>

I just don't get why a lot of people are trying to make a lot of justifications to exempt them form something.I agree with what you have said that exepmtions must be allowed based on circumstances that are deemed as worthy.

---
