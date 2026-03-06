---
author: DBlogAdmin
comments: false
date: 2019-08-17 11:57:58+00:00
layout: post
link: https://dlang.org/blog/2019/08/17/bug-bounties-have-arrived/
slug: bug-bounties-have-arrived
title: Task Bounties Have Arrived
wordpress_id: 2172
categories:
- Community
- D Foundation
permalink: /bug-bounties-have-arrived/
redirect_from: /2019/08/17/bug-bounties-have-arrived/
---

![](http://dlang.org/blog/wp-content/uploads/2018/02/bug.jpg)In 2013, [Facebook launched a page for D bounties at BountySource](https://forum.dlang.org/post/l671kq$1uoh$1@digitalmars.com). It saw a burst of excitement, a few bugs fixed, and then went quiet. By 2017, it [had been "deemed unsuccessful"](https://forum.dlang.org/post/oimn65$134r$1@digitalmars.com). In internal discussions on fundraising options and how to increase community participation in fixing bugs, the topic of bug bounties has often come up, but the failure of the BountySource page has led us to different alternatives and we've never tried to revive it. Last year, before we opened our OpenCollective page, we requested that BountySource refund all unpaid bounties and received confirmation that they had done so. Still, we're always open to new approaches.

In June of this year, Mathias Lang, the CTO of [BPF Korea (BOS Platform Foundation)](https://bosagora.io/team), signaled interest in offering bounties on a few issues and asked the D Language Foundation for suggestions on how to go about it. Given our increasing usage of Flipcause to handle our donations, and the benefits we get in terms of processing fees when more of our donation money is going through Flipcause, it was an easy decision to make. We could set up a menu with multiple campaigns, one for each bounty. Community members would be free to add to money to specific bounties, try their hand at fixing them, or request new bounties be added to the menu.

As of today, [the D Task Bounties menu is live](https://www.flipcause.com/secure/cause_pdetails/NjI2Njg=).


## The Task Bounty System


There are several bounties available, some of which are tied to a Bugzilla issue. The remainder provide a description of the task and the conditions for receiving the bounty. Click a card on the menu to see the details, including the amount of the bounty. You can also increase the bounty of any of the issues by clicking on the card and submitting a donation.

There's also a Task Bounty Catch-All campaign. This is where you can add to the bounties for multiple tasks with one credit card transaction; click the Catch-All card, enter the total amount you'd like to donate, and specify in a comment how the money is to be dispersed. You can also seed a new bounty through the Catch-All campaign; donate the amount you'd like to seed and either provide a Bugzilla issue number in the comment field or go to the menu at the top of the page, click Contact, and provide a detailed description of the issue and the conditions under which the bounty will be payable.

Bounties on any D ecosystem project are acceptable. Where possible, when seeding a new one, please try to connect it to a Bugzilla issue, a GitHub issue, or equivalent in the project's issue tracker. We prefer bounties that are not tied to any issue to be the exception rather than the rule.

The conditions for payment of any bounty tied to a core DLang Bugzilla issue is always the same by default: the issue must be closed through a pull request being merged and subsequently shipped in a release of the affected program (dmd, dub, rdmd, etc..). Conditions may be altered by the person who seeds the bounty (e.g., if you decide to seed a bounty for a Bugzilla issue, you can request that payment be made when the PR is merged). Conditions for payment of bounties on issues for other projects must be stated in the description.

I want to reiterate: the conditions for payment of the bounty are entirely up to the person who seeds it, but they must be set when seeding the bounty (or left to the default in the case of Bugzilla issues).To receive a bounty, please email social@dlang.org when the conditions for payment have been met. You'll need links to the pull request(s) and any other information we need to verify that you're eligible. You'll also need an account with a payment service of some kind (preferably [Circle](https://www.circle.com/pay), [TransferWise](https://transferwise.com/), or [PayPal](https://www.paypal.com/)).

We'll see how it goes and adapt with any rules or restrictions as needed. Our goal is to improve the D ecosystem for everyone, and if some folks can make a little money out of it in the process that's even better. Ideas, suggestions, and feedback are always welcome.


## Other News


I want to thank everyone who has donated $60.00 to [the HR Fund in exchange for a DConf 2019 t-shirt](https://www.flipcause.com/secure/cause_pdetails/NTg4NzE=). I'd also like to thank everyone who has donated directly [to the HR Fund Campaign](https://www.flipcause.com/secure/cause_pdetails/NTUxOTc=) (for a DMan shirt or not). A big thanks especially to WekaIO, who have provided us with the lion's share of the total. With the $360 we've raised with the DConf t-shirts, the fund is currently at $16,345. We still need to keep it growing, so don't get complacent!

The application deadline [for SAOC 2019](https://dlang.org/blog/symmetry-autumn-of-code/) is tomorrow,  midnight AoE. Don't be late!

Finally, for those of you who would like to support the D Language Foundation through Amazon Smile but never remember to go to smile.amazon.com, there's a browser plugin for that! You can get [Smile Always for Chrome](https://chrome.google.com/webstore/detail/smile-always/jgpmhnmjbhgkhpbgelalfpplebgfjmbf?hl=en) or [Smart Amazon Smile for Firefox](https://addons.mozilla.org/en-US/firefox/addon/smart-amazon-smile/?src=search). That way, you can always be sure to support your selected charity with 0.5% of your purchase price when you shop at Amazon.
