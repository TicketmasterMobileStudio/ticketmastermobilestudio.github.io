---
layout: post
status: publish
published: true
title: Intercepting Requests with NSURLProtocol
author:
  display_name: Josh Johnson
  login: jnjosh
  email: josh@twotoasters.com
  url: ''
author_login: jnjosh
author_email: josh@twotoasters.com
wordpress_id: 148
wordpress_url: http://objectivetoast.com/?p=148
date: '2014-05-19 10:53:02 -0400'
date_gmt: '2014-05-19 14:53:02 -0400'
categories:
- Fundamentals
tags:
- URLMock
- URL Loading
- Networking
- NSURLProtocol
---
<p>We’ve been discussing <a href="https://github.com/twotoasters/URLMock">URLMock</a> a lot recently. We <a href="http://objectivetoast.com/2014/04/28/introducing-urlmock/">introduced the library</a> and provided some use cases for it. Later, we <a href="http://objectivetoast.com/2014/05/12/nsproxy-nsobjects-lesser-known-sibling/">discussed how we used NSProxy</a> to build the message counting system URLMock uses for testing. That’s all great, but how does URLMock actually work?</p>
<p><!--more--></p>
<p>To step back for a moment, let’s review what URLMock is actually doing. From our earlier post:</p>
<blockquote>
<p><em>“URLMock is an open-source Cocoa framework for mocking and stubbing URL requests and responses … It works seamlessly with apps that use the NSURL loading system, e.g., NSURLConnection or AFNetworking. You don’t even have to change your client code.”</em></p>
</blockquote>
<p>There are a few things to take away from this:</p>
<ol>
<li>URLMock provides mocking or stubbing for URL requests and responses.</li>
<li>You don’t have to change your client code if it uses <code>NSURLConnection</code> or AFNetworking.</li>
<li>While we don't mention this explicitly, URLMock doesn't use any private APIs to intercept your client code's requests and respond to them.</li>
</ol>
<p><code>UMKMockURLProtocol</code> is the the primary interface into URLMock. Once you enable this protocol it intercepts your app’s requests and responds to them as you specify. So, how does it work? How can we intercept requests sent by your application while being a good citizen and avoiding private APIs?</p>
<h2>Introducing NSURLProtocol</h2>
<p>Foundation’s <a href="https://developer.apple.com/library/mac/documentation/cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html#//apple_ref/doc/uid/10000165i">URL loading system</a> provides many handlers for responding to URL-based requests. Out of the box, it supports for several protocols, including <code>http:</code>, <code>https:</code>, <code>ftp:</code>, <code>file:</code>, <code>data:</code>. What if you had your own protocol that’s not supported? Do you have to go build the whole URL stack from scratch?</p>
<p>Thankfully, the answer is no. <a href="https://developer.apple.com/library/mac/documentation/cocoa/reference/foundation/classes/NSURLProtocol_Class/Reference/Reference.html"><code>NSURLProtocol</code></a> is your hook into Foundation’s URL Loading system. You create a subclass of <code>NSURLProtocol</code> that implements your protocol and register it with the URL loading system. When a request is sent, all registered protocols—from the most to least recently registered—are asked if they can handle the request. The first protocol to respond positively is chosen to handle the request. You don't even need to create an instance of your protocol, the system does this for you.</p>
<p>Once your protocol is chosen, it is sent the <code>‑startLoading</code> message where you can decide how to respond to the request. At this point, your responsibilities are to provide the <code>client</code> (a property of <code>NSURLProtocol</code> that conforms to the <a href="https://developer.apple.com/library/ios/documentation/cocoa/reference/foundation/Protocols/NSURLProtocolClient_Protocol/Reference/Reference.html"><code>NSURLProtocolClient</code> protocol</a>) with information about the loading process. If you've ever implemented the delegate callbacks for <code>NSURLConnection</code>, these methods should look a bit familiar to you.</p>
<p>The best part here is that your protocols take precedence over Apple’s, so you can change basic URL loading behavior for existing protocols like <code>http</code>. This is exactly how <a href="https://github.com/twotoasters/URLMock">URLMock</a> works. When you enable <code>UMKMockURLProtocol</code>, you are registering it as a handler in the URL loading system. When asked if it can handle a request, it checks its list of registered mock requests and returns <code>YES</code> if it finds a match.</p>
<p>Let’s see how this works by implementing our own <code>NSURLProtocol</code>.</p>
<h2>Introducing TWTHasselhoffImageProtocol</h2>
<p>Suppose you are working on displaying remote images in your app and are about to board an airplane. Sure, you could temporarily change all your images to be local, but then your client code would be littered with all these changes that you’ll need to delete later. Let’s make a protocol that responds with an image from the bundle whenever an image is requested. For maximum awesomeness, let’s make that image always be a photo of David Hasselhoff.</p>
<p>First we have our client code. This is the code that will request the photo we'd like to display.</p>
```objc
NSString *kittenImageString = @"http://cutekittens.com/kittens_cute_kitten.jpg";
NSURL *url = [NSURL URLWithString:kittenImageString];

NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
[request addValue:@"image/jpeg" forHTTPHeaderField:@"Content-Type"];

[NSURLConnection sendAsynchronousRequest:request queue:self.imageQueue completionHandler:
^(NSURLResponse *response, NSData *data, NSError *connectionError) {

    // Create the image directly from the data response.
    UIImage *kittenImage = [UIImage imageWithData:data scale:0.0];

    // Update the image view with the image    
    [[NSOperationQueue mainQueue] addOperation:[NSBlockOperation blockOperationWithBlock:^{
        self.imageView.image = kittenImage;
    }]];
}];
```
<p>This works as expected. We run the application and see our cute kitten.</p>
<p>Now we start to have some fun. We’ll create <code>TWTHasselhoffImageProtocol</code>, which intercepts image requests and responds with some Hasselhoff. You <a href="https://gist.github.com/jnjosh/6f078e93f1df9a3dc14c">can download this class and follow along</a>.</p>
<p>The first thing we need to do is tell the URL loading system if we can handle this request. We do this using <code>+canInitWithRequest:</code>. We’ll return <code>YES</code> if the request is for a PNG or JPEG.</p>
```objc
+ (BOOL)canInitWithRequest:(NSURLRequest *)request
{
    NSSet *validContentTypes = [NSSet setWithArray:@[ @"image/png",
                                                      @"image/jpg",
                                                      @"image/jpeg" ]];

    return [validContentTypes containsObject:request.allHTTPHeaderFields[@"Content-Type"]];
}
```
<p>Next we'll implement <code>‑startLoading</code> and <code>‑stopLoading</code> and simply return our Hasselhoff photo.</p>
```objc
‑ (void)startLoading
{
    id &lt;NSURLProtocolClient&gt; client = self.client;
    NSURLRequest *request = self.request;

    NSDictionary *headers = @{ @"Content-Type": @"image/jpeg" };
    NSData *imageData = UIImageJPEGRepresentation([UIImage imageNamed:@"David_Hasselhoff.jpeg"], 1.0);

    NSHTTPURLResponse *response = [[NSHTTPURLResponse alloc] initWithURL:request.URL
                                                              statusCode:200
                                                             HTTPVersion:@"HTTP/1.1"
                                                            headerFields:headers];

    [client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
    [client URLProtocol:self didLoadData:imageData];
    [client URLProtocolDidFinishLoading:self];
}

‑ (void)stopLoading
{
    // We send all the data at once, so there is nothing to do here.
}
```
<p>The only remaining piece of the puzzle here is the registration step. We’ll simply register it at the app’s launch.</p>
```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    [NSURLProtocol registerClass:[TWTHasselhoffImageProtocol class]];
    return YES;
}
```
<p>Now when we run the application, instead of seeing a cute kitten we should see Hasselhoff. The idea that you don’t have to change your client code should be clear. We are still requesting the cute kitten image, but our protocol knows better. It knows what we really wanted was the Hoff.</p>
<h2>Summary</h2>
<p>Foundation’s URL loading system is a powerful abstraction around network requests. It allows you to easily add your own handlers to provide support for new protocols or change existing behavior. This can help a lot when you are missing resources and need to stay productive.</p>
<p><a href="https://github.com/twotoasters/URLMock">URLMock</a> is a great example of the things you can do by exploiting <code>NSURLProtocol</code>. It provides a lightweight interface to intercept requests and respond however you see fit. Take some time to look through <a href="https://github.com/twotoasters/URLMock/blob/master/URLMock/Mock%20URL%20Protocol/UMKMockURLProtocol.m"><code>UMKMockURLProtocol</code></a> to explore a more detailed example of an <code>NSURLProtocol</code> subclass.</p>
<p>If you have any questions about this article or <code>NSURLProtocol</code>, get in touch with me on Twitter at <a href="http://twitter.com/jnjosh">@jnjosh</a>.</p>
