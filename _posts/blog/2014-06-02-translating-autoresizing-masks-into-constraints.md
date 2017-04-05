---
layout: post
status: publish
published: true
title: Translating Autoresizing Masks Into Constraints
author:
  display_name: Andrew Hershberger
  login: andrew
  email: andrew@twotoasters.com
  url: ''
author_login: andrew
author_email: andrew@twotoasters.com
wordpress_id: 157
wordpress_url: http://objectivetoast.com/?p=157
date: '2014-06-02 10:21:07 -0400'
date_gmt: '2014-06-02 14:21:07 -0400'
categories:
- Fundamentals
- User Interface
tags: []
---
<p>Auto Layout has been a great improvement to the layout system on iOS. It allows expressing layout rules more explicitly and reduces the need for custom layout logic. I especially appreciate that you can mix Auto Layout with manual layout in cases where setting a view's frame expresses the intended behavior more clearly.</p>
<p><!--more--></p>
<p>When I started considering a mixed approach to layout, I faced the question of what to do with <code>translatesAutoresizingMaskIntoConstraints</code>. As explained in the <a href="https://developer.apple.com/library/ios/documentation/userexperience/conceptual/AutolayoutPG/AdoptingAutoLayout/AdoptingAutoLayout.html">Auto Layout Guide</a>,</p>
<blockquote>
<p>"If you have a view that does its own custom layout by calling <code>setFrame:</code>, your existing code should work. Just don't call <code>setTranslatesAutoresizingMaskIntoConstraints:</code> with the argument <code>NO</code> on views that you place manually."</p>
</blockquote>
<p>This approach seemed clear to me until I read Ole Begemann's excellent post about <a href="http://oleb.net/blog/2014/03/how-i-learned-to-stop-worrying-and-love-auto-layout/">mixing manual layout with Auto Layout</a>:</p>
<blockquote>
<p>If you find yourself in a situation that is difficult to solve with Auto Layout, just don’t use it for that particular view…set <code>translatesAutoresizingMaskIntoConstraints = NO</code> on the view since you will position and size it manually, anyway."</p>
</blockquote>
<p>To translate or not to translate? To answer this question, I needed a better understanding of what happens when <code>translatesAutoresizingMaskIntoConstraints</code> is <code>YES</code>.</p>
<p>The <a href="https://developer.apple.com/library/ios/documentation/uikit/reference/uiview_class/UIView/UIView.html#//apple_ref/doc/uid/TP40006816-CH3-SW168">docs</a> describe its behavior as follows:</p>
<blockquote>
<p>If this value is <code>YES</code>, the view’s superview looks at the view’s autoresizing mask, produces constraints that implement it, and adds those constraints to itself (the superview).</p>
</blockquote>
<p>Based on this description, I expected to be able to look in the <code>‑[UIView constraints]</code> array to find the translated constraints, but I quickly learned that the generated constraints are not added there.</p>
<p>After some careful exploration with the debugger, I determined that the constraints are generated on demand during layout by the private method <code>‑[UIView(UIConstraintBasedLayout) _constraintsEquivalentToAutoresizingMask]</code>.</p>
<p>This was an important first step to understanding <code>translatesAutoresizingMaskIntoConstraints</code>. When it is set to <code>YES</code>, the layout system adds the generated constraints to the set of constraints as it solves the layout. When it is set to <code>NO</code>, these constraints are not generated. In this case, if there are other constraints that involve that view they must be sufficient to define its size and position, or if there are no constraints that involve that view, its size and position will stay the same. This means that if you're setting the frame manually, either approach to setting <code>translatesAutoresizingMaskIntoConstraints</code> will work. Turning off translation results in less work for the constraint solver, but whether the CPU savings involved are significant is something to determine on a case-by-case basis.</p>
<p>I could have stopped here. Perhaps, had I listened to the voice of reason, I would have stopped here. Instead I was enticed by a suggestion from Two Toasters' <a href="https://twitter.com/ObjectiveToast/status/453231203488112640">self-proclaimed handsomest developer</a> that it might be informative to reimplement <code>‑_constraintsEquivalentToAutoresizingMask</code>. Very well…</p>
<h2>Let's Build Autoresizing Mask Translation</h2>
<p>The method for my reimplementation is named <code>‑twt_constraintsEquivalentToAutoresizingMask</code>. I also declared <code>‑_constraintsEquivalentToAutoresizingMask</code> in a category on <code>UIView</code> so that I could use it to check my solution. Here's what the test setup looks like:</p>
```objc
@interface UIView (TWTTesting)

- (NSArray *)_constraintsEquivalentToAutoresizingMask;

@end


@implementation TWTViewController

- (void)viewDidLoad
{
    [super viewDidLoad];

    UIView *testView = [[UIView alloc] initWithFrame:CGRectMake(160, 120, 80, 80)];
    [self.view addSubview:testView];

    for (UIViewAutoresizing mask = 0; mask &lt; 8; mask++) {
        testView.autoresizingMask = mask;
        NSLog(@"view: %@", testView);
        NSLog(@"constraints: %@", [testView _constraintsEquivalentToAutoresizingMask]);
        NSLog(@"twt_constraints: %@", [testView twt_constraintsEquivalentToAutoresizingMask]);
    }
}

@end
```
<p>The test iterates from 0 (<code>0b000</code>) to 8 (<code>0b111</code>) to cover all possible combinations of the horizontal bits of the autoresizing mask:</p>
```objc
UIViewAutoresizingNone                 = 0,
UIViewAutoresizingFlexibleLeftMargin   = 1 &lt;&lt; 0,
UIViewAutoresizingFlexibleWidth        = 1 &lt;&lt; 1,
UIViewAutoresizingFlexibleRightMargin  = 1 &lt;&lt; 2,
```
<p>To keep things simple, I only implemented translation for the horizontal autoresizing mask bits since the vertical cases would apply the same logic.</p>
<p>At each iteration, the view's <code>autoresizingMask</code> is set, and then the constraints generated by Apple's translation method and those generated by my reimplementation are logged to the console for manual comparison.</p>
<p>Here are the constraints Apple's implementation generates:</p>
```objc
testView.frame: (160 120; 80 80)
testView.superview.bounds: (0 0; 320 480)
Reimplementation constraints are instances of NSLayoutConstraint.
Apple's constraints are instances of NSAutoresizingMaskLayoutConstraint.

Autoresizing Mask: UIViewAutoresizingNone
Constraints:
    H:[testView(80)]
    testView.centerX == + 200

Autoresizing Mask: UIViewAutoresizingFlexibleLeftMargin
Constraints:
    H:[testView(80)]
    testView.centerX == superview.width - 120

Autoresizing Mask: UIViewAutoresizingFlexibleWidth
Constraints:
    testView.width == superview.width - 240
    testView.centerX == superview.centerX + 40

Autoresizing Mask: UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleWidth
Constraints:
    testView.width == 0.333333 * superview.width - 26.6667
    testView.centerX == 1.66667 * superview.centerX - 66.6667

Autoresizing Mask: UIViewAutoresizingFlexibleRightMargin
Constraints:
    H:[testView(80)]
    testView.centerX == + 200

Autoresizing Mask: UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleRightMargin
Constraints:
    H:[testView(80)]
    testView.centerX == 0.625 * superview.width

Autoresizing Mask: UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleRightMargin
Constraints:
    testView.width == 0.5 * superview.width - 80
    testView.centerX == 0.5 * superview.centerX + 120

Autoresizing Mask: UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleRightMargin
Constraints:
    testView.width == 0.25 * superview.width
    testView.centerX == 0.625 * superview.width
```
<p>In each case, two constraints are generated: one for the view's width and one for its position via <code>centerX</code>. I'll start with the width constraints since there are fewer cases to handle. The easiest is the non-flexible width case. Its constraint is generated like this:</p>
```objc
[NSLayoutConstraint constraintWithItem:self
                             attribute:NSLayoutAttributeWidth
                             relatedBy:NSLayoutRelationEqual
                                toItem:nil
                             attribute:NSLayoutAttributeNotAnAttribute
                            multiplier:0
                              constant:CGRectGetWidth(self.frame)];
```
<p>The flexible width case is a bit more complicated and starts out like this:</p>
```objc
[NSLayoutConstraint constraintWithItem:self
                             attribute:NSLayoutAttributeWidth
                             relatedBy:NSLayoutRelationEqual
                                toItem:self.superview
                             attribute:NSLayoutAttributeWidth
                            multiplier:multiplier
                              constant:constant];
```
<p>The values of <code>multiplier</code> and <code>constant</code> depend on whether the margins are flexible. When both margins are flexible, the width is proportional to the superview's width:</p>
```objc
if (flexibleLeftMargin &amp;&amp; flexibleRightMargin) {
    multiplier = width / superviewWidth;
    constant = 0;
}
```
<p>If only one margin is flexible, the width is proportional to the width of the portion of the superview not covered by the fixed margin. The constant is found by observing that the width of the superview that generates a zero-width for <code>self</code> is the width of the fixed margin, which, written as an equation, is <code>0 = fixedMargin × multiplier + constant</code>. Solving for <code>constant</code> yields the expression we need:</p>
```objc
else if (flexibleLeftMargin) {
    multiplier = width / (superviewWidth - rightMargin);
    constant = 0 - rightMargin * multiplier;
}
else if (flexibleRightMargin) {
    multiplier = width / (superviewWidth - leftMargin);
    constant = 0 - leftMargin * multiplier;
}
```
<p>Finally, if neither margin is flexible, the width is just a fixed difference from the superview's width:</p>
```objc
else {
    multiplier = 1;
    constant = width - superviewWidth;
}
```
<p>The <code>centerX</code> constraints all start out like this:</p>
```objc
[NSLayoutConstraint constraintWithItem:self
                             attribute:NSLayoutAttributeCenterX
                             relatedBy:NSLayoutRelationEqual
                                toItem:self.superview
                             attribute:toItemAttribute
                            multiplier:multiplier
                              constant:constant];
```
<p>The definitions of <code>toItemAttribute</code>, <code>multiplier</code>, and <code>constant</code> break down into six cases. The first one is for a flexible left and right margin and works whether the width is flexible or not:</p>
```objc
if (flexibleLeftMargin &amp;&amp; flexibleRightMargin) {
    toItemAttribute = NSLayoutAttributeRight;
    multiplier = CGRectGetMidX(self.frame) / superviewWidth;
    constant = 0;
}
```
<p>This just keeps <code>self.centerX</code> a consistent percent of the way across its superview.</p>
<p>If you look carefully, you'll notice that this is actually different from the constraint that Apple generates. Apple's translation generates a constraint that relates <code>self.centerX</code> to <code>self.superview.width</code>, but if you try to generate a constraint like that, <code>NSLayoutConstraint</code> raises an exception:</p>
```
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '*** 
+[NSLayoutConstraint constraintWithItem:attribute:relatedBy:toItem:attribute:multiplier:constant:]: 
Invalid pairing of layout attributes'
```
<p>The workaround is to relate <code>self.centerX</code> to <code>self.superview.right</code>, which produces the same effect in cases where the <code>superview.bounds.origin</code> is (0, 0).</p>
<p>Next up are the two cases with one flexible margin and a flexible width:</p>
```objc
else if (flexibleLeftMargin &amp;&amp; flexibleWidth) {
    toItemAttribute = NSLayoutAttributeCenterX;
    multiplier = CGRectGetMidX(self.frame) / (0.5 * (superviewWidth - rightMargin));
    constant = - (rightMargin / 2) * multiplier;
}
else if (flexibleWidth &amp;&amp; flexibleRightMargin) {
    toItemAttribute = NSLayoutAttributeCenterX;
    multiplier = CGRectGetMidX(self.frame) / (0.5 * (superviewWidth - leftMargin));
    constant = leftMargin - (leftMargin / 2) * multiplier;
}
```
<p>These cases are a little harder. The multiplier is set up to adjust <code>self.centerX</code> in proportion to the change in the center of the region of the superview not covered by the fixed margin. The constant is found from the known cases where the width of that region is zero:</p>
<ul>
<li>In the fixed left margin case, <code>self.centerX</code> will equal <code>leftMargin</code> when <code>superview.centerX</code> is <code>leftMargin/2</code>: <code>leftMargin = (leftMargin / 2) × multiplier + constant</code></li>
<li>In the fixed right margin case, <code>self.centerX</code> will equal <code>0</code> when <code>superview.centerX</code> is <code>rightMargin/2</code>: <code>0 = (rightMargin / 2) × multiplier + constant</code></li>
</ul>
<p>The fourth case is for a flexible left margin with everything else fixed:</p>
```objc
else if (flexibleLeftMargin) {
    toItemAttribute = NSLayoutAttributeRight;
    multiplier = 1;
    constant = CGRectGetMidX(self.frame) - CGRectGetMaxX(self.superview.frame);
}
```
<p>In this case, the <code>self.centerX</code> stays a constant distance from the `superview's right edge.</p>
<p>The case for a flexible right margin with everything else fixed is similar:</p>
```objc
else if (flexibleRightMargin) {
    toItemAttribute = NSLayoutAttributeLeft;
    multiplier = 1;
    constant = CGRectGetMidX(self.frame);
}
```
<p>Interestingly, Apple implements this case differently. Apple sets <code>self.centerX</code> equal to a constant, but once again, <code>NSLayoutConstraint</code> raises an exception if you try to do that:</p>
```
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '*** 
+ [NSLayoutConstraint constraintWithItem:attribute:relatedBy:toItem:attribute:multiplier:constant:]: 
A constraint cannot be made that sets a location equal to a constant. Location attributes must be 
specified in pairs'
```
<p>The final case is for when neither margin is flexible but the width is flexible:</p>
```objc
else if (flexibleWidth) {
    toItemAttribute = NSLayoutAttributeCenterX;
    multiplier = 1;
    constant = CGRectGetMidX(self.frame) - CGRectGetMidX(self.superview.bounds);
}
```
<p>This just keeps <code>self.centerX</code> at a fixed offset from <code>superview.centerX</code>.</p>
<p>That's it! Here are the complete results:</p>
```
testView.frame: (160 120; 80 80)
testView.superview.bounds: (0 0; 320 480)
Reimplementation constraints are instances of NSLayoutConstraint.
Apple's constraints are instances of NSAutoresizingMaskLayoutConstraint.

Autoresizing Mask: UIViewAutoresizingNone
Apple's constraints:
    H:[testView(80)]
    testView.centerX == + 200
Reimplementation:
    H:[testView(80)]
    testView.centerX == superview.left + 200

Autoresizing Mask: UIViewAutoresizingFlexibleLeftMargin
Apple's constraints:
    H:[testView(80)]
    testView.centerX == superview.width - 120
Reimplementation:
    H:[testView(80)]
    testView.centerX == superview.right - 120

Autoresizing Mask: UIViewAutoresizingFlexibleWidth
Apple's constraints:
    testView.width == superview.width - 240
    testView.centerX == superview.centerX + 40
Reimplementation:
    testView.width == superview.width - 240
    testView.centerX == superview.centerX + 40

Autoresizing Mask: UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleWidth
Apple's constraints:
    testView.width == 0.333333 * superview.width - 26.6667
    testView.centerX == 1.66667 * superview.centerX - 66.6667
Reimplementation:
    testView.width == 0.333333 * superview.width - 26.6667
    testView.centerX == 1.66667 * superview.centerX - 66.6667

Autoresizing Mask: UIViewAutoresizingFlexibleRightMargin
Apple's constraints:
    H:[testView(80)]
    testView.centerX == + 200
Reimplementation:
    H:[testView(80)]
    testView.centerX == superview.left + 200

Autoresizing Mask: UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleRightMargin
Apple's constraints:
    H:[testView(80)]
    testView.centerX == 0.625 * superview.width
Reimplementation:
    H:[testView(80)]
    testView.centerX == 0.625 * superview.right

Autoresizing Mask: UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleRightMargin
Apple's constraints:
    testView.width == 0.5 * superview.width - 80
    testView.centerX == 0.5 * superview.centerX + 120
Reimplementation:
    testView.width == 0.5 * superview.width - 80
    testView.centerX == 0.5 * superview.centerX + 120

Autoresizing Mask: UIViewAutoresizingFlexibleLeftMargin | UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleRightMargin
Apple's constraints:
    testView.width == 0.25 * superview.width
    testView.centerX == 0.625 * superview.width
Reimplementation:
    testView.width == 0.25 * superview.width
    testView.centerX == 0.625 * superview.right
```
<p>This exercise helped me to demystify the process of translating autoresizing masks, and I hope it has been a help to you as well. You can look at the <a href="https://github.com/ObjectiveToast/TranslatingAutoresizingMasks">complete implementation</a> of this example on GitHub. If you're looking for a challenge, try extending it to implement the translation for the vertical bits. You can find me on Twitter <a href="https://twitter.com/a_hershberger">@a_hershberger</a>. I'd love to hear from you!</p>
