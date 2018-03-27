---
title: "DevOps and Management"
date: 2018-03-25T12:29:54-05:00
publishDate: 2018-03-28T08:00:00-05:00
draft: false
tags:
  - DevOps
  - Deployment
---

TL;DR: What do you do if your management doesn't "get" DevOps and Continuous Integration/Continuous Deployment?<!--more-->

## The Longer Version ##

I recently gave a talk at [NEW CodeCamp](http://newcodecamp.com) on a topic that over the last several months has become something I enjoy talking about and working with - Octopus Deploy. After the session, I was talking with someone who attended about their own deployment process. I've thought about our conversation, and I've decided to write my thoughts on the situation.

Unfortunately, his story is likely not all that unusual. In short, their update process consists mostly of dropping only the assemblies or files that were modified into their production environment, rather than the full software package. In the case of websites, it would be HTML or ASPX files, and in the case of a windows application it would be the supporting binary DLLs (the primary executable would rarely change, understandably).

From the perspective of the company management, this can be a very attractive method of doing things: if you have multiple bug fixes in the works, it's easy to deploy the fix in any order once it has been tested (assuming they change different files), and the process appears to be relatively low-risk. It also allows their testing pipeline to be filled relatively efficiently while giving the impression of a real CI/CD pipeline to production.

## What if we... ##

One idea that we came up with to address this situation was to package each individual file as its own package, and then deploy only those files that change. On the surface, this seems like a reasonable solution - in fact, in many ways as developers we do this as part of normal development. Think of what is actually happening when you update NuGet/NPM packages, Ruby Gems, etc. The difference is that these dependencies are wrapped into and bundled with your application and deployed at the same time as you generally deploy a new version of your application, not independently upgraded. And there tends to be a very good reason for that.

I always seem to come back to the concept of emergent bugs - bugs that occur because of two independent changes that, on their own, operate correctly but when combined produce unexpected results. These changes can sometimes be found through various automated testing techniques, but other times it takes end-to-end testing. If you treat each change only in isolation, there is no way of identifying these kinds of interactions until it goes to production. And once there, it can be more difficult to track down the issue, since you need to determine the state of each individual component to determine when the bug arises.

Then there's also the practicality of maintaining such a system. Within the context of Octopus (and other systems that operate similarly), you would have to specifically require each package that would need to be deployed; the process to deploy your product would be a massive number of steps (at least one for each file), and every package would have their own distinct version number, causing a headache trying to figure out just what version of which file should go at any given time. This isn't even touching on the maintenance when new files inevitably need to be added to the application.

## But why? ##

Finally, I have to take a step back and really ask about the elephant in the room: Why? Really, doing this type of system doesn't actually give any additional flexibility or benefit over traditional Continuous Deployment systems. In fact, in some circumstances it actually gives you _less_ flexibility.

One perspective I always tend to have when it comes to deploying software is on the ability to scale out. In this type of scenario, if you wanted to deploy your software to a new location, you have to do more work proportional to the number of locations to deploy to. You now have to remember to update these files in multiple locations; in the case of a new deployment you have to deploy your "base" image (which could possibly, and I expect likely, be a few months old), then figure out which files need to be changed from there. In either case, you now have a considerable potential for having mismatched versions in your own production environment, which can make debugging new issues even more difficult.

Compare that to how modern CD systems work: everything goes, every time. A new location to deploy to is effectively no different than updating an existing one, assuming the environments are the same. If the concern is really being able to have multiple development (and testing) efforts going at the same time, it can accomplished very easily by using a modern version control system like Git. You can even use an established process like GitFlow or GitHubFlow, but isn't really necessary if you don't want to. In either case, each effort can be done in a different branch and when it's tested/ready to go/whatever, it gets merged into the primary branch, built, and deployed. You don't even need to be on a regular release cadence (like once a month, every week, etc), unless you want to be.

## Personal Lessons ##

If there's one thing I've learned while thinking about this situation and the possible solutions, it's that CI/CD best practices are best practices for reason. When management doesn't fully understand the _why_ of a process, it can be difficult to move to a better place. But the nice thing about the current state of technology is that doing a proper deployment pipeline doesn't mean you have to sacrifice your agility of getting features and bug fixes out the door.

When I think about these best practices, I always come back to is something I learned early in my career. At my employer, we would have auditors come in on a yearly basis and examine the IT department (software engineering included) to help identify potential risks and concerns. During this audit, they would look at things like who had access to mission-critical servers, examine how access to those servers was regulated, and even checked that the various procedures were being followed.

When it came to software development, they pulled a random sample of code changes (normally driven by commit messages in source control), and looked at a couple things around each one - namely, why the change was made (who requested it, why was it needed, etc), documentation around testing and/or sign off of the change and _proof that the code that was tested and approved was the same code that ended up in the production environment_.

That last point has always stuck with me, and I think that when it comes to things like CI/CD, it's something that should always be baked into the system - even if you aren't being audited on a regular basis. As software becomes more pronounced to our every day lives, being able to tie a specific change to a given version (and vice versa) becomes important as well, whether you're writing the next generation of AI software for self-driving cars or a game for people to enjoy on the bus on their way to work.

Unfortunately, in the scenario above, it is far from the case. Sure, you can compare the version (assuming it changed) or a hash of each DLL, and hopefully tie that back to a build system and ensure they match. But if you have any kind of complexity in your project, that would be a long and tedious task. It would be much easier to say, across the board, "This change was incorporated into this package version, and this package version was put at production at this time", which should be easier to verify. Future packages can be proven to have been built on top of that version, so you can still be confident those changes are in some way still there.

## Fine. Now what? ##

Unfortunately, I don't think there is a technical solution to this problem. Nothing we can do will suddenly make things more correct, or necessarily even easier, at least not for the long term. The best option is probably to sit down with management, and have a conversation around what they see as the benefits to this process, and try to correct their misconceptions. Once you have them on the same page regarding the current state of their deployment process, then you can begin to make changes - such as a new branching model that accommodates their desire to have multiple development and testing efforts simultaneously. From there you can begin to automate your processes and not only take advantage of the scalability that provides, but also have more faith in your processes and know that things will be done reliably and consistently.