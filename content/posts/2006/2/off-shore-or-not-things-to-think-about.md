+++
title = "Off-shore or not? Things to think about..."
date = 2006-02-06T14:04:00.000-06:00
updated = 2006-02-06T14:14:58.263-06:00
draft = false
url = '/2006/02/off-shore-or-not-things-to-think-about.html'
tags = []
+++

I just returned from a wonderful 6-day cruise, and it got me thinking about being at sea, and that reminded me that I gave some thoughts to a friend about doing offshore development projects a few months back. Here is the brain-dump based on my real-life experiences in three projects and environments.

If I were a project manager that was just about to take on my first offshore project:

### What would you tell me are the critical success factors?

* Clear understanding of the working process (e.g. Test First or CMMI or whatever)
* Strong version control system and procedure (Subversion for remote access at minimum)
* Strong control of the development environment. You don’t want the far-removed developers working on systems containing things you many not want to include. I suggest you use VirtualPC machines and/or RDP to development images.
* Clear specifications, with a formal process for answering questions about design.
* Understanding that the off-shore developers will rotate through your assignments, so don’t expect to train someone “your way” to be a long term investment. Rather, interview for like-minded developers up front.
* Always allocate at least one senior developer to answer questions and review code. Do NOT expect the code to have an quality unless you personally inspect for it.
* Daily teleconference with the developers, preferably at their start-of-day. This is a remoted version of the daily stand-up meeting to establish progress and delegate answering of questions.

### What were big mistakes that you made?

What which you would warn future PMs?

* Assumed that we would have the same team throughout; that simply doesn’t happen.
* Failed to inspect code early on, once daily inspection was in-place, progress-rate and quality both went up.
* Not detailed enough on the functional specification. If you have a internal developer worthy and willing, have them stub the unit tests before the offshore team begins the coding.

### What were the primary friction points?

What elements of the process that slowed you down, frustrated you, and/or took you off course?

* The time-overlap is a good thing as long as you are not waiting on the other team. You can do reviews of the previous “night’s work” during your work day and leave `TODO`s for the team when you go home, but without daily reporting in (the stand-up), it is very easy to lose days of work.
* In the case of India, there were many holidays, so we often were surprised by someone not being available for conference calls.
* Much of the management communications when dealing with large firms centers on making sure that they are not to blame for flaws and delays. You have to push back very hard to make sure that the managers understand that a successful project outcome is the ONLY target. I found that I had much better luck working with the developers without the firm’s managers in the loop.
* Many times, a disagreement about how something worked degraded into a “you told us to do this, so we did it exactly as stated”, without taking language differences and domain-specific knowledge into consideration, this needs to be controlled early to avoid poisoning the team with bickering.

### How detailed did you make your designs and requirements?

* The better the specifications in the “how” realm, the better. You don’t need to give them more than a screen mockup for the UI, but every single interaction should be detailed. We found that activity diagrams and state diagram to be very useful.
* A data dictionary of terms and Domain Specific Languages is an absolute requirement. If you are interfacing to existing databases and systems, you need to have both a system-level description of the fields (in the parlance of the system) and the business use of those fields (in the parlance of the end-user of the system). Most specifications will use both terms, so a cross reference is absolutely required.
* Don’t insult your partner; don’t worry about minutiae like pixel position and exact colors. Firstly because it stifles their creativity, and secondly those things change frequently, and each would be a change request that you will be billed for.
* Each specification should be given a version number and any changes to the specification should be tracked and agreed to by both parties. Formal version control of the specifications is just as important as the source code. I suggest Team System for this.

### What process did you have in place for code review?

Was it sufficient?

* Every day, first thing when I arrive, I get the current tips and do a mass compare. I used Beyond Compare (easily one of the coolest tools I’ve ever used, and cheap) and compared the entire directory tree. Doing it daily insured there was never much to review at one time.
* All database changes should be scripted and checked into source control, and thus become part of the review process.
* All changes should be “eight hour chunks” or less, so that any work can be checked in every night. This insures regular progress and won’t overwhelm the review process. Any longer-running changes should be broken down into smaller chunks.
* Every defect found in review becomes a defect in the tracking system, even “style” issues. This insures they get completed, and are considered part of the work-product.
* In general, reporting review issues is enough; but sometimes the on-shore senior developer will have to make framework-level changes or fix a defect in order to serve as an instructional example of the corrections needed elsewhere.
* If you have a promotion scheme setup, you can label things for incorporation to the public-facing builds as the review determines they are good.

### How was the development environment set up?

* Several configurations on different projects. For some, a Citrix Metaframe (RDP/Terminal Server) setup was used. This insured complete control of the development environment, which is key to insuring the build process has everything setup the same as the development process. It did incur a lot of overhead and delays getting the environments setup and changed, but that was specific to the company.
* Another strategy, which I vastly prefer is to use a VirtualPC or VMWare virtual machine which is pre-setup with the development environment. That image can be distributed on a DVD or CD (depending on size), and then all development takes place on the local machine. Incidentally, I do this now for all my development and actually RDP into the appropriate virtual machine as this gives a very responsive and repeatable environment.

### What have you found to be the best way to track the progress of the projects? 

How have you kept a handle on your progress in relationship to the project plan?

* I think classic burn down rates and velocity are great measures of progress that are easily understood. I recommend using TargetProcess http://www.targetprocess.com/ or Team Foundation to track the state of everything.

### Have you found an optimal frequency of build?

* Every check-in for a build.
* Every day for a developer deployment build 
* A least weekly for QA review.

### Have you found an optimal frequency of code reviews?

* Daily. **No discussion.**
