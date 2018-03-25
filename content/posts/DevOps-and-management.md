---
title: "DevOps and Management"
date: 2018-03-25T12:29:54-05:00
draft: true
---

I recently gave a talk at [NEW CodeCamp](http://newcodecamp.com) on a topic that over the last several months has become something I enjoy talking about and working with - Octopus Deploy. After the session, I was talking with someone who attended about their own deployment process. After thinking about his situation, I've decided to write my thoughts on the situation.

The situation he described is not likely all that unusual. He basically told me that their deployment of updates mainly consisted of dropping only the modified assemblies or files into their production environment. In the case of websites, it would be HTML or ASPX files, and in the case of a windows application it would be the supporting binary DLLs (the primary executable would rarely change, understandably).

From the perspective of the company management, this is a very attractive method of doing things: if you have multiple bug fixes in the works, it's easy to deploy the fix in any order once it has been tested (assuming they change different files), and the process appears to be relatively low-risk. It also allows their testing pipeline to be filled relatively efficiently while giving the impression of a real CI/CD pipeline to production.

With that all set up, let's go over what some potential solutions _could_ be, and how they work out from a CI/CD best practice perspective.

## Deploy each file as its own package

## Conclusion
If there's one thing I've learned while thinking about this situation and the possible solutions, it's that CI/CD best practices are best practices for reason.

One thing I keep coming back to is something I learned early in my career. We would have auditors come in on a yearly basis and examine the IT department, software engineering included, to help identify potential risks and concerns. During this audit, they would look at things like who had access to mission-critical servers, examine how access to those servers was regulated, and even checked that the various procedures were being followed. 

When it came to software development, they pulled a random sample of code changes (normally driven by commit messages in source control), and looked at a couple things around each one - namely, why the change was made, documentation around testing/sign off of the change and _proof that the code that was tested and approved was the same code that ended up in the production environment_. 

That last point has always stuck with me, and I think that when it comes to things like CI/CD, it's something that should always be baked into the system - even if you aren't being audited on a regular basis. As software becomes more pronounced to our every day lives, being able to tie a specific change to a given version (and vice versa) becomes important as well, whether you're writing the next generation of AI software for self-driving cars or a game for people to enjoy on the bus on their way to work.
