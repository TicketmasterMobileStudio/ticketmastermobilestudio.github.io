---
layout: post
status: publish
published: true
title: NSNumberFormatter and Objective-C Types
author:
  display_name: Andrew Hershberger
  login: andrew
  email: andrew@twotoasters.com
  url: ''
author_login: andrew
author_email: andrew@twotoasters.com
wordpress_id: 129
wordpress_url: http://objectivetoast.com/?p=129
date: '2014-05-05 13:55:47 -0400'
date_gmt: '2014-05-05 17:55:47 -0400'
categories:
- Fundamentals
tags:
- NSNumberFormatter
- multiplier
- Objective-C types
- gotcha
- bug
---
<p>On a recent project, I ran into some very unexpected behavior with <code>NSNumberFormatter</code>. I was working with an API that delivered price information in cents. The price needed to be displayed in dollars. I decided to store the value in cents to maintain consistency with the API and to use <code>NSNumberFormatter</code> to generate the string for the UI. It seemed like a great solution.</p>
<p><!--more--></p>
<p>Here's how it worked:</p>
<pre><code>#!objc
// Ninety-nine dollars in cents (this comes from the API)
NSNumber *numberInCents = @9900;

NSNumberFormatter *numberFormatter = [[NSNumberFormatter alloc] init];

[numberFormatter setNumberStyle:NSNumberFormatterCurrencyStyle];
[numberFormatter setMultiplier:@0.01];

NSString *formattedString = [numberFormatter stringFromNumber:numberInCents];
</code></pre>
<p>This generated the string <code>$99.00</code>. So far, so good.</p>
<p>For this particular project, one of the requirements was to remove the fractional part of the formatted number if it was zero. I got that working, but I wanted to make sure that the fractional part was still displayed when necessary. To test it, I changed the value of <code>numberInCents</code> to <code>@9999</code>. The result? <code>$99.00</code>. What was going on?</p>
<p>After much confusion, I came across a <a href="http://stackoverflow.com/q/8970954/171089">question</a> from someone with the same problem. The issue was that the <a href="https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html">Objective‑C type</a> of <code>numberInCents</code> was <code>i</code> which stands for <code>int</code>. You can get the Objective-C type from any <code>NSValue</code>—the superclass of <code>NSNumber</code>—with the method <code>‑[NSValue objCType]</code>. For some reason, <code>NSNumberFormatter</code> takes this into account when applying the multiplier. The behavior is similar to what you would get when doing an integer division: <code>9999 / 100 = 99</code>.</p>
<p>The workaround was to make sure that the Objective‑C type of the number was always a floating-point type:</p>
<pre><code>#!objc
// Ninety-nine dollars, ninety-nine cents in cents (this comes from the API)
NSNumber *numberInCents = @9999;

NSNumberFormatter *numberFormatter = [[NSNumberFormatter alloc] init];

[numberFormatter setNumberStyle:NSNumberFormatterCurrencyStyle];
[numberFormatter setMultiplier:@0.01];

// Ensure that the number to format has a floating-point Objective-C type
NSNumber *floatingPointNumberInCents = @([numberInCents floatValue]);
NSString *formattedString = [numberFormatter stringFromNumber:floatingPointNumberInCents];
</code></pre>
<p>This gave the expected result: <code>$99.99</code>.</p>
<p>It is not clear whether this is the intended behavior for <code>NSNumberFormatter</code>, so I filed a radar (<a href="http://www.openradar.me/radar?id=5897585755684864">rdar://16775214</a>) to request clarification. Fortunately for now, the workaround is straightforward. Have you encountered any similar issues with <code>NSNumberFormatter</code>? Let me know <a href="https://twitter.com/a_hershberger">@a_hershberger</a> on Twitter.</p>
