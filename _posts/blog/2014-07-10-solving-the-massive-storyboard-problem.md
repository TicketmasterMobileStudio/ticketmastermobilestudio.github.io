---
layout: post
status: publish
published: true
title: Solving the Massive Storyboard Problem
author:
  display_name: Josh Johnson
  login: jnjosh
  email: josh@twotoasters.com
  url: ''
author_login: jnjosh
author_email: josh@twotoasters.com
wordpress_id: 182
wordpress_url: http://objectivetoast.com/?p=182
date: '2014-07-10 10:27:08 -0400'
date_gmt: '2014-07-10 14:27:08 -0400'
categories: iOS Development
tags:
- Storyboards
- Interface Builder
- Xibs
- Xcode
---
<p><!--<br />
    Author: Josh Johnson<br />
    Categories: Tools, UI<br />
    Tags: Storyboards, Interface Builder, Xibs, Xcode<br />
    Keywords: Storyboards, Interface Builder, Xibs, Xcode<br />
--></p>
<p>At <a href="http://twotoasters.com">Two Toasters</a>, we’ve been building iOS interfaces in code for years. Every so often, someone will suggest that we try using Interface Builder and storyboards, but we’ve traditionally found the experience to be frustrating and unproductive. The tools seemed immature and cumbersome, and with all of our experience building UIs by hand, it never really seemed like a worthy undertaking.</p>
<p>Over the last year though, things have begun to change. There were signs at WWDC 2013 that Apple was putting serious effort into improving storyboards and Interface Builder. A few weeks ago, Apple introduced Adaptive Layout and Size Classes in the iOS 8 SDK, and Xcode 6 features many improvements that aim squarely to address the complaints that people have had about Interface Builder. Apple clearly believes Interface Builder and storyboards are a viable tool for building UIs more productively and efficiently.</p>
<p><!--more--></p>
<p>We recently had a client project that gave us the opportunity to reconsider our stance on Interface Builder and storyboards. While using storyboards was a pain at first, once we got the hang of it, we became a lot more productive and spent a lot less time writing tedious layout code. Throughout the project, we developed patterns and techniques that made us more productive while solving the problems we had run into in the past. This post is the first in a series about those techniques. This week: solving the massive storyboard problem.</p>
<h2>The Massive Storyboard</h2>
<p>One common criticism of Storyboards is that the views for the whole application are left in one massive storyboard file.</p>
<p><img src="/wp-content/uploads/2014/07/crazy-storyboard.png" alt="Bad Storyboard" /></p>
<p>This is overwhelming. Not only is it very difficult to track down the screen or segues you care about, but it can really slow down Xcode. The solution is simple, break up your storyboard.</p>
<p><img src="/wp-content/uploads/2014/07/nicer-storyboard.png" alt="Nicer Storyboard" /></p>
<p>For our project, we identified the core workflows in the app and created a storyboard for each of those workflows. For example, the app’s screens for sign-in and onboarding can be grouped together in <code>Onboarding.storyboard</code>. Screens related to account settings might go in <code>AccountSettings.storyboard</code>.</p>
<p>These are rough examples, but most apps have a pretty good division of the types of actions a user is taking. These are logical boundaries to break up your storyboards. Aside from making it easier to navigate a smaller Storyboard, this also helps reduce opportunities for merge conflicts. It’s also really easy to cross over that boundary between Storyboard files. As an example, instead of connecting a button to another view controller by connecting a segue to it, just connect the button to an <code>IBAction</code> method:</p>
```objc

‑ (IBAction)displayAccountSettings:(id)sender
{
    UIViewController *mainViewController = 
        [[UIStoryboard storyboardWithName:@"AccountSettings" bundle:nil]
         instantiateInitialViewController];

    [self presentViewController:mainViewController
                       animated:YES
                     completion:nil];
}
```
<h2>Keep an open mind</h2>
<p>This has made our storyboards much more sane. You may have had a bad opinion of using storyboards or even Interface Builder because of problems like this. Constantly evaluating and re-evaluating the tools and techniques you use will help you be more productive. Our latest experience with storyboards and Interface Builder has convinced us that the tools are mature enough to help us be more productive. We’ll be writing more about this topic in the future, so stay tuned!</p>
