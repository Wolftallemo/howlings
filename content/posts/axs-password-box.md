---
title: "AXS and how to not do UX"
date: 2022-12-18T18:00:00-05:00
draft: false
---

A while ago I made [this tweet](https://twitter.com/i/status/1579674638829121536) about AXS's not
exactly great signup flow. I don't expect perfection (that's near impossible once a project is
sufficiently large enough), but come on, how did they come up with this?

AXS has two main places where you sign in. The first is on the home page, the other is on FanSight
(tix.axs.com) during checkout. This section specifically concerns FanSight.

To start off, can you tell the difference between what
[this login form](/2022/12/axs-home-login.png) and
[this login form](/2022/12/axs-fansight-login.png) is? The difference is how created sessions are
handled. If you log in via the home page, you get full access to your account as you would expect.
But if you log in through FanSight, things don't work the same way.

When you log in through FanSite, you expect that the session works across the entire site right?
Well, that's not what happens. Sessions initiated through FanSite expire far faster than normal
sessions, and despite also working on the home page, they do not work on the account settings page.
Which means in order to access the tickets you just purchased, you now have to log in a second
time.

About the shorter expiration times, when I say shorter, I mean way too short. They seem to only be
valid for about 10 minutes from my testing, whereas normal sessions last far longer.

In contrast, (even though I hate them otherwise) Ticketmaster at least has a somewhat sane auth
implementation, keeping everything under `auth.ticketmaster.com`
