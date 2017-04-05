---
layout: post
status: publish
published: true
title: Sharing Utilities Using CocoaPods Subspecs
author:
  display_name: Prachi Gauriar
  login: prachi
  email: prachi@twotoasters.com
  url: http://twitter.com/prachigauriar/
author_login: prachi
author_email: prachi@twotoasters.com
author_url: http://twitter.com/prachigauriar/
wordpress_id: 21
wordpress_url: http://objectivetoast.com/?p=21
date: '2014-03-03 04:00:47 -0500'
date_gmt: '2014-03-03 09:00:47 -0500'
categories:
- Process
tags:
- CocoaPods
- Subspecs
- TWTToast
- Sharing
- Utilities
---
<p>In mid-January, we published <a href="https://github.com/twotoasters/toast" title="TWTToast">TWTToast</a>, a project that collects various utility classes and categories we’ve written over the years. While it’s interesting and useful, we’ll save highlighting its modules for another day. In today’s post, we’re going to focus on how it’s packaged.</p>
<p><!--more--></p>
<p>Until recently, we didn’t have a good way to share lots of small, unrelated utility modules. Packaging them as a monolithic library is bad for users who only need to include a few modules. GitHub Gists granularly organize the modules, but also make them hard to find. Creating a git submodule for each utility module would almost work, but managing so many repositories would be very tedious.</p>
<p>Our ideal solution for organizing and sharing utilities meets three criteria:</p>
<ol>
<li>Users are able to easily include only the modules they need.</li>
<li>Code is available in a central repository that all our team members can easily find, use, and contribute to.</li>
<li>Adding and maintaining modules fits into our current development and review processes.</li>
</ol>
<p>CocoaPods subspecs fulfill all three of these.</p>
<h2>CocoaPods Subspecs</h2>
<p><a href="http://cocoapods.org" title="CocoaPods">CocoaPods</a> is the de-facto standard for distributing libraries in the Cocoa community, but few know about <a href="http://guides.cocoapods.org/syntax/podspec.html#subspec" title="CocoaPods Subspecs">subspecs</a>. Subspecs allow pod authors to break their podspecs into discrete chunks, each of which can be added to a Podfile like any other pod. This allows a library’s users to include only the parts they need, while its developers can treat it as a single project.</p>
<p>With TWTToast, each utility module—typically a header and implementation file—has an associated subspec in our <a href="https://github.com/twotoasters/Toast/blob/master/TWTToast.podspec" title="TWTToast Podspec">podspec</a>. For example, the snippet below defines our <code>UIKit</code> subspec, which has two sub-subspecs: <code>AutoLayout</code> and <code>Color</code>.</p>
<pre><code>#!ruby
## Subspec for Files Related to UIKit
s.subspec 'UIKit' do |ss|
  ss.subspec 'AutoLayout' do |sss|
    sss.source_files = "UIKit/Auto Layout/*.{h,m}"
  end

  ss.subspec 'Color' do |sss|
    sss.source_files = "UIKit/Color/*.{h,m}"
  end

  …
end
</code></pre>
<p>If you only want our <code>Color</code> module, you can get it by adding the following line to your Podfile,</p>
<pre><code>#!ruby
pod 'TWTToast/UIKit/Color'
</code></pre>
<p>None of our other utilities are included in your project. You can specify any combination of modules this way.</p>
<p>We’ve found that this approach works very well for everyone. Module users get easy-to-use, à la carte module inclusion, and we can store all our utilities in a single repository on GitHub, where it fits in with our existing development processes. This makes it easy for us to add and share new modules.</p>
<p>Next time you’re thinking about publishing your utility classes and categories, consider distributing them using CocoaPods subspecs. We think they’re a great, low-overhead way to organize and share utilities with the Cocoa community. Also, go have a look at <a href="https://github.com/twotoasters/toast" title="TWTToast">TWTToast</a>! We’ve got several useful utilities in there now, and we’ll be adding more soon!</p>
