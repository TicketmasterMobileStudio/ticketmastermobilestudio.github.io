---
layout: post
status: publish
published: true
title: Better Debugging with Quick Look
author:
  display_name: Josh Johnson
  login: jnjosh
  email: josh@twotoasters.com
  url: ''
author_login: jnjosh
author_email: josh@twotoasters.com
wordpress_id: 82
wordpress_url: http://objectivetoast.com/?p=82
date: '2014-03-31 12:14:07 -0400'
date_gmt: '2014-03-31 16:14:07 -0400'
categories:
- Tools
tags:
- Debugging
- Quick Look
---
<p><a href="http://en.wikipedia.org/wiki/Quick_Look">Quick Look</a> was introduced back in OS X 10.5 Leopard as a system-wide quick preview feature for documents and images. When Apple announced Xcode 5 at WWDC 2013, one of the new tools introduced was Quick Look for debugging. With this you could preview <a href="http://objectivetoast.com/wp-content/uploads/2014/03/quicklook.images.png">images</a>, <a href="http://objectivetoast.com/wp-content/uploads/2014/03/quicklook.color_.png">colors</a>, and other common types while stepping through your application. All you have to do is hover over a supported variable and click on the Quick Look icon <img src="http://objectivetoast.com/wp-content/uploads/2014/03/quicklook.png" alt="Quick Look" /> or hit the space bar when the variable is selected in the Variables View.</p>
<p><!--more--></p>
<p>Quick Look is really useful when working with Core Graphics or any custom drawing code. For example, last year when working on <a href="http://twotoasters.com/ideas/2013/luvocracy/">Luvocracy</a>, we designed a sliding panel over each collection view cell. This was implemented as a lightweight <code>CAShapeLayer</code> that was given a path and translated into a view when needed. Initially, I miscalculated the shape of the path and ended up with an incorrectly drawn overlay view. Being able to use Quick Look saved the day. Viewing the calculated path made it much easier to see what was wrong and fix it.</p>
<p><img src="http://objectivetoast.com/wp-content/uploads/2014/03/quicklook.path_.debugging.png" alt="Quick Look during Debugging" /></p>
<h2>Quick Look on Custom Types</h2>
<p>Unfortunately, Xcode 5 only supports Quick Look for built-in types. Thankfully, with the recently released Xcode 5.1, <a href="https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/CustomClassDisplay_in_QuickLook/Introduction/Introduction.html">developers can provide their own Quick Look previews by implementing a single method</a>. Simply override <code>‑debugQuickLookObject</code> and return a Quick Look-compatible type. These Quick Look compatible types can be one of the following:</p>
<ul>
<li><strong>Images</strong> using <code>NSImage</code>, <code>UIImage</code>, <code>CIImage</code> (only if the target is OS X Mavericks or later), or <code>NSBitmapImageRep</code>.</li>
<li><strong>Colors</strong> using <code>NSColor</code> or <code>UIColor</code>.</li>
<li><strong>Bezier Paths</strong> using <code>NSBezierPath</code> or <code>UIBezierPath</code>.</li>
<li><strong>Locations</strong> as <code>CLLocation</code>.</li>
<li><strong>Views</strong> using <code>NSView</code> or <code>UIView</code>.</li>
<li><strong>Strings</strong> with <code>NSString</code> or <code>NSAttributedString</code> and their mutable variants.</li>
<li><strong>URLs</strong> with <code>NSURL</code>.</li>
<li><strong>Data</strong> using <code>NSData</code>.</li>
<li><strong>Cursors</strong> with <code>NSCursor</code>.</li>
</ul>
<p>Apple provides great examples of <a href="https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/CustomClassDisplay_in_QuickLook/CH02-std_objects_support/CH02-std_objects_support.html#//apple_ref/doc/uid/TP40014001-CH3-SW21">what each of these looks like</a>.</p>
<p>With this new debugging feature in Xcode, the example above could have implemented this method and provided a better visualization of the CAShapeLayer. This layer was exposed to the cell via a custom UIView subclass called OverlayView. Since the path was cached until it needed to change, OverlayView could have enabled Quick Look with just one line of code:</p>
<pre><code>#!objc
‑ (id)debugQuickLookObject
{
   return [self overlayPath];
}
</code></pre>
<h2>Let's visualize Auto Layout</h2>
<p>It can be really difficult to debug layout issues when building a view with complicated Auto Layout constraints. Being able to visualize the constraints at runtime could go a long way to help fix broken layouts. Below is a simplistic version of this idea, but it shows what you can do by adding custom Quick Look to your bag of debugging tricks.</p>
<p>The first step is to generate the image we'd like to display. In my sample view, I'll be displaying a simple image and some text all aligned on their center <em>Y</em> values.</p>
<p><img src="http://objectivetoast.com/wp-content/uploads/2014/03/quicklook.sample.png" alt="Sample View" /></p>
<p>I'll create a simple method to create the constraints and, if we're currently debugging, also create the visualization.</p>
<pre><code>#!objc
‑ (void)setupConstraints
{
    NSDictionary *views = NSDictionaryOfVariableBindings(_imageView, _nameLabel);
    NSArray *constraintStrings = @[ @"H:|-20-[_imageView(100)]-20-[_nameLabel]-20-|",
                                    @"V:|[_nameLabel]|", 
                                    @"V:|[_imageView]|" ];

    [self twt_addConstraintsWithVisualFormatStrings:​constraintStrings views:​views];

#if DEBUG 
    [self resetConstraintVisualization]; 
#endif
}
</code></pre>
<p><em>Note: <code>‑twt_addConstraintsWithVisualFormatStrings:​views:</code> is from our <a href="https://github.com/twotoasters/Toast/blob/master/UIKit/Auto%20Layout/UIView%2BTWTConvenientConstraintAddition.h">convenient constraint addition category</a> in <a href="https://github.com/twotoasters/Toast">Toast</a>.</em></p>
<p>The <code>‑resetConstraintVisualization</code> method can now scan through the constraints and draw redlines to help visualize them.</p>
<pre><code>#!objc
‑ (void)resetConstraintVisualization
{
    UIGraphicsBeginImageContextWithOptions(self.bounds.size, NO, 0);

    // draw the existing view hierarchy into the image context
    [self drawViewHierarchyInRect:​self.bounds afterScreenUpdates:​YES];

    // draw red lines for constraints
    CGContextRef context = UIGraphicsGetCurrentContext();
    [[UIColor redColor] setStroke];

    for (NSLayoutConstraint *constraint in self.constraints) {

        // only draw leading edge constraints for simplicity
        if (constraint.firstAttribute == NSLayoutAttributeLeading) {
            UIView *leadingView = (UIView *)[constraint secondItem];
            CGFloat startX = CGRectGetMaxX(leadingView.frame);
            CGFloat startY = CGRectGetMidY(leadingView.frame);
            if (leadingView == self) {
                startX = CGRectGetMinX(self.bounds);
                startY = CGRectGetMidY(self.bounds);
            }

            UIView *trailingView = (UIView *)[constraint firstItem];
            CGFloat endX = CGRectGetMinX(trailingView.frame);
            CGFloat endY = CGRectGetMidY(trailingView.frame);

            // draw line for this constraint
            CGContextMoveToPoint(context, startX, startY);
            CGContextAddLineToPoint(context, endX, endY);
            CGContextStrokePath(context);
        }
    }

    self.debugLayoutImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
}
</code></pre>
<p>The second and final step is the easiest. Implement <code>‑debugQuickLookObject</code> and return <code>self.debugLayoutImage</code>.</p>
<pre><code>#!objc
‑ (id)debugQuickLookObject
{
    return self.debugLayoutImage;
}
</code></pre>
<p><img src="http://objectivetoast.com/wp-content/uploads/2014/03/quicklook.redlines.png" alt="Quick Look at Auto Layout" /></p>
<h2>With great power, comes great responsibility</h2>
<p>It's important to remember that <code>‑debugQuickLookObject</code> runs in the debugger inside a paused application. Your implementation should do as little work as possible and avoid any side effects. The recommended approach is to cache the Quick Look object and simply return it as in the two examples above.</p>
<p>Debugging with Quick Look is a huge boost to productivity. Instead of logging graphics-related objects and properties, you can now visualize what those objects actually look like. This means you can spend less time debugging and more time solving the fun problems in your app.</p>
