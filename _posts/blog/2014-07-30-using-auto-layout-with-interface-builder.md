---
layout: post
status: publish
published: true
title: Using Auto Layout with Interface Builder
author:
  display_name: Steve Foster
  login: steve
  email: steve@twotoasters.com
  url: ''
author_login: steve
author_email: steve@twotoasters.com
wordpress_id: 196
wordpress_url: http://objectivetoast.com/?p=196
date: '2014-07-30 14:49:22 -0400'
date_gmt: '2014-07-30 18:49:22 -0400'
categories:
- Tools
tags:
- Interface Builder
- Xcode
- Auto Layout
- Developer Workflows
---
<p>I'll admit it, I was an Interface Builder hater. In the past everything I did was in code: Auto Layout, custom views, custom cells, everything. The frustrations of Xcode 4's implicit constraint generation made using Auto Layout difficult. Going to code was far simpler.</p>
<p><!--more--></p>
<p>Of course there are downsides to code. More code means more maintenance, more opportunity for errors and the seemingly endless <em>Build, Run, Crash, Fix Constraints</em> development cycle.</p>
<p>Apple made huge improvements to Interface Builder in Xcode 5 that make using it with Auto Layout a lot better. First and formost, it no longer implicitly adds constraints which results in more predictable layouts. More importantly, it warns you if your layout is ambiguous or broken, which basically eliminates the <em>Build, Run, Crash, Fix</em> cycle.</p>
<p>Using Interface Builder has been an adjustment, and there are definitely times that I want to delete a non-cooperative XIB and just do it in code. However I’m finding myself much more productive when it comes to Auto Layout. Here are a few techniques that have really helped me integrate Interface Builder into my workflow.</p>
<h2>Renaming Constraints</h2>
<p>As you create constraints in Interface Builder, they appear in the View Hierarchy list. Unfortunately, Xcode 5 doesn’t allow you to reorder the items in this view, and it maddeningly limits how wide the list can be expanded. This makes it difficult to distinguish between constraints at a glance.</p>
<p><a href="http://objectivetoast.com/wp-content/uploads/2014/07/Renaming-Constraints.png"><img src="http://objectivetoast.com/wp-content/uploads/2014/07/Renaming-Constraints-273x300.png" alt="Renaming Constraints" width="273" height="300" class="alignnone size-medium wp-image-204" /></a></p>
<p>Fortunately, you can rename your constraints. These names are only used for the View Hierarchy list, so they have no effect on the way your app works. It doesn’t solve all my problems with the View Hierarchy list, but it helps.</p>
<h2>Intrinsic Sizes &amp; Placeholders</h2>
<p>In most cases, the intrinsic content size of a view is not known until runtime. Leaving the view without any constraints will cause Interface Builder to complain about an ambiguous or broken layout. To squelch these warning messages, use placeholder constraints and mark them to be removed at runtime.</p>
<h3>Configuring Placeholder Constraints</h3>
<ol>
<li>Select a view </li>
<li>Open the Size inspector</li>
<li>Change the Intrinsic Size drop-down menu from Default (System Defined) to Placeholder. </li>
<li>Define the Width and/or Height.</li>
</ol>
<p><a href="http://objectivetoast.com/wp-content/uploads/2014/07/Placeholder1.png"><img src="http://objectivetoast.com/wp-content/uploads/2014/07/Placeholder1-300x253.png" alt="Placeholder1" width="300" height="253" class="alignnone size-medium wp-image-200" /></a><a href="http://objectivetoast.com/wp-content/uploads/2014/07/Placeholder2.png"><img src="http://objectivetoast.com/wp-content/uploads/2014/07/Placeholder2-300x165.png" alt="Placeholder2" width="300" height="165" class="alignnone size-medium wp-image-201" /></a></p>
<h3>Defining a Constraint to be Removed at Runtime</h3>
<ol>
<li>Select a view</li>
<li>Open the Size inspector</li>
<li>Select the down-arrow next to the gear icon and click on Select &amp; Edit.</li>
<li>At the bottom of the constraint configuration window check the box next to Placeholder for Remove at build time.</li>
</ol>
<p><a href="http://objectivetoast.com/wp-content/uploads/2014/07/RemoveAtBuildtime.png"><img src="http://objectivetoast.com/wp-content/uploads/2014/07/RemoveAtBuildtime-300x175.png" alt="RemoveAtBuildtime" width="300" height="175" class="alignnone size-medium wp-image-203" /></a></p>
<p>In this situation you will obviously need to to build and run to validate your constraints.</p>
<h2>Control-dragging in the View Hierarchy</h2>
<p>Within the View Hierarchy it is possible to set constraints by Control-dragging from one object to another. I find this to be the easiest way to add constraints. To do this, hold down the Control key and drag from one view to another the way you would when connecting an IBOutlet. Upon release you are presented with a popup menu of options for setting constraints.</p>
<p><a href="http://objectivetoast.com/wp-content/uploads/2014/07/Control-Dragging.png"><img src="http://objectivetoast.com/wp-content/uploads/2014/07/Control-Dragging-210x300.png" alt="Control-Dragging" width="210" height="300" class="alignnone size-medium wp-image-198" /></a></p>
<h2>IBOutlets</h2>
<p>IBOutlets are basically just object properties that bridge your XIBs to your code. Like other properties, they’re not just limited to UI elements. You can have IBOutlets to anything, including Auto Layout constraints that you’ve created in Interface Builder. This is great for situations where your layout changes at runtime by modifying a constraint’s constant. Connect an outlet to the constraint in Interface Builder, and modify its constant in code at runtime.</p>
<p><a href="http://objectivetoast.com/wp-content/uploads/2014/07/IBOutlet.png"><img src="http://objectivetoast.com/wp-content/uploads/2014/07/IBOutlet-300x116.png" alt="IBOutlet" width="300" height="116" class="alignnone size-medium wp-image-199" /></a></p>
<hr />
<p>Apple has made a lot of great strides with Interface Builder, with Auto Layout seeing some of the best improvements as far as I’m concerned. The tips and techniques above made adopting Interface Builder a lot easier for me, and I hope they help you too. If you have any questions or cool Interface Builder tips of your own, share them with me on Twitter <a href="https://twitter.com/flightblog">@flightblog</a>.</p>
