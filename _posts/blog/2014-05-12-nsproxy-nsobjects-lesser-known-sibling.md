---
layout: post
status: publish
published: true
title: NSProxy, NSObject’s Lesser-Known Sibling
author:
  display_name: Prachi Gauriar
  login: prachi
  email: prachi@twotoasters.com
  url: http://twitter.com/prachigauriar/
author_login: prachi
author_email: prachi@twotoasters.com
author_url: http://twitter.com/prachigauriar/
wordpress_id: 133
wordpress_url: http://objectivetoast.com/?p=133
date: '2014-05-12 08:00:50 -0400'
date_gmt: '2014-05-12 12:00:50 -0400'
categories: iOS Development Tutorials
tags:
- URLMock
- NSProxy
- Message-Oriented Programming
- Message Forwarding
- Forward Invocation
---
<p>When we were testing <a href="https://github.com/twotoasters/URLMock" title="URLMock on GitHub">URLMock</a>, one of the first things we needed to verify was that we were sending the appropriate messages to <code>NSURLConnection</code> delegates. URLMock can send response data in multiple chunks, so we needed to check that certain delegate messages were received multiple times, while others were received once or not at all.</p>
<p><!--more--></p>
<p>One way to do this would be to add a counter property to our <code>NSURLConnectionDelegate</code> object for each message in the delegate protocol. Then, in our delegate method implementations, we would just increment the appropriate counter. During unit testing, we would just have to verify that each counter has the expected value.</p>
<p>This whole plan of attack is kind of gross. It’s very specific to the one delegate we’re testing, so if we ever need to adapt the pattern to a different protocol, we can’t reuse much of our code. It also doesn’t scale well when you have a lot of delegate methods. Still, I liked the basic idea of counting messages and using those counters in our tests. To implement the idea more elegantly, I decided to use Objective-C message forwarding.</p>
<p>But before we get to that…</p>
<h2>A Brief Digression on Sending Messages</h2>
<p>Any introduction to Objective‑C worth its salt mentions that unlike Java and C#, you don’t <em>call methods</em> in Objective-C, you <em>send messages</em> to objects. The distinction is subtle, so most people just nod their heads and move on. However, the message-oriented nature of Objective‑C is one of its key differentiators from other compiled object-oriented languages. For example, message sending allows you to send the same message to objects of completely different types:</p>
```objc
for (id collection in @[ mutableArray, mutableSet, mutableOrderedSet ]) {
    [collection addObject:@"Hooray for unbounded polymorphism!"];
}
```
<p>You can also construct and send messages dynamically at runtime:</p>
```objc
- (NSSet *)validatorsForKey:(NSString *)key
{
    NSString *capitalizedKey = [key twt_capitalizedCamelCaseString];
    NSString *selectorString = [NSString stringWithFormat:@"validatorsFor%@", capitalizedKey];
    SEL selector = NSSelectorFromString(selectorString);
    return [self respondsToSelector:selector] ? [self performSelector:selector] : nil;
}
```
<p>If you’re feeling feisty, you can do less typical things, like disavowing part of your superclass’s interface:</p>
```objc
- (instancetype)init
{
    // Don’t respond to -init. –initWithObject: should always be used
    [self doesNotRecognizeSelector:_cmd];
    return nil;
}
```
<p>Finally, you can forward messages to another object entirely. It’s this concept that we can use to easily and generally solve the problem of counting how many times an object has received a given message.</p>
<h2>Forwarding Messages with NSProxy</h2>
<p>To implement our message counter, we’re going to subclass one of Cocoa’s more esoteric classes, <a href="https://developer.apple.com/library/ios/documentation/cocoa/reference/foundation/classes/NSProxy_Class/Reference/Reference.html" title="NSProxy Class Reference"><code>NSProxy</code></a>. As the name implies, <code>NSProxy</code> objects stand in for one or more other objects. They are used almost exclusively to forward messages.</p>
<p><code>NSProxy</code> is unique in that it’s the only other root class besides <code>NSObject</code> in the Cocoa frameworks. That is, like <code>NSObject</code>, it has no superclass. The two root classes do have partially overlapping interfaces via the <a href="https://developer.apple.com/library/ios/documentation/cocoa/reference/foundation/Protocols/NSObject_Protocol/Reference/NSObject.html" title="NSObject Protocol Reference"><code>NSObject</code> <em>protocol</em></a>, which declares messages like <code>+alloc</code>, <code>‑respondsToSelector:</code>, and <code>‑performSelector:</code>, but for the most part <code>NSProxy</code> responds to far fewer messages.</p>
<p>To forward messages with <code>NSProxy</code>, you need to override two methods: <a href="https://developer.apple.com/library/ios/documentation/cocoa/reference/foundation/classes/NSProxy_Class/Reference/Reference.html#//apple_ref/occ/instm/NSProxy/methodSignatureForSelector:" title="‑[NSProxy methodSignatureForSelector:] Documentation"><code>‑methodSignatureForSelector:</code></a> and <a href="https://developer.apple.com/library/ios/documentation/cocoa/reference/foundation/classes/NSProxy_Class/Reference/Reference.html#//apple_ref/occ/instm/NSProxy/forwardInvocation:" title="‑[NSProxy forwardInvocation:] Documentation"><code>‑forwardInvocation:</code></a>.</p>
<ul>
<li><code>‑methodSignatureForSelector:</code> responds with an <code>NSMethodSignature</code> object that describes the arguments and return type of the given selector. Returning <code>nil</code> implies that the proxy does not recognize the specified selector. All <code>NSObjects</code> respond to this message as well. </li>
<li><code>-forwardInvocation:</code> does the actual business of forwarding a particular message to the appropriate object.</li>
</ul>
<p>Now that we’ve got a grip on <code>NSProxy</code>, let’s implement our message counter.</p>
<h2>Message Counting Proxy</h2>
<p>The idea of our message counting proxy is pretty simple. We’ll create an <code>NSProxy</code> subclass called <code>UMKMessageCountingProxy</code> which has two fundamental properties: the object for which it’s counting received messages—its <em>proxied object</em>—and a mutable dictionary that maps selectors to the number of times they’ve been received. To use a message counting proxy, we’ll just do something like this:</p>
```objc
id messageCountingProxy = [[UMKMessageCountingProxy alloc] initWithObject:realObject];
[messageCountingProxy foo];

…

NSUInteger count = [messageCountingProxy receivedMessageCountForSelector:@selector(foo:)];
```
<p>Okay, so let’s get to work. <code>UMKMessageCountingProxy</code>’s interface looks like:</p>
```objc
@interface UMKMessageCountingProxy : NSProxy

@property (nonatomic, strong, readonly) NSObject *object;

- (instancetype)initWithObject:(NSObject *)object;
- (NSUInteger)receivedMessageCountForSelector:(SEL)selector;

@end
```
<p>Our implementation file starts with a private class extension and our initializer.</p>
```objc
@interface UMKMessageCountingProxy ()
@property (nonatomic, strong, readonly) NSMutableDictionary *receivedMessageCounts;
@end


@implementation UMKMessageCountingProxy

- (instancetype)initWithObject:(NSObject *)object
{
    NSParameterAssert(object);

    // Don't call [super init], as NSProxy does not recognize -init.
    _object = object;
    _receivedMessageCounts = [[NSMutableDictionary alloc] init];

    return self;
}
```
<p>There’s nothing unusual here except that our initializer doesn’t call its superclass’s. This is because there isn’t one; <code>NSProxy</code> does not respond to <code>‑init</code>.</p>
<p>Next, let’s implement message forwarding. We’ll start by overriding <code>‑methodSignatureForSelector:</code>. We’re supposed to return an <code>NSMethodSignature</code> object for the specified selector. While that might seem complicated, it’s actually really easy. Since we’ll be forwarding the message to the our proxied object, we just ask it for the appropriate method signature.</p>
```objc
- (NSMethodSignature *)methodSignatureForSelector:(SEL)selector
{
    return [self.object methodSignatureForSelector:selector];
}
```
<p>Done.</p>
<p>Next, we need to implement <code>-forwardInvocation:</code>. Our goal here is to increment the count for the appropriate selector, and then re-route the <a href="https://developer.apple.com/library/ios/documentation/cocoa/reference/foundation/classes/NSInvocation_Class/Reference/Reference.html" title="NSInvocation Class Reference">invocation</a> to our proxied object. To do this, we’ll convert the invocation’s selector to a string and use that to look up and increment the current received message count in our <code>receivedMessageCounts</code> dictionary. We’ll then use <code>‑[NSInvocation invokeWithTarget:]</code> to re-invoke the invocation with a different target, namely our proxied object.</p>
```objc
- (void)forwardInvocation:(NSInvocation *)invocation
{
    NSString *selectorString = NSStringFromSelector(invocation.selector);
    NSUInteger count = [self.receivedMessageCounts[selectorString] unsignedIntegerValue];
    self.receivedMessageCounts[selectorString] = @(count + 1);
    [invocation invokeWithTarget:self.object];
}
```
<p>And we’re done with message forwarding. All that’s left is to implement <code>‑receivedMessageCountForSelector:</code> to return the number of times we’ve responded to a given selector. This is implemented exactly how you’d expect:</p>
```objc
- (NSUInteger)receivedMessageCountForSelector:(SEL)selector
{
    return [self.receivedMessageCounts[NSStringFromSelector(selector)] unsignedIntegerValue];
}
```
<p>And that’s all there is to it. Take a look at the full <a href="https://github.com/twotoasters/URLMock/blob/master/URLMock/Utilities/UMKMessageCountingProxy.h" title="UMKMessageCountingProxy.h">interface</a> and <a href="https://github.com/twotoasters/URLMock/blob/master/URLMock/Utilities/UMKMessageCountingProxy.m" title="UMKMessageCountingProxy.m">implementation</a> files to see it all put together.</p>
<h2>Summary</h2>
<p>This is just one use of <code>NSProxy</code>, but there are plenty of others: <a href="https://github.com/erikdoe/ocmock" title="OCMock on GitHub">OCMock</a> uses <code>NSProxy</code> to implement mock objects. You could also use <code>NSProxy</code> to implement the <a href="http://en.wikipedia.org/wiki/Decorator_pattern" title="The Decorator pattern">Decorator pattern</a> in Objective-C. On OS X (but not iOS), Apple provides <a href="https://developer.apple.com/library/mac/documentation/cocoa/reference/foundation/Classes/NSProtocolChecker_Class/Reference/Reference.html" title="NSProtocolChecker Class Reference"><code>NSProtocolChecker</code></a>, a class that only forwards messages to an object if the messages appear in a particular protocol. Creating an iOS implementation is a moderately challenging exercise that I recommend you try. Here’s <a href="https://github.com/objectivetoast/ProtocolChecker" title="ProtocolChecker on GitHub">my solution</a>.</p>
<p>In any case, it’s unlikely that you’ll frequently subclass <code>NSProxy</code>. Still, it’s a powerful class that illustrates how Objective-C’s message-oriented nature can be harnessed to solve certain problems elegantly and concisely, and it’s great to have in your bag of tricks.</p>
<p>Have questions or other potential uses of <code>NSProxy</code>? Get in touch with me on Twitter at <a href="https://twitter.com/prachigauriar/" title="PrachiGauriar on Twitter">@prachigauriar</a>.</p>
