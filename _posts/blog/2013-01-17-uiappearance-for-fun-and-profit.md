---
layout: post
status: publish
published: true
title: UIAppearance for fun and profit
author:
  display_name: Josh Johnson
  login: jnjosh
  email: josh@twotoasters.com
  url: ''
author_login: jnjosh
author_email: josh@twotoasters.com
wordpress_id: 39
wordpress_url: http://objectivetoast.com/?p=39
date: '2013-01-17 23:05:22 -0500'
date_gmt: '2013-01-18 04:05:22 -0500'
categories:
- Fundamentals
tags:
- UIAppearance
---
<p>Building custom user interfaces is something our <a href="http://twotoasters.com">iOS development team</a> encounters on a daily basis. Anything you can do to make it faster to develop or make your reusable controls easily customizable is an easy victory. <a href="http://developer.apple.com/library/ios/#documentation/uikit/reference/UIAppearance_Protocol/Reference/Reference.html">UIAppearance</a> gives you this easy victory.</p>
<p>While <a href="http://developer.apple.com/library/ios/#documentation/uikit/reference/UIAppearance_Protocol/Reference/Reference.html">UIAppearance</a> isn't a brand new topic, it seems to get less love than it should. <a href="http://developer.apple.com/library/ios/#documentation/uikit/reference/UIAppearance_Protocol/Reference/Reference.html">UIAppearance</a> is a protocol available since <strong>iOS 5</strong> that allows a developer to quickly configure the appearance of user interface controls provided by Apple. The fact that that your custom controls can take part in UIAppearance is talked about even less. This post will cover the basics of using UIAppearance and will explore adding support to your custom interface components.</p>
<h2>UIAppearance Basics</h2>
<p>You'll be glad to know that <a href="http://developer.apple.com/library/ios/#documentation/uikit/reference/UIAppearance_Protocol/Reference/Reference.html">UIAppearance</a> is a straightforward API with almost no learning curve. To use it you need only know three things.</p>
<ul>
<li>Properties and methods exposed to UIAppearance are decorated with the <code>UI_APPEARANCE_SELECTOR</code> attribute.</li>
<li>The class method <code>+[UIControl appearance]</code> will set the UIAppearance system to apply the specified properties when the control is created.</li>
<li>The class method <code>+[UIControl appearanceWhenContainedIn:]</code> will set the UIAppearance system to apply the specified properties when the control is created, but only when it is created in the specified containers.</li>
</ul>
<p>That is all there is to it. <code>UI_APPEARANCE_SELECTOR</code>, <code>+ appearance</code>, <code>+ appearanceWhenContainedIn:</code></p>
<h2>Adding some style to UINavigationBar</h2>
<p>Let's see how this works. Say you want to style your <em>UINavigationBar</em> to be something a little different than the standard iOS style. First, look at the header file for <em>UINavigationBar</em> (you can find this by right-clicking on the symbol and choosing 'Jump to Definition' on it in Xcode).</p>
<p>Search for <code>UI_APPEARANCE_SELECTOR</code>. You'll see it attached to many of the properties and methods, but not all. This is your guide to what is available by <a href="http://developer.apple.com/library/ios/#documentation/uikit/reference/UIAppearance_Protocol/Reference/Reference.html">UIAppearance</a>. In our case, the first time we see the attribute is with:</p>
<pre><code>@property(nonatomic,retain) UIColor *tintColor UI_APPEARANCE_SELECTOR;
</code></pre>
<p>This means that the tintColor property is available for UIAppearance. This means I can now apply a global style affecting the tint to all <em>UINavigationBar</em> objects by telling the appearance proxy:</p>
<pre><code>[[UINavigationBar appearance] setTintColor:[UIColor redColor]];
</code></pre>
<p>Now, no matter where a <em>UINavigationBar</em> is displayed within my app, it will be tinted red. Classy.</p>
<p>You can make this even more specific by telling the style to only apply when it is inside your own <code>YOURCustomViewController</code>. Consider the code:</p>
<pre><code>[[UINavigationBar appearanceWhenContainedIn:[YOURCustomViewController class], nil]
                               setTintColor:[UIColor redColor]];
</code></pre>
<p>You've now told it to only apply the red tint color when a <em>UINavigationBar</em> is displayed inside a <em>YOURCustomViewController</em>. However, if I add a <em>UINavigationBar</em> to another view controller, it will not be tinted red. Only in the case that an instance of a <em>UINavigationBar</em> is in the hierarchy below an instance of a <em>YOURCustomViewController</em> will it be tinted red. Still classy.</p>
<p>All of these appearance styles can be applied as early as you like in your application. For example, you can set all your styles in <code>-[UIApplicationDelegate application:didFinishLaunchingWithOptions:]</code>.</p>
<h2>Supporting different iOS versions</h2>
<p>As can often occur when working with any API, certain versions of can provide different functionality. How can we build an application level style when the available methods and properties on Apple's controls are changing?</p>
<p>Thankfully, Objective-C is a dynamic language that allows runtime introspection on your objects. In a similar style to how you would ask your object if it <code>-respondsToSelector:</code>, you can ask a class if it's <em>instances</em> respond to a method as well.</p>
<p>Consider this sample:</p>
<pre><code>if ([UINavigationBar instancesRespondToSelector:@selector(setShadowImage:)]) {
    [[UINavigationBar appearance] setShadowImage:awesomeShadow];
}
</code></pre>
<p>The <code>shadowImage</code> property is new to <strong>iOS 6</strong>. Using <code>+[NSObject instancesRespondToSelector:</code>, we can inspect whether <em>any</em> instance of an object of this type will respond to this selector. Since you'll be using the class type to apply an appearance, <strong>not an instance</strong>, this class method will tell you if this property can be used.</p>
<h2>Using UIAppearance in your custom UI views</h2>
<p>As stated earlier, you too can take advantage of UIAppearance in your custom views. Following the same pattern as seen above, let's create a UIButton subclass that provides a property through UIAppearance.</p>
<p>I want to expose the font of the label on a UIButton. After subclassing, I'll create a property that is decorated with ``UI_APPEARANCE_SELECTOR<code>called</code>titleFont`.</p>
<pre><code>@interface TWTButton : UIButton

@property (nonatomic, strong) UIFont *titleFont UI_APPEARANCE_SELECTOR;

@end
</code></pre>
<p>Next, I create the setter implementation and set the title label's font property.</p>
<pre><code>#import "TWTButton.h"

@implementation TWTButton

- (void)setTitleFont:(UIFont *)titleFont
{
    if (_titleFont != titleFont) {
        _titleFont = titleFont;
        [self.titleLabel setFont:_titleFont];
    }
}

@end
</code></pre>
<p>Now, you can set its appearance as if it was always available via <a href="http://developer.apple.com/library/ios/#documentation/uikit/reference/UIAppearance_Protocol/Reference/Reference.html">UIAppearance</a>:</p>
<pre><code>[[TWTButton appearance] setTitleFont:myFont];
</code></pre>
<p>For a more detailed example of this, check out <a href="https://gist.github.com/d20337095c696330460e">TWTButton</a>.</p>
