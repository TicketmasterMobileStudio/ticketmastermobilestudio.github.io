---
layout: post
status: publish
published: true
title: Open Source Happenings
author:
  display_name: Prachi Gauriar
  login: prachi
  email: prachi@twotoasters.com
  url: http://twitter.com/prachigauriar/
author_login: prachi
author_email: prachi@twotoasters.com
author_url: http://twitter.com/prachigauriar/
wordpress_id: 179
wordpress_url: http://objectivetoast.com/?p=179
date: '2014-07-02 11:15:15 -0400'
date_gmt: '2014-07-02 15:15:15 -0400'
categories:
- Tools
tags:
- TWTToast
- open-source
- URLMock
- TWTValidation
- frameworks
---
<p><!--<br />
  Author: Prachi Gauriar<br />
  Categories: Tools<br />
  Tags: Open Source, URLMock, TWTValidation, TWTToast, frameworks<br />
  Keywords: Open Source, URLMock, TWTValidation, TWTToast, frameworks<br />
--></p>
<p>It’s been a busy few weeks in <a href="http://twotoasters.com" title="Two Toasters">Two Toasters</a> land! In addition to our normal client workload, we’ve also had major releases of each of our three major open source projects. Here’s a quick look at what’s new.</p>
<p><!--more--></p>
<h2>URLMock 1.2</h2>
<p>This week, we released <a href="https://github.com/twotoasters/URLMock" title="URLMock on GitHub">URLMock 1.2</a>! The new version adds the following major features:</p>
<ul>
<li>Support for stream-based requests, including those created with <code>NSURLSession</code>.</li>
<li>More complete support for Rails/Rack-style nested parameter query strings. We should now be able to encode and decode query strings with deeply nested objects, e.g., strings inside dictionaries inside arrays.</li>
<li>Pattern-based request matching using <code>UMKPatternMatchingMockRequest</code>. </li>
</ul>
<p>We think this last feature will really improve the way you use URLMock for mocking and stubbing. With <code>UMKPatternMatchingMockRequest</code>, you can match requests using a URL pattern and generate a response dynamically using a block. Here’s a quick example from the README:</p>
<pre><code>NSString *pattern = @"http://hostname.com/accounts/:accountID/followers";
UMKPatternMatchingMockRequest *mockRequest =  [[UMKPatternMatchingMockRequest alloc] initWithPattern:pattern];
mockRequest.HTTPMethods = [NSSet setWithObject:kUMKMockHTTPRequestPostMethod];

mockRequest.responderGenerationBlock = ^id&lt;UMKMockURLResponder&gt;(NSURLRequest *request, NSDictionary *params) {
    NSDictionary *requestJSON = [request umk_JSONObjectFromHTTPBody];

    // Respond with 
    //   { 
    //     "follower_id": «New follower’s ID»,
    //     "following_id":  «Account ID that was POSTed to» 
    //   }
    UMKMockHTTPResponder *responder = [UMKMockHTTPResponder mockHTTPResponderWithStatusCode:200];
    [responder setBodyWithJSONObject:@{ @"follower_id" : requestJSON[@"follower_id"] 
                                        @"following_id" : @([params[@"accountID"] integerValue]) }];
    return responder;
};

[UMKMockURLProtocol addExpectedMockRequest:mockRequest];
</code></pre>
<p>This mock request matches all requests whose URL matches the pattern http://hostname.com/accounts/<em>accountID</em>/followers, where <em>accountID</em> is an account ID placeholder. The mock request dynamically generates a responder using its responder generation block. In the example above, the mock request generates a response that includes data from both the request’s URL and body. See the <a href="http://cocoadocs.org/docsets/URLMock/1.2.1/Classes/UMKPatternMatchingMockRequest.html" title="UMKPatternMatchingMockRequest Documentation">UMKPatternMatchingMockRequest</a> docs for more specifics about how to use pattern-matching mock requests.</p>
<p>We’re really proud of this release. Drop by our <a href="https://github.com/twotoasters/URLMock" title="URLMock on GitHub">GitHub page</a> for more details!</p>
<h2>TWTValidation 1.0</h2>
<p>We’re also happy to announce a brand new framework from the Two Toasters team, <a href="https://github.com/twotoasters/TWTValidation" title="TWTValidation on GitHub">TWTValidation</a>! TWTValidation is a Cocoa framework for declaratively validating data. Inspired by Rails validators, TWTValidation makes it easy to declare how your objects should be validated. You can then validate your objects whenever you see fit and get detailed error objects that explain why validation failed.</p>
<p>There’s enough in TWTValidation to warrant a post by itself, so be on the lookout for that in the coming weeks. In the meantime, check out the README and example project on <a href="https://github.com/twotoasters/TWTValidation" title="TWTValidation on GitHub">GitHub</a> to get a taste of what you can do.</p>
<h2>TWTToast 0.10</h2>
<p>Finally, we’ve been steadily adding new features and functionality to our utilities project, <a href="https://github.com/twotoasters/Toast" title="TWTToast on GitHub">TWTToast</a>. In the last few months, we’ve added classes and categories that make key-value observing easier; simplify model class deserialization with <a href="https://github.com/Mantle/Mantle" title="Mantle on GitHub">Mantle</a>; clean up segue preparation code; and more. TWTToast is used in nearly all of our client projects these days and is bound to contain something you’ll find useful. <a href="https://github.com/twotoasters/Toast" title="TWTToast on GitHub">Take a look</a>!</p>
<hr />
<p>That’s it for this week! We’re really proud of these releases, and we hope you find them useful. We’re always looking for ways to improve our open source offerings, so if you find a bug or have an idea for a new feature, open an issue on GitHub!</p>
