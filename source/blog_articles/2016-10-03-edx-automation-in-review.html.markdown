---
title: Reviewing My EdX Automation
date: 2016-10-03
tags: automation, edX, yml, capybara, selenium, webdriver, firefox, openedx, edge, SPOC, MOOC, instructor, learner
author: Sam Joseph
---

Michael was away on Friday and I busied myself with admin work.  I was making clones of the "Agile Development using Ruby on Rails" MOOC for instructors around the world (two in the US, one in Ireland).  The instructors are provided with Small Private Online Classes (SPOCs), a term coined by Armando Fox to describe the copies of the MOOC that we make available on edX edge.  Edge is an edX Learning Management System (LMS) separate from the main hosted edX system.  It is different in that the classes don't show up in the main edX listings, and that we are never quite sure if the software is the same version as the main edX instance.  

I've been working with edX for the past few years, and I have lots of interaction with their support folks, but very rarely with their developers.  edX's organisational model is somewhat opaque to me, but I sense this is a somewhat common pattern for larger organisations, where you generally can't talk to the engineers directly. Instead you have to raise issues with non-technical support people.  I've encountered the same with Heroku, Do-It, StackExchange and a few other large-ish organisations.  As a developer myself, I'm not a huge fan of it.  I would prefer to be talking directly to the engineers without the time delay and chinese whispers effect, but I think I understand the respective organisation's fears about overloading their individual developers, and their desire to coordinate their operation at a higher level.  Although I think their fears are somehat unfounded.  I've had a great experience with Travis interacting with their engineers via GitHub.

Anyhow, edX has started doing hourly hangouts once a month to try and explain what they're doing on the open edX side of things, which is a great initiative.  edX have open-sourced all their LMS code (which is awesome) and so anyone can download and install and run their own instance of the edX LMS.  [CraftAcademy](https://craftacademy.se/international/) have been doing this with some success, but I'm unclear about the relationship between the open edX code base and what happens to be running on the hosted edX servers, edge edX and main edX.  Some version names would be super useful, and in the recordings of the open edX hangouts I'm hearing names like "Eucalyptus".  That's great, but what's unclear to me is what version of the edX software is on edX edge or the main edX.  I'd love to have a little "about menu" in the edX interface where I could click through to see the precise version and even a link through to the changelog or features list.

If we hosted our own edX for the SPOC or the MOOC we'd have a lot more clarity about which versions of the software we were running.  Hosting our own is not trivial given that there are various moving parts besides the LMS, such as the edX queues that our auto graders use.  Maybe a day or two to setup and an unclear amount of time for maintenance.  At the moment UCBerkeley pays edX for hosting services, which means we get access to support from non-technical staff, but it's always slightly unclear which version of the software we are working against, or which features are being rolled out when.  In the latest version of the MOOC there have been some changes to the forum software, so that on initial load you see a list of categories of discussions rather than the list of discussions themselves.

![](https://www.dropbox.com/s/k5c8un5gwye6zr6/Screenshot%202016-10-03%2008.53.22.png?dl=1)

I've asked edX support about this, and apparently this is intended to encourage learners to think about where they are posting.  Here's what I see if I click through on "all discussions":

![](https://www.dropbox.com/s/i6mk6xw2r18fhm9/Screenshot%202016-10-03%2008.54.19.png?dl=1)

That's what the initial view used to look like.  I could immediately see the updates.  The first few times I came to the discussion page, I thought there were no discussions there at all, and it took me a few times before I realised that I could click through to see discussions.  Several TAs have confirmed my impression, and we seem to be seeing much lower forum activity than in previous runs of the course.  That said things are picking up now, so perhaps people are getting used to the need to click through.  Then again, perhaps that is entirely unrelated to the interface change.  Difficult to know for certain without a randomised trial.

And who knows, maybe this is a good thing, with some people who might have written out a long question, instead trying things again themselves and solving it on their own.  Again it's very hard to tell.  One thing I do know is that it's slightly disconcerting trying to run a course and not quite knowing what version of the software you are running against and how it might suddenly change.  I guess that's the same for lots of other "Software as a Service" that I use, such as GitHub, Heroku, Google Drive etc.; it's just that with those guys, at least GitHub, I feel like I can generally find a blog post, or something in the interface telling me about any changes.

That said, I was blindsided by some Heroku policy changes over the summer, and edX probably feels more bumpy to me because I have full admin access to our auto graders and can carefully control the version of the software that's being used to grade student assignments, but not the LMS that's being used to handle submission and delivery of them.  edX is trying to support lots more use cases than just our "Agile Development using Ruby on Rails" MOOC, and UCBerkeley is delivering many other MOOCs as well, so our situation is somewhat of a corner case, even if we are one of the most popular MOOCs on the platform.

As part of the deal of being editor of the "Engineering Software as a Service" textbook, I'm also the community manager for the instructors around the world who are working against the MOOC materials.  As I mentioned we currently host SPOCs for those instructors on edge edX, and I have an interface for creating new instances, into which I can import clones of the MOOC.  I've attempted to automate this process with a set of [scripts](https://github.com/saasbook/SPOC).  I'm not particularly proud of the code, but it's an attempt to escape from the robotic operations I have to repeat each time an instructor wants a new instance of a SPOC.

The automation has been only partially successful.  edX as yet has no API for managing courses.  I see they do have some [APIs](https://open.edx.org/open-edx-rest-apis) for enrollment, users and data-analytics on Open edX, which maybe I can use for managing TA accounts next year, but again perhaps that will require us to host our own LMS...?  Anyway, about the automation, I'm using Capybara and Selenium web driver and FireFox 47 to run a headed operation that opens a browser and logs in using my edge account credentials and then types the relevant SPOC information into a form.  I put instructor information in this YML format:

```yml
:joseph_2014: !ruby/struct:Struct::Course 
  title: Software Engineering
  institution: HPU
  id: CSCI4249
  date: 2015_Fall
  contact: sjoseph@hpu.edu
```

And my scripts read that in and then create the course through the web interface, and set the relevant instructor as an admin.  There are then two other steps that I cannot seem to automate.  One is importing the course XML from the MOOC, since the file upload operation involves steps I can't seem to replicate with Selenium, and two, re-setting the course title and id after the upload, which unfortunately gets overridden by the MOOC import.  Both of these operations in the edX interface involve a lot of JavaScript that makes automation difficult.  

I wrote the scripts a while ago, and it may be time to have another go at automating the file upload process, but my fear will be that I'll spend an hour or two on it and still not quite get it working.  A different approach might be to automate manipulating the course XML file locally to fix the name and id, which is more likely to succeed and remove one of those tedious manual steps.  I also worked out on Friday how to merge the two parts of the course, which many instructors want and seems scriptable.  Maybe if I get that all sorted, I can get the system to the point that it takes me to the right page, has the right file ready for upload, and all I need to do is hit a single button?

It's a difficult trade off relating to how many times I will be repeating this operation, which in turn depends on the evolving numbers of SPOC instructors, and the presence of an alternative model called CCX that provides instructors with a skin on the MOOC that is less flexibility in terms of modification.  I guess I'll invest more time in automating all this on another admin day, but pouring resources into this is all predicated on me being a salaried instructor, and I'm still on leave of absence from HPU.  Our instructor community is based on a freemium model where students buying copies of our super-cheap textbook provide the only income, and despite winning some awards the sales are still modest.

Maybe we need to adjust that model?  Armando's working on bundling more coupons with the book.  Perhaps I should be running paid sessions working through book chapters or something?  On the automation side, what I'd also really like from edX is some kind of coordinated course creation API.  My scripts for automating SPOC creation could break any time that edX updates their interface, and they often break when firefox updates.  Of course creating lots of micro-courses is not necessarily part of edX's bigger strategy.  I've put in a lot of volunteer-time because I wanted to join in with the bigger education revolution.  It looks like I'll have to carefully re-assess how to move forward from here.








