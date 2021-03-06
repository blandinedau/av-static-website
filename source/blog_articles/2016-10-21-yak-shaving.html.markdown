---
title: Yak Shaving
date: 2016-10-21
tags: devops heroku pull-requests automation database permissions accounts admin pipeline
author: Sam Joseph
---

So I thought I’d try and get a little solo programming time in today, but it turned into a yak-shaving devops session. Yak shaving? It’s when you end up needing a ladder to paint your house, and you have to borrow a ladder from your friend, but you need to go get his keys from his neighbour who has to finish shaving his yak … I know, it makes no sense - the point is about how you have one goal and then you get successively distracted by a series of activities that become more and more tangentially related to the original goal. I was trying to get our Premium plus upgrade functionality into production. This was going to involve doing a data migration. I wanted to get an instance of the branch with that code up in Heroku to test the migration.

Heroku’s automatic deploy of pull requests to Heroku instances is nice when it works. It came out of beta, but still seems flaky. I started poking at it. First up it was saying that there wasn’t permission to deploy the app. I’d just moved websiteone-develop from my personal account to my company NeuroGrid Ltd group account since all my personal app hours had been burned up by agile-bot. Raoul had originally set up the automated deploys for the websiteone develop server, so somehow the automatic deploy of the PR was coming from his account. I played with moving him from a collaborator to an admin role, which moved things forward a little. The next error was the deploy was failing due to a [database error](https://github.com/AgileVentures/WebsiteOne/issues/1348). One of the frustrating things about this was the delayed feedback loop. I’d make a change in the `app.json` file that specifies how Heroku will auto deploy PRs, or a change in Heroku’s admin console, and it takes five minutes or so to find out if the deploy has worked or not. I was task switching between trying to fix the automated PR deploy and cleaning up what looked like a puffing billy sandbox leak on the branch itself. Ultimately though it was the PR deploy that was taking up more and more of my brain.

Was this another [banana in a coconut](https://blog.craftacademy.se/let-go-of-the-banana/)? Really if I wanted to get the Premium upgrade feature out I should focus on pushing that to some new Heroku instance and leave fixing automated PR deploys to another time. Of course automated PR deploys are great when they work, and they help project managers (including me) review PRs faster, and if it worked I wouldn’t have to do all the fiddly deploy to get the ENV vars set up for a new Heroku instance to test if the new feature actually worked. The great thing about Heroku PR deploys (when they work) is that they copy all the environment vars from a specified server (in this case develop) and things are good to go. However, running `rake db:seed` to pre-populate the database as part of the `app.json` post-deploy script was giving this error message:

```
    FATAL:  permission denied for database "postgres"
    DETAIL:  User does not have CONNECT privilege.
```

but ironically, immediately after that we had all the output from what looked like a successful run of db:create and de:seed. Confusing. We’d had a similar [issue on LocalSupport previously](https://www.pivotaltracker.com/story/show/116276111), which had ultimately been fixed by … I’m not sure. LocalSupport automated Heroku PR deploys had been working for a while. They were now stuck on a separate issue of us having too many apps, even though they were set to auto-delete after 5 days of inactivity. Actually now that I type it out, I think the problem might be that the LocalSupport develop server is on my personal account, and I have a load of junk apps there; gotta clean those out - argh, yak shaving! Either way, the definite trend over the last few months appears to have been Heroku gradually introducing new limits which we are falling foul of in new and unpleasant ways.

Anyhow, the LocalSupport ticket (and an old Heroku ticket) included some ideas for changing app.json, e.g. including an explicit migrate step, and this from the Heroku folks:

> There is a caveat when building review apps that we have requested a new database addon, but it’s not guaranteed to be provisioned during the build phase. It sounds like something in your application is trying to access the database before your database is up and ready to receive connections.
> 
> The ideal fix is to track down why the app is connecting to the database during build and try to prevent that. If that’s not an option, we also have a buildpack that you can use to wait for your database to come up: https://github.com/heroku/heroku-buildpack-addon-wait.

I didn’t get to trying the build pack suggestion, instead I went through a series of changes to app.json; moving the develop server into a heroku pipeline (see image below); an analysis of the stack trace associated with the above database fail, which seemed to possibly implicate [airbrake](https://github.com/airbrake/airbrake/issues/620) but probably not. Ultimately I did not get it working, but did succeed in spamming Raoul and myself with failed Slack invites that get generated as part of the `rake db:setup`. I posted support requests to Heroku, which as of this morning have not been picked up and as of this morning I tried removing the `rake db:setup` completely, which allowed the app to report “successful” deployment, but when I looked there was just nothing there.

![heroku pipeline](https://www.dropbox.com/s/x6bmiswu6j89q8s/Screenshot%202016-10-21%2011.27.48.png?dl=1)

Despite being admin, Raoul reported that he couldn’t even see the develop server in the Heroku GUI, and had removed his account over night. I re-added him, put the `rake db:setup` back in, adjusted the config to allow us to set Slack invites to be disabled on develop, and tried another push. I felt like I could burn a lot of time on this. The next sequence of things to try with this nasty delayed feedback loop are:

1) remove `rake db:setup` again - maybe I can then just run from command line to a) get a working instance and b) understand the error better   
2) try the suggested `heroku-buildpack-addon-wait` from Heroku   
3) re-create the Github web hook that is presumably lending Raoul’s credentials to every attempt to deploy an instance for a PR

Conversely I could say that this is all yak-shaving, and even worse, it’s yak-shaving with a time delay as to knowing when the yak is shaved. If I really wanted to get back on track to deploying the feature I might well be better off just deploying a Heroku instance from scratch and starting work on prodding at the feature. Then I could get my head back to the data migration we need, so that I can write some manual migration instructions, or write a script for it. We will see. Having blogged about all this yak shaving it seems like it’s time to work through all the email and Slack messages that are backing up in my inboxes. Yaks!

### Related Videos

* ["Kent Beck" scrum](https://www.youtube.com/watch?v=ZZa-80c9qos)
