---
layout: post
status: publish
published: true
title: Good Documentation
author:
  display_name: Duncan Lewis
  login: duncan
  email: duncan@twotoasters.com
  url: ''
author_login: duncan
author_email: duncan@twotoasters.com
wordpress_id: 191
wordpress_url: http://objectivetoast.com/?p=191
date: '2014-07-21 10:23:00 -0400'
date_gmt: '2014-07-21 14:23:00 -0400'
categories:
- Process
- Fundamentals
tags:
- Documentation
- Comments
- Best Practice
- HeaderDoc
- Doxygen
---
<p><!--<br />
  Author: Duncan Lewis<br />
  Categories: Documentation, Best Practices<br />
  Tags:<br />
  Keywords: Documentation, Comments, Best Practice, HeaderDoc, Doxygen<br />
--></p>
<p>Every day I find myself relying on well documented code to work efficiently and productively. Good documentation helps me understand what a class or method does, what it was created for, or what that other developer was thinking when they wrote the module my code hinges on. Unfortunately, I have a history of preferring to <em>read</em> good documentation much more than I’ve preferred <em>writing</em> it, often because it seems too difficult and time consuming.</p>
<p>Lately however, I've been working on writing better documentation, and I've found it's much easier than I expected. In this article, I'll share some tips to help you document your own code more quickly and easily. We'll look at what makes or breaks documentation, in which cases it is most vital, and what my process is for writing better docs.</p>
<p><!--more--></p>
<h2>First, What is Bad Documentation?</h2>
<p>The first step to writing good documentation is understanding what makes bad documentation bad. We've all been burned by bad documentation before. It takes many forms: it can be too complicated, omit important details, or even worse can be just plain wrong. Basically, the defining characteristic of bad documentation is that it fails to give the reader the information they need to use a piece of code correctly.</p>
<h2>The Good (Documentation) Stuff</h2>
<p>Good documentation informs the reader of the details they <em>need to know</em>. When someone reads the documentation of your method or class, they're looking for details about how that code behaves and works-details like:</p>
<ul>
<li>What does the code do?
<ul>
<li>Sometimes, the most helpful thing for a reader is understanding what functionality is available to them in a piece of code and whether or not that code can help them accomplish their goals. A conceptual description of a class or method can go a long way to doing this.</li>
</ul>
</li>
<li>Parameters and Edge Cases
<ul>
<li>Readers not only need to know what values to pass to a method, but how the method behaves if they pass in unexpected things, such as <code>nil</code>, or a value outside of an expected range.</li>
</ul>
</li>
<li>Side Effects
<ul>
<li>If invoking a method changes the application or object state or does something non-obvious, like caching a result, this is something the reader will want to know about. </li>
</ul>
</li>
<li>Failure Behavior
<ul>
<li>Does your method have expected failure cases? If so, the person using it will want to know about these cases so they can handle them accordingly.</li>
</ul>
</li>
<li>Execution Environment
<ul>
<li>Is your method not thread safe? Does your method need to be run on the main queue? Does it use a lot of resources or run for a very long time? Again, these are great things to call out in documentation.</li>
</ul>
</li>
<li>Return Values
<ul>
<li>If your return value is non-obvious, it's especially important to note what that value means, and what it tells you about the method's execution.</li>
</ul>
</li>
</ul>
<p>While this might seem like a lot of information to document, you don’t have to write a novel for each of your methods. Good documentation is concise. Don’t burden your readers with any details they don’t need. Think about it like encapsulation: tell them what they need to know to use your code while avoiding too much detail about the implementation.</p>
<h2>Writing Good Documentation</h2>
<p>Even armed with the knowledge of <em>what</em> good documentation is, approaching a chunk of code to document can be daunting. Here are a few steps I use to document a method from scratch.</p>
<ol>
<li>Figure out what your method <em>does</em>.
<ul>
<li>You should be able summarize what your method or class does in a single sentence, as if you were describing it out loud to a friend. </li>
<li>If you're having trouble describing a method in a single sentence, that method might be doing too much. Try refactoring it into smaller, more basic components.</li>
</ul>
</li>
<li>Look at method parameters.
<ul>
<li>Document what each parameter is used for to give the reader an intuitive understanding of how to use it. </li>
<li>If there are any edge cases or other interesting behaviors of your parameters, be sure to include those in your documentation.</li>
</ul>
</li>
<li>Scan your code for side effects, error cases, and execution assumptions.
<ul>
<li>Document the things in your code that affect the caller, like thrown exceptions or changes to object state.</li>
<li>Looking for error/edge cases to call out in your documentation can also help you write better code, because it helps you think critically about how your code handles uncommon situations.</li>
</ul>
</li>
<li>Look at how your code behaves.
<ul>
<li>Does your method behave differently based on the application’s or object’s state? Be sure to document each of the different behaviors and the conditions that trigger them.</li>
</ul>
</li>
<li>Finally, look at your return value.
<ul>
<li>Make sure you communicate what the return value signifies, and the different values it can have.</li>
</ul>
</li>
</ol>
<p>By following these steps and describing your code in straightforward terms, you can quickly and easily churn out some good documentation. Let's give it a try!</p>
<h3>Example Time</h3>
<p>Let’s apply the steps above to write some documentation for some real code. For our example, we’ll use <a href="https://github.com/twotoasters/URLMock">URLMock’s</a> <code>‑[UMKMockHTTPMessage setBodyWithJSONObject:]</code>, which sets the body of an HTTP message to the JSON representation of an object.</p>
<p>Aside: I'll be using the HeaderDoc format (which you can read about below) to structure my comment blocks, so you'll see things like <code>@abstract</code> or <code>@param</code> which are tags helpful to the format's auto-generation engine.</p>
<pre><code>#!objc
- (void)setBodyWithJSONObject:(id)JSONObject
{
    NSData *JSONData = [NSJSONSerialization dataWithJSONObject:JSONObject options:0 error:NULL];
    if (!JSONData) {
        @throw [NSException exceptionWithName:NSInvalidArgumentException
                                       reason:UMKExceptionString(self, _cmd, @"Invalid JSON object")
                                     userInfo:nil];
    }

    self.body = JSONData;
    if (![self valueForHeaderField:kUMKMockHTTPMessageContentTypeHeaderField]) {
        [self setValue:kUMKMockHTTPMessageUTF8JSONContentTypeHeaderValue 
        forHeaderField:kUMKMockHTTPMessageContentTypeHeaderField];
    }
} 
</code></pre>
<p>And as I said earlier, the best place to start documentation is with a description of what the code does. I'll use the <code>@abstract</code> tag that HeaderDoc provides specifically for short descriptions like this.</p>
<pre><code>#!objc
/**
  @abstract Sets the receiver's body to a serialized form of the specified JSON object.
*/
- (void)setBodyWithJSONObject:(id)JSONObject {...}
</code></pre>
<p>Next, lets look at the method's <code>JSONObject</code> parameter. The noteworthy things about it are that it is used as the JSON body for the receiver, and that it must be a valid JSON object (as seen by the error handling around the <code>NSJSONSerialization</code> calls). The documentation of <code>+[NSJSONSerialization<br />
dataWithJSONObject:options:error:]</code> states that its JSON object parameter must not be <code>nil</code> and must be a valid JSON object, we'll echo those requirements in the parameter documentation. This time I'll use the HeaderDoc tag <code>@param</code>.</p>
<pre><code>#!objc
/**
  @abstract Sets the receiver's body to a serialized form of the specified JSON object.

  @param JSONObject The JSON object to serialize and set as the receiver's body. Must not be nil, and 
  must be a valid JSON object.
*/
- (void)setBodyWithJSONObject:(id)JSONObject {...}
</code></pre>
<p>Now we move on to steps 3 and 4 from above. I see that the method handles an error case by throwing an exception, and has a side effect of invoking <code>‑setValue:forHeaderField:</code> if a specific header value isn't already set, so I'll need to call those behaviors out. Note that the <code>kUMKMockHTTPMessageContentTypeHeaderField</code> and <code>kUMKMockHTTPMessageUTF8JSONContentTypeHeaderValue</code> constants correspond to "Content-Type" and "application/json; charset=utf-8", respectively.</p>
<pre><code>#!objc
/**
  @abstract Sets the receiver's body to a serialized form of the specified JSON object.

  @param JSONObject The JSON object to serialize and set as the receiver's body. Must not be
      nil, and must be a valid JSON object.

  @throws NSInvalidArgumentException if JSONObject is not a valid JSON object.

  @discussion If the receiver does not already have a value for the Content-Type header field,
      sets the value of that header to "application/json; charset=utf-8".
*/
- (void)setBodyWithJSONObject:(id)JSONObject {...}
</code></pre>
<p>Lastly, Step 5 would have me look at the return value, but this method doesn't return anything so we're done! We've generated some pretty useful documentation, and it wasn't that hard and didn't take too long.</p>
<h3>Parting Pointers</h3>
<p>Crafting helpful and concise documentation may seem intimidating at first, but hopefully with the tips and steps I've explained, you'll be motivated to write your own good documentation. Before I go, I'll leave you with a few more tips which can help you improve the scope and efficiency of your documentation.</p>
<ul>
<li>Pick a Format
<ul>
<li>It's a good idea to pick a documentation format before starting. Sticking to a consistent format helps us maintain readability as we write our documentation.</li>
<li><a href="https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/HeaderDoc/intro/intro.html">HeaderDoc</a>, <a href="http://gentlebytes.com/appledoc/">appledoc</a> and <a href="www.doxygen.org">doxygen</a> are all popular formats worth checking out.</li>
</ul>
</li>
<li>Use Consistent Terminology
<ul>
<li>When describing a complex system, using consistent terminology for the same ideas or objects helps the reader relate similar functionality across multiple methods. </li>
</ul>
</li>
<li>Mark designated initializers
<ul>
<li>When subclassing and creating your own initializer, make sure to mark the designated initializers for your class. Subclasses need to know what the designated initializer is so that they know to override it.</li>
</ul>
</li>
<li>Create Self-Documenting method names
<ul>
<li>You don't need to write chapters of documentation for every little method. Care to take a guess at what a method like <code>‑(NSInteger)numberOfItems</code> would do on your custom <code>ItemContainer</code> class? There are some cases where you don't need to go overboard with documentation.</li>
</ul>
</li>
<li>Write Documentation as if the reader only has access to your headers, not your implementations.
<ul>
<li>Again, this is the encapsulation point. Documentation should tell the user exactly what they need to know to use your API; no more, no less.</li>
</ul>
</li>
</ul>
<hr />
<p>Now you are armed with all the knowledge and experience you need to go forth and write your own documentation. If you have your own documentation tips, message me on Twitter at <a href="https://twitter.com/dfowj">@dfowj</a>! Thanks for reading!</p>
