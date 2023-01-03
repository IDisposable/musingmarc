+++
title = "Editor inheritance is the root of all evil."
date = 2005-01-10T16:30:00.002-06:00
updated = 2008-04-15T12:31:38.095-05:00
draft = false
url = '/2005/01/editor-inheritance-is-root-of-all-evil.html'
tags = [".Net","best practice"]
+++

Say you've got some code that works. Say you get a new task that is very similar to the task the exist code performs. If you are a rank beginner, you don't notice the similarity and write (hopefully) the same code other than the requirement differences. It works, but your time was largely wasted because you wrote from scratch, making many of the same mistakes and anguishing over many of the same design decisions. You've also added maintenance costs in the form of two chunks of possibly very dissimilar code. If you want to advance beyond this stage, you have to **know the code**, and you have learn the code by reading other peoples' code. If it's good code, you learn good practices as a bonus, but in any case, you will know what's out there to reuse. If you are slightly more mature in your practices, you remember or find that old code and you decide to adapt the existing code. You duplicate the code and tweak it for the new needs. It works, but now any change you make _might_ have to be made in more than one place, and you have to remember to check in both places. If you are even more mature, you recognize that copying the block of code is bad, extract it out to a helper method and parameterize it. This is good because you can now look in **one** place for later modifications, and the parameterization makes it clear how the usages differ. If you are even more mature, you could at that point decide to take some time to factor out any other implied parameterizations that could aid in the reuse of this code. This is a good exercise, but remember not to introduce features that are not needed in the near future. Anything you write should have a clear business need, or you are just introducing more code to maintain that is never going to be used. Rather, keep an open mind to how you can make the method more general without adding code or reducing clarity. Sure, you know all this stuff, and what does that have to do with today's subject? This rant is brought to you by the following code snippet:

```csharp
if ( ! recordId.Equals(string.Empty) && editMode.Equals("I") ) {
    // do stuff.
}
```

Which was dutifully copied from about 1000 other places in some code I was reviewing today. Sure, it looks like okay code (depending on your style preferences), and where it was copied from _it was_. But not now! This time, instead of testing some property that is a string, it's testing an integer. Guess how often that first half of the condition is false \[you saw the not, right?\]? This is editor inheritance gone bad. Take some good code, copy it to some other place and don't account for the variations that should have been obvious while you were grabbing it. How could we have done this right? As a coder, why didn't we write a trivial helper method with proper overloads for the expected data types. Nothing fancy needed:

```csharp
static bool IsSupplied(string mightBeMissing) {
     return false == mightBeMissing.Equals(string.Empty);
}
static bool IsSupplied(int mightBeZero) {
    return 0 != mightBeZero;
}
```

Then using IsSupplied everywhere would at least be guaranteed to check the right thing. And if we decide to change the representation of a missing value, there's only a couple of places to update. As a senior developer I'm sure you would probably never have made this sort of mistake. What I'm preaching to **you** is that we owe it to those that follow in our coding footsteps to have **good, best-practice code**. When the next newbie comes along and does a copy/paste of our code, wouldn't it be better that **your** code is a clean example of what code should look like? You owe it to the people that follow in your footsteps!
