---
layout: post
status: publish
published: true
title: Introducing URLMock
author:
  display_name: Prachi Gauriar
  login: prachi
  email: prachi@twotoasters.com
  url: http://twitter.com/prachigauriar/
author_login: prachi
author_email: prachi@twotoasters.com
author_url: http://twitter.com/prachigauriar/
wordpress_id: 121
wordpress_url: http://objectivetoast.com/?p=121
date: '2014-04-28 08:00:17 -0400'
date_gmt: '2014-04-28 12:00:17 -0400'
categories:
- Testing
- Tools
tags:
- open-source
- URLMock
- NSURL
- mocking
- stubbing
- request
- response
- framework
---
<p><!--<br />
  Author: Prachi Gauriar<br />
  Categories: Tools, Testing<br />
  Tags: URLMock, NSURL, mocking, stubbing, request, response, framework, open source<br />
  Keywords:<br />
--></p>
<p>Testing networking code is hard. The web service you’re connecting to can fail in myriad ways, from basic networking errors to malformed responses. All but the simplest APIs return a wide variety of data that your app has to handle correctly. Sometimes APIs can be so complex that it’s hard to verify that you’re calling things in the right order, much less sending appropriate parameters and handling responses correctly.</p>
<p><!--more--></p>
<p>There aren’t a lot of good solutions out there. The best seems to be to write a special test server that responds to API requests with canned responses so you can test your code. Because writing the app <em>and</em> the test server is high on your list of things to do. Realistically, you’ll probably do what most developers do: forgo thorough testing and hope for the best. Let’s all agree that this isn’t the best strategy.</p>
<p>A better strategy would be to use a native library to transparently intercept network requests inside your app and respond with data or errors for testing. You could then verify that you sent the correct requests and handled all the various responses and error conditions correctly. Since the library is native, you could use it for either manual or automated testing environments. If only that library existed…</p>
<p><a href="http://github.com/twotoasters/URLMock" title="URLMock">URLMock</a> is an open-source Cocoa framework for mocking and stubbing URL requests and responses, brought to you by the fine folks at <a href="http://twotoasters.com" title="Two Toasters">Two Toasters</a>. It works seamlessly with apps that use the <code>NSURL</code> loading system, e.g., <code>NSURLConnection</code> or <a href="http://github.com/afnetworking/afnetworking" title="AFNetworking">AFNetworking</a>. You don’t even have to change your client code. Just tell URLMock how to respond to a URL request, and it takes care of the rest.</p>
<p>In this post, we’re going to show you how you can use URLMock in your networking code’s automated tests. We’re going to focus on HTTP, but URLMock can be used with other protocols and for manual testing as well. Check out the <a href="https://github.com/twotoasters/URLMock/blob/master/README.md" title="URLMock README">readme</a> for more details.</p>
<h2>Core Concepts</h2>
<p>URLMock revolves around three basic types of objects:</p>
<ul>
<li><em>Mock requests</em> describe the URL requests that URLMock should intercept. Mock HTTP requests are represented by <code>UMKMockHTTPRequest</code> objects, which intercept requests that match a given URL, HTTP method, and request body. Parameters can be checked as well.</li>
<li><em>Mock responders</em> respond to intercepted URL requests. <code>UMKMockHTTPResponder</code> instances can respond with data or an <code>NSError</code>.</li>
<li>Finally, <code>UMKMockURLProtocol</code> is the primary interface to the URLMock system. After registering mock requests with it, it intercepts your app’s URL requests and responds to them as you specified. It also does the bookkeeping so that you can check if any mock requests were unserviced or any unexpected requests were received.</li>
</ul>
<h2>Unit Testing with URLMock</h2>
<p>We’re going to write tests for a simple <a href="http://openweathermap.org/API" title="Open Weather Map API">OpenWeatherMap API</a> client. The full example project is <a href="http://github.com/objectivetoast/URLMockDemo" title="Example Repo">available on GitHub</a>.</p>
<p>Also, URLMock includes a lot of test helpers to make testing easier. We’ll be using them liberally throughout the example, so take a look at <a href="https://github.com/twotoasters/URLMock/blob/master/URLMock/Utilities/UMKTestUtilities.h" title="UMKTestUtilities.h">the header</a> to see what the different <code>UMK*</code> functions do.</p>
<p>The code we’ll be testing is <code>‑[TWTOpenWeatherMapAPIClient fetchTemperatureForLatitude:​longitude:​success:​failure:]</code>, which fetches the current temperature for the specified location.</p>
<pre><code>#!objc
- (NSOperation *)fetchTemperatureForLatitude:(NSNumber *)latitude
                                   longitude:(NSNumber *)longitude
                                     success:(void (^)(NSNumber *))successBlock
                                     failure:(void (^)(NSError *))failureBlock
{
    NSParameterAssert(latitude &amp;&amp; ABS(latitude.doubleValue) &lt;= 90.0);
    NSParameterAssert(longitude &amp;&amp; ABS(longitude.doubleValue) &lt;= 180.0);

    return [self.operationManager GET:@"weather"
                           parameters:@{ @"lat" : latitude, @"lon" : longitude }
                              success:^(AFHTTPRequestOperation *operation, id response) {
                                  if (successBlock) {
                                      successBlock([response valueForKeyPath:@"main.temp"]);
                                  }    
                              } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
                                  if (failureBlock) {
                                      failureBlock(error);
                                  }
                              }];
}
</code></pre>
<p>Keen-eyed readers may notice some potential bugs in this implementation. Let’s see how it’s tested in <code>TWTOpenWeatherMapAPIClientTests.m</code>:</p>
<pre><code>#!objc
- (void)testFetchTemperatureForLatitudeLongitude
{
    __block NSNumber *temperature = nil;
    [self.APIClient fetchTemperatureForLatitude:@(35.99)
                                      longitude:@(-78.9)
                                        success:^(NSNumber *kelvins) {
                                            temperature = kelvins;
                                        }
                                        failure:nil];

    // Assert that temperature != nil before 2.0s elapse
    UMKAssertTrueBeforeTimeout(2.0, temperature != nil, @"temperature isn't set in time");
}
</code></pre>
<p>This is a pretty bad test. It checks that an object is returned, but doesn’t check if the API client gets the right value out of the API response. It also doesn’t test error conditions, like what happens when the API request fails or the response is malformed. Let’s see if URLMock can help.</p>
<h3>Enabling URLMock</h3>
<p>Before updating our tests, we need to tell URLMock to intercept URL loading requests. We do this using <code>+[UMKMockURLProtocol enable]</code>. We should also enable verification, which allows us to check if we received any unexpected requests or didn’t receive any expected ones. We can do these both in our <code>XCTestCase</code>’s <code>+setUp</code> method:</p>
<pre><code>#!objc
+ (void)setUp
{
    [super setUp];
    [UMKMockURLProtocol enable];
    [UMKMockURLProtocol setVerificationEnabled:YES];
}
</code></pre>
<p>When we’re done testing, we should undo what we did in <code>+setUp</code> so we don’t inadvertently interfere with our other testing code. The ideal place for this is in <code>+tearDown</code>.</p>
<pre><code>#!objc
+ (void)tearDown
{
    [UMKMockURLProtocol setVerificationEnabled:NO];
    [UMKMockURLProtocol disable];
    [super tearDown];
}
</code></pre>
<p>Finally, before we run our individual tests, we want URLMock to be in a clean state. We can accomplish this by invoking <code>+[UMKMockURLProtocol reset]</code> in our <code>‑setUp</code> method.</p>
<pre><code>#!objc
- (void)setUp
{
    [super setUp];
    [UMKMockURLProtocol reset];
    …
}
</code></pre>
<p>Now that we’re set up, let’s actually write some tests. We’ll write one that tests success with good data, one that tests failure due to a network error, and one that tests failure due to a malformed JSON response.</p>
<h3>A Better Test for Success</h3>
<p>To ensure that <code>‑fetchTemperatureForLatitude:​longitude:​success:​failure:</code> succeeds with good data, we first need to test that it accesses the correct API endpoint with the correct parameters:</p>
<pre><code>#!objc
- (void)testFetchTemperatureForLatitudeLongitudeCorrectData
{
    NSNumber *latitude = @12.34;
    NSNumber *longitude = @-45.67;
    NSNumber *temperature = @289.82;

    NSURL *temperatureURL = [self temperatureURLWithLatitude:latitude longitude:longitude];
</code></pre>
<p>Here we get the URL from a helper method in our test class. It’s important to note that we don’t ask the API client for the URL to use. If we were in the habit of trusting the API client author to always do things correctly, we wouldn’t be writing unit tests.</p>
<p>Now we need to create a mock request to match the URL request that the API client’s method will access. The request should be an HTTP GET and have no HTTP body. We can create a mock request like so:</p>
<pre><code>#!objc
    UMKMockHTTPRequest *mockRequest = [UMKMockHTTPRequest mockHTTPGetRequestWithURL:temperatureURL];
</code></pre>
<p>Next, let’s tell URLMock how to respond when it sees this request. HTTP servers typically use a status code of 200 to indicate success, so our response should do that too. Also, the body of the response should be JSON that looks like <code>{ "main": { "temp": «Temperature» } }</code>. We can mock this up really easily:</p>
<pre><code>#!objc
    UMKMockHTTPResponder *mockResponder = [UMKMockHTTPResponder mockHTTPResponderWithStatusCode:200];
    [mockResponder setBodyWithJSONObject:@{ @"main" : @{ @"temp" : temperature } }];
    mockRequest.responder = mockResponder;
</code></pre>
<p>That last line is really important. It tells the mock request which responder to use when it’s serviced by URLMock. Speaking of which, we need to register our mock request with <code>UMKMockURLProtocol</code>.</p>
<pre><code>#!objc
    [UMKMockURLProtocol expectMockRequest:mockRequest];
</code></pre>
<p>We’re done registering our mock request and responder. While that wasn’t hard, it took more code than I’d like. URLMock includes some helper methods to make this easier. Using one of those, we can reduce those previous lines to a single method invocation:</p>
<pre><code>#!objc
    [UMKMockURLProtocol expectMockHTTPGetRequestWithURL:temperatureURL
                                     responseStatusCode:200
                                           responseJSON:@{ @"main" : @{ @"temp" : temperature } }];
</code></pre>
<p>That’s much better. So, we’ve prepped URLMock to expect an HTTP GET request for the endpoint URL and respond with the JSON body above. Now we need to actually invoke the API client method and verify that it behaves correctly.</p>
<pre><code>#!objc
    __block BOOL succeeded = NO;
    __block BOOL failed = NO;
    __block NSNumber *kelvins = nil;
    [self.APIClient fetchTemperatureForLatitude:latitude
                                      longitude:longitude
                                        success:^(NSNumber *temperatureInKelvins) {
                                            succeeded = YES;
                                            kelvins = temperatureInKelvins;
                                        }
                                        failure:^(NSError *error) {
                                            failed = YES;
                                        }];

    UMKAssertTrueBeforeTimeout(1.0, succeeded, @"success block is not called");
    UMKAssertTrueBeforeTimeout(1.0, !failed, @"failure block is called");
    UMKAssertTrueBeforeTimeout(1.0, [kelvins isEqualToNumber:temperature], @"incorrect temperature");
</code></pre>
<p>This code fetches the temperature and saves away the response. We make sure that the success block is called, the failure block isn’t, and that the temperature passed to the success block is the one we sent using our mock responder.</p>
<p>As a final step, we should make sure that no additional URL requests get made. To do this, we use <code>+[UMKMockURLProtocol verifyWithError:]</code>:</p>
<pre><code>#!objc
    NSError *verificationError = nil;
    XCTAssertTrue([UMKMockURLProtocol verifyWithError:&amp;verificationError], @"verification failed");
}
</code></pre>
<p>And that’s it. If you run this test, it should pass. Next, let’s test some error conditions.</p>
<h3>Responding with Errors</h3>
<p>Responding to requests with an error object is trivial:</p>
<pre><code>#!objc
- (void)testFetchTemperatureForLatitudeLongitudeError
{
    …
    NSURL *temperatureURL = [self temperatureURLWithLatitude:latitude longitude:longitude];
    [UMKMockURLProtocol expectMockHTTPGetRequestWithURL:temperatureURL responseError:[self randomError]];
    …
}
</code></pre>
<p>This code should be pretty self-explanatory. We generate a random error using a test case helper method and tell URLMock to respond with that. To verify that the API client handles it properly, we do the following:</p>
<pre><code>#!objc
    __block BOOL succeeded = NO;
    __block BOOL failed = NO;
    [self.APIClient fetchTemperatureForLatitude:latitude
                                      longitude:longitude
                                        success:^(NSNumber *temperature) {
                                            succeeded = YES;
                                        }
                                        failure:^(NSError *error) {
                                            failed = YES;
                                        }];

    UMKAssertTrueBeforeTimeout(1.0, !succeeded, @"success block is called");
    UMKAssertTrueBeforeTimeout(1.0, failed, @"failure block is not called");

    NSError *verificationError = nil;
    XCTAssertTrue([UMKMockURLProtocol verifyWithError:&amp;verificationError], @"verification failed");
</code></pre>
<p>This is a lot like the success case, but we make sure that the failure block is called and that the success case isn’t. And that’s it. If we run this test, it should pass too.</p>
<p>For our final test, let’s respond to the request with malformed data.</p>
<h3>Malformed Data, Malformed Method</h3>
<p>Our test for malformed data is a blend between the previous two. We register a mock response that responds with some random JSON.</p>
<pre><code>#!objc
- (void)testFetchTemperatureForLatitudeLongitudeMalformedData
{
    …
    NSURL *temperatureURL = [self temperatureURLWithLatitude:latitude longitude:longitude];
    [UMKMockURLProtocol expectMockHTTPGetRequestWithURL:temperatureURL
                                     responseStatusCode:200
                                           responseJSON:UMKRandomJSONObject(3, 3)];
    …
</code></pre>
<p>We then test that it fails as before:</p>
<pre><code>#!objc
    __block BOOL succeeded = NO;
    __block BOOL failed = NO;
    [self.APIClient fetchTemperatureForLatitude:latitude
                                      longitude:longitude
                                        success:^(NSNumber *temperature) {
                                            succeeded = YES;
                                        }
                                        failure:^(NSError *error) {
                                            failed = YES;
                                        }];

    UMKAssertTrueBeforeTimeout(1.0, !succeeded, @"success block is called");
    UMKAssertTrueBeforeTimeout(1.0, failed, @"failure block is not called");
}
</code></pre>
<p>If you run this test, there’s a high likelihood the API client method will crash. The API client code contains the line <code>[response valueForKeyPath:@"main.temp"]</code>, but it doesn’t validate that <code>response</code> is a dictionary that contains the required keys. If it’s not, the method could crash in <code>‑valueForKeyPath:</code>. Fixing this bug and getting the test to pass is left as an exercise for the reader.</p>
<h2>Conclusion</h2>
<p>That’s just a taste of what you can do with <a href="http://github.com/twotoasters/URLMock" title="URLMock">URLMock</a>. We’ve used it on a few client projects over the last few months and uncovered some pretty significant bugs in our error handling code. We’ve got a major release planned soon that will hopefully improve it and make it even more flexible. Until then, install it, play with it, and consider using it on your next project!</p>
