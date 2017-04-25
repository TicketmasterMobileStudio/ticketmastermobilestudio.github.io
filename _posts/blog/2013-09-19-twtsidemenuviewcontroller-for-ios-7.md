---
layout: post
status: publish
published: true
title: TWTSideMenuViewController for iOS 7
author:
  display_name: Josh Johnson
  login: jnjosh
  email: josh@twotoasters.com
  url: ''
author_login: jnjosh
author_email: josh@twotoasters.com
wordpress_id: 41
wordpress_url: http://objectivetoast.com/?p=41
date: '2013-09-19 23:09:26 -0400'
date_gmt: '2013-09-20 03:09:26 -0400'
categories: iOS Development Open-Source
tags:
- open-source
- two toasters
---
<p>One of the most common implementations of menu views has been the "Side Drawer", "basement", or "Side Menu" made popular in apps such as Facebook and Path. When someone taps the infamous “Hamburger” to open a side menu the main screen slides to the right (or left in some implementations) to reveal another screen <em>below</em>. This works well in iOS 6 and earlier because the status bar exists in a 20pt tall area that is isolated from the rest of the application. In iOS 7, the status bar is overlaid on the screen below it. What does it mean for sidebars or "basement" views in iOS 7?</p>
<p>Soon after iOS 7 was announced at WWDC in June, many designers on <a href="http://dribbble.com">Dribbble</a> started playing around with a new approach for doing these menus. Many came to a <a href="http://dribbble.com/shots/1114754-Social-Feed-iOS7">similar approach</a> <a href="http://dribbble.com/shots/1185823-Side-menu">where the main screen</a> <a href="http://dribbble.com/shots/1154748-WhatsApp-iOS-7-Redesign">would scale down and to the right</a>.</p>
<p>Working with the team at <a href="http://www.luvocracy.com">Luvocracy</a>, we were inspired by this approach and with a few ideas of our own we created <a href="https://github.com/twotoasters/TWTSideMenuViewController">TWTSideMenuViewController</a>.</p>
<p>TWTSideMenuViewController is a container view controller built to manage a menu view and a main view in a beautiful way.</p>
<p><img src="http://cl.ly/UDgW/download/TWTSideMenu.gif" alt="" /></p>
<p>With iOS 7, apps are encouraged to use the whole screen and not rely on the 20pt status bar to be outside of the main screen of your app. This breaks the existing side bar idea in that the status bar with a single style is overlaid on two views, the menu view and the main view.</p>
<p><img src="http://cl.ly/UCmv/download/side-bar-bad.png" alt="" /></p>
<p>With <a href="https://github.com/twotoasters/TWTSideMenuViewController">TWTSideMenuViewController</a> we solve this by moving and scaling the main view down and away from the status bar. Inspired by iOS 7, we also improved the idea of this scaled down view approach to think of the menu and the main view as a single view with the menu and main view in the same visual plane. We merely change the viewport to focus on different aspects of the application. This can be seen in iOS 7 when you launch an app and zoom into it or when you use the new app switcher.</p>
<p>We think it is a clean approach that looks beautiful in iOS 7. Below is a video of it in action in the <a href="https://itunes.apple.com/us/app/luvocracy/id684437187?mt=8">Luvocracy iOS app</a>.</p>
<p><iframe width="420" height="315" src="//www.youtube.com/embed/yOR5O53JJq8?rel=0" frameborder="0" allowfullscreen></iframe></p>
