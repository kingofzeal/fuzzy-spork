---
title: "DevOps and Management"
date: 2018-03-25T12:29:54-05:00
draft: true
---

I recently gave a talk at [NEW CodeCamp](http://newcodecamp.com) on a topic that over the last several months has become something I enjoy talking about and working with - Octopus Deploy. After the session, I was talking with someone who attended about their own deployment process. After thinking about his situation, I've decided to write my thoughts on the situation.

The situation he described is not likely all that unusual. He basically told me that their deployment of updates mainly consisted of dropping only the modified assemblies or files into their production environment. In the case of websites, it would be HTML or ASPX files, and in the case of a windows application it would be the supporting binary DLLs (the primary executable would rarely change, understandably).

From the perspective of the company management, this is a very attractive method of doing things: if you have multiple bug fixes in the works, it's easy to deploy the fix in any order once it has been tested (assuming they change different files), and the process appears to be relatively low-risk. It also allows their testing pipeline to be filled relatively efficiently while giving the impression of a real CI/CD pipeline to production.

