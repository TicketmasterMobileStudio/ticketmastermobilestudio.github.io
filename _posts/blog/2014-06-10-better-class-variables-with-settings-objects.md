---
layout: post
status: publish
published: true
title: Better Class Variables with Settings Objects
author:
  display_name: Prachi Gauriar
  login: prachi
  email: prachi@twotoasters.com
  url: http://twitter.com/prachigauriar/
author_login: prachi
author_email: prachi@twotoasters.com
author_url: http://twitter.com/prachigauriar/
wordpress_id: 167
wordpress_url: http://objectivetoast.com/?p=167
date: '2014-06-10 08:00:25 -0400'
date_gmt: '2014-06-10 12:00:25 -0400'
categories:
- Patterns
- Fundamentals
tags:
- Class Variables
- Objective-C
- Settings Objects
- Atomic Accessors
- Properties
---
<p><!--<br />
  Author: Prachi Gauriar<br />
  Categories: Design Patterns<br />
  Tags: Class Variables, Objective-C<br />
  Keywords: Keyword 1, Keyword 2, …, Keyword N<br />
--></p>
<p>What a week! WWDC 2014 was chock full of exciting news for developers, and we at Two Toasters ate it up. We’re super excited about extensions, custom keyboards, Hand-off, size classes, and of course Swift. There’s a lot to digest, but rest assured that we’ll be discussing iOS 8’s new APIs and technologies here in the coming weeks and months.</p>
<p>Today though, we’re going to discuss a really simple design pattern that works around one of Objective-C’s shortcomings: the lack of class variables. When we started working on <a href="https://github.com/twotoasters/URLMock/" title="URLMock on GitHub">URLMock</a>, we found ourselves in a bind. Because of <a href="http://objectivetoast.com/2014/05/19/intercepting-requests-with-nsurlprotocol/" title="Intercepting Requests with NSURLProtocol"><code>NSURLProtocol</code>’s architecture</a>, URLMock’s primary interface is the <a href="http://cocoadocs.org/docsets/URLMock/1.1/Classes/UMKMockURLProtocol.html" title="UMKMockURLProtocol Documentation at CocoaDocs.org"><code>UMKMockURLProtocol</code></a> class itself, not instances of it. To add an expected mock request, you have to use <code>+expectMockRequest:</code>; to verify that your URL code is working as expected, you have to use <code>+verifyWithError:</code>. This means that all our bookkeeping data—whether URLMock is enabled, whether verification is enabled, what mock requests are expected, which unexpected requests have been received, etc.—have to be stored at the class level.</p>
<p><!--more--></p>
<p>Objective-C doesn’t support class variables. In most cases, this isn’t a big deal. You can use a <code>static</code> variable and write accessor methods to accomplish what you need:</p>
<pre><code>#!objc
static BOOL UMKMockURLProtocol_enabled;
static NSMutableArray *UMKMockURLProtocol_expectedMockRequests;

// Private methods
+ (BOOL)isEnabled
{
    return UMKMockURLProtocol_enabled;
}

+ (void)setEnabled:(BOOL)enabled
{
    UMKMockURLProtocol_enabled = enabled;
}

…

// Public methods
+ (void)enable
{
    [self setEnabled:YES];
    // Perform other necessary tasks
}

+ (void)disable
{
    [self setEnabled:NO];
    // Perform other necessary tasks
}
</code></pre>
<p>That’s not too bad, right? Unfortunately, these <code>static</code> variables are accessible everywhere in our implementation file, including inside any functions, categories, or secondary classes we implement, so we need to be disciplined to avoid any mishaps. One way to help is to prefix our variable names with our class’s name to distinguish them from non-class variables. To give them initial values, we override <code>+initialize</code>:</p>
<pre><code>#!objc
+ (void)initialize
{
    if (self == [UMKMockURLProtocol class]) {
        UMKMockURLProtocol_enabled = NO;
        UMKMockURLProtocol_expectedMockRequests = [[NSMutableArray alloc] init]];
    } 
}
</code></pre>
<p>This isn’t too bad either, though it’s definitely getting worse. A lot of Objective-C programmers aren’t familiar with <a href="https://developer.apple.com/library/ios/documentation/cocoa/reference/foundation/classes/nsobject_class/reference/reference.html#//apple_ref/occ/clm/NSObject/initialize" title="NSObject +initialize Documentation"><code>+initialize</code></a>, and even fewer understand <a href="https://www.mikeash.com/pyblog/friday-qa-2009-05-22-objective-c-class-loading-and-initialization.html" title="Mike Ash Friday Q&amp;A: 2009-05-22: Objective-C Class Loading and Initialization">why we wrapped our initializers in that if statement</a>, but hey, they’ll learn right?</p>
<p>Note too that our accessors are not <a href="https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html#//apple_ref/doc/uid/TP40011210-CH5-SW37" title="Programming with Objective-C: Encapsulating Data">atomic</a>. Atomic accessors are arguably more useful for class variables than instance variables, since class accessor methods can be easily invoked from any number of threads simultaneously. While writing atomic accessors isn’t horrible, it’s horrible enough with all this other code that I’ve had to write that I’m whiney and sick of this class variable thing and want a better way.</p>
<h2>A Better Way</h2>
<p>The problem with class variables in Objective-C is that we don’t get any of the niceties of declared properties. We muck up our namespace with these static variables that we had to declare, and we waste time writing atomic accessor methods. Things would be so much better if we could just declare class variables like properties. So, let’s try that.</p>
<p>The game plan is simple. Let’s create a simple private class called <code>UMKMockURLProtocolSettings</code>. Its only purpose is to store and manage our class variables. We’ll declare it like so:</p>
<pre><code>#!objc
@interface UMKMockURLProtocolSettings : NSObject

@property (atomic, assign, getter = isEnabled) BOOL enabled;
@property (atomic, assign, getter = isVerificationEnabled) BOOL verificationEnabled;
@property (atomic, strong, readonly) NSMutableArray *expectedMockRequests;
@property (atomic, strong, readonly) NSMutableArray *unexpectedRequests;
@property (atomic, strong, readonly) NSMutableDictionary *servicedRequests;

// And so on…

@end
</code></pre>
<p>Our implementation is pretty trivial:</p>
<pre><code>#!objc
@implementation UMKMockURLProtocolSettings

- (instancetype)init
{
    self = [super init];
    if (self) {
        _expectedMockRequests = [[NSMutableArray alloc] init];
        _unexpectedRequests = [[NSMutableArray alloc] init];
        _servicedRequests = [[NSMutableDictionary alloc] init];
        // And so on…
    }

    return self;
}

@end
</code></pre>
<p>In <code>UMKMockURLProtocol</code>, we declare a private class method that lazily instantiates an instance of this class:</p>
<pre><code>#!objc
+ (UMKMockURLProtocolSettings *)settings
{
    static UMKMockURLProtocolSettings *settings = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&amp;onceToken, ^{
        settings = [[UMKMockURLProtocolSettings alloc] init];
    });

    return settings;
}
</code></pre>
<p>And that’s it. Because the <code>settings</code> variable is declared <code>static</code> inside a method, it doesn’t pollute our namespace. Whenever we want to access a class variable, we first invoke <code>+settings</code>, and then access the variable in question:</p>
<pre><code>#!objc
+ (void)enable
{
    self.settings.enabled = YES;
    // Perform other necessary tasks
}

+ (void)disable
{
    self.settings.enabled = NO;
    // Perform other necessary tasks
}

+ (void)expectMockRequest:(id&lt;UMKMockURLRequest&gt;)mockRequest
{
    …
    [self.settings.expectedMockRequests addObject:mockRequest];
}
</code></pre>
<p>Now we have class variables with atomic accessors that are initialized in a way any Objective-C programmer should understand. But wait, there’s more! Since we now have a dedicated object that manages our class variables, we can add methods to it. For example, URLMock users can reset <code>UMKMockURLProtocol</code>’s various bookkeeping data structures using <code>+reset</code>. It basically clears out these collections in a thread-safe manner. We can implement this in the settings class itself:</p>
<pre><code>#!objc
@implementation UMKMockURLProtocolSettings

…

- (void)reset
{
    // Do some thread safety stuff
    [self.expectedMockRequests removeAllObjects];
    [self.unexpectedRequests removeAllObjects];
    [self.servicedRequests removeAllObjects];
}

@end
</code></pre>
<p>In <code>UMKMockURLProtocol</code>, we just send the settings object the <code>+reset</code> message:</p>
<pre><code>#!objc
+ (void)reset
{
    [self.settings reset];
}
</code></pre>
<h2>Summary</h2>
<p>The concept of a settings object is incredibly simple, but it buys you a lot. First and foremost, it’s easier for most Objective-C programmers to understand and use correctly. It doesn’t require that you name your variables in a special way or use lower-level initialization patterns that are unfamiliar to most developers. You don’t have to pollute your file’s namespace with variables that should only be accessible to your class. You get atomic accessors for free. Finally, you can manage all of your class-level data in one place, which keeps concerns nicely separated and reduces the likelihood of mistakes. We’ve used it in a few different spots in URLMock and other projects at Two Toasters, and we’re really pleased with how much nicer it makes our code. Take a look at <a href="https://github.com/twotoasters/URLMock/blob/master/URLMock/Mock%20URL%20Protocol/UMKMockURLProtocol.m" title="UMKMockURLProtocol Implementation on GitHub">UMKMockURLProtocol’s implementation</a> for a more full-fledged example.</p>
<p>And that’s it for this week’s post. See you next week, <a href="http://youtu.be/iwbsx6LvnfY?t=43s" title="Same Bat-Time, Same Bat-Channel!">same Bat-time, same Bat-channel</a>!</p>
