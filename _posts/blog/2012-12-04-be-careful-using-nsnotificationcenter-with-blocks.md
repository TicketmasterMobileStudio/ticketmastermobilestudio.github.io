---
layout: post
status: publish
published: true
title: Be careful using NSNotificationCenter with Blocks
author:
  display_name: Josh Johnson
  login: jnjosh
  email: josh@twotoasters.com
  url: ''
author_login: jnjosh
author_email: josh@twotoasters.com
wordpress_id: 37
wordpress_url: http://objectivetoast.com/?p=37
date: '2012-12-04 22:58:12 -0500'
date_gmt: '2012-12-05 03:58:12 -0500'
categories:
- Fundamentals
tags:
- NSNotificationCenter
- Blocks
---
<p><a href="https://developer.apple.com/library/ios/#documentation/Cocoa/Reference/Foundation/Classes/nsnotificationcenter_Class/Reference/Reference.html">NSNotificationCenter</a> is a long existing mechanism for broadcasting messages to zero or many listeners. Many of Apple's frameworks work deeply by notifiying you via an NSNotification when a message is posted. Traditionally, the workflow has been to follow a pattern similar to this:</p>
<pre><code>- (void)viewDidLoad 
{
    [super viewDidLoad];
    [[NSNotificationCenter defaultCenter] addObserver:self 
                                             selector:@selector(someMethod:) 
                                                 name:kMyNotificationIdentifier 
                                               object:nil];
}

- (void)dealloc
{
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}

- (void)someMethod:(NSNotification *)note
{
    // Message received
}
</code></pre>
<h2>Blocks!</h2>
<p>As of iOS 4 and the introduction of blocks, Mac and iOS developers can now use blocks to subscribe to NSNotification broadcasts. This new method also allows you to specify an NSOperationQueue to perform the block action on.</p>
<pre><code>- (void)viewDidLoad
{
    [super viewDidLoad];
    [[NSNotificationCenter defaultCenter] addObserverForName:kMyNotificationIdentifier 
                                                      object:nil 
                                                       queue:nil
                                                  usingBlock:^(NSNotification *note) {
        // message received
    }];
}
</code></pre>
<h2>Wait, what?</h2>
<p>Not so fast! Who is the observer? What removes this observer? What if you load several view controllers that do this? This block based method is not as simple as it appears. Reading the docs, we learn that <code>addObserverForName:object:queue:usingBlock:</code> actually returns an opaque observer that you are meant to retain, and subsequently <code>-removeObserver</code> with. Let's take a look at what this looks like.</p>
<pre><code>@implementation MyViewController
{
    id _notificationObserver;
}

// ...

- (void)viewDidLoad
{
    [super viewDidLoad];
    _notificationObserver = [[NSNotificationCenter defaultCenter] addObserverForName:kMyNotificationIdentifier 
                                                                              object:nil 
                                                                               queue:nil
                                                                          usingBlock:^(NSNotification *note) {
        // message received
    }];
}

// dealloc, or potentially a method popping this view from the stack
- (void)dealloc
{
    [[NSNotificationCenter defaultCenter] removeObserver:_notificationObserver];
}
</code></pre>
<p>As you can see, you still need to track the observer of a notification and remove it. Similar to what you would do if you were using the selector-based notification listener. Oddly enough, this is an example of a block-based API not really improving things. For this API, the selector-based NSNotificationCenter listener is a much simpler option as you don't have to maintain the observer seperately.</p>
