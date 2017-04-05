---
layout: post
status: publish
published: true
title: Interactive Transitions
author:
  display_name: Andrew Hershberger
  login: andrew
  email: andrew@twotoasters.com
  url: ''
author_login: andrew
author_email: andrew@twotoasters.com
wordpress_id: 107
wordpress_url: http://objectivetoast.com/?p=107
date: '2014-04-14 13:41:13 -0400'
date_gmt: '2014-04-14 17:41:13 -0400'
categories:
- User Interface
tags:
- UIViewController
- transitions
- animations
- interactive
- UIPercentDrivenInteractiveTransition
---
<p>In my <a href="http://objectivetoast.com/2014/03/17/custom-transitions-on-ios/">previous post</a>, I explained how to create custom animated transitions. This time, I'm going to show how to make interactive transitions.</p>
<p>As a quick review from last time, there are three main roles involved in an animated transition: the from and to view controllers and the animation controller. In a non-interactive transition, the animation controller defines a duration and sets up the animations between the two views. Both views are placed inside of a container view during the transition.</p>
<p>Interactive transitions build on this structure by adding a fourth role: the interaction controller. This new role is played by an object that conforms to the <code>UIViewControllerInteractiveTransitioning</code> protocol.</p>
<p><!--more--></p>
<p>Apple provides an interaction controller you can use directly called <code>UIPercentDrivenInteractiveTransition</code>. One way to understand what it does is to imagine that the animation controller sets up an animation timeline, and then <code>UIPercentDrivenInteractiveTransition</code> scrubs the playhead back and forth along the timeline. It's up to you to decide how user input should map to the percent-driven interaction controller's <code>percentComplete</code> property, which determines the position of the metaphorical playhead.</p>
<p>This example only covers how to use <code>UIPercentDrivenInteractiveTransition</code>, but you can also implement your own interaction controllers if needed.</p>
<p>Interactive transitions are available in all the same cases as custom animated transitions: navigation controller push &amp; pop, changing tabs in a tab bar controller, and modal presentations. Since the same concepts apply to all of these cases, I am only going to present a single example: a navigation controller pop transition with a pinch gesture to control a scale and fade animation.</p>
<h2>Implementing the Interactive Transition</h2>
<p>There are several phases to this interactive transition:</p>
<ol>
<li>Detect that user input has begun.</li>
<li>Start the transition.</li>
<li>Return the animation controller from the navigation controller's delegate.</li>
<li>Return the interaction controller from the navigation controller's delegate.</li>
<li>Update the interaction controller upon subsequent user input.</li>
<li>Tell the interaction controller to finish or cancel the transition when user input stops.</li>
</ol>
<h3>Detect User Input</h3>
<p>Most commonly, the beginning of user input is the start of some kind of continuous touch gesture. I added a <code>UIPinchGestureRecognizer</code> instance to my view controller's view. I will discuss this implementation pattern a little later, but for now, all you need know is that <code>TWTPopTransitionController</code> manages the gesture recognizer logic and also plays the roles of the animation controller and interaction controller.</p>
<p>In TWTPopTransitionController.m:</p>
<pre><code>#!objc
- (id)init
{
    self = [super init];
    if (self) {
        _pinchGestureRecognizer = [[UIPinchGestureRecognizer alloc] init];
        [_pinchGestureRecognizer addTarget:self action:@selector(handlePinchGesture:)];
    }
    return self;
}
</code></pre>
<p>In TWTViewController.m:</p>
<pre><code>#!objc
- (void)viewDidLoad
{
    [super viewDidLoad];

    self.view.backgroundColor = [UIColor twt_nextColor];

    if (self.popTransitionController.pinchGestureRecognizer) {
        [self.view addGestureRecognizer:self.popTransitionController.pinchGestureRecognizer];
    }
}
</code></pre>
<h3>Start the Transition</h3>
<p>As soon as the interaction begins, you need to trigger the view controller transition. I set up my view controller to call <code>‑popViewControllerAnimated:</code> when the pinch gesture recognizer's state changes to <code>UIGestureRecognizerStateBegan</code>.</p>
<p>In TWTPopTransitionController.m:</p>
<pre><code>#!objc
- (void)handlePinchGesture:(UIPinchGestureRecognizer *)pinchGestureRecognizer
{
    …
    switch (pinchGestureRecognizer.state) {
        case UIGestureRecognizerStateBegan:
        {
            self.interactive = YES;
            [self.delegate transitionControllerInteractionDidStart:self];
            break;
        }
        …
    }
}
</code></pre>
<p>In TWTViewController.m:</p>
<pre><code>#!objc
- (void)transitionControllerInteractionDidStart:(id&lt;TWTTransitionController&gt;)transitionController
{
    [self.navigationController popViewControllerAnimated:YES];
}
</code></pre>
<h3>Return the Animation Controller</h3>
<p>When the transition starts, the navigation controller asks its delegate for an animation controller. This is required. Otherwise, the navigation controller will not ask for an interaction controller. I set up my navigation controller delegate to return an animation controller that scales down and fades out the from view controller's view.</p>
<p>One important caveat here is that if you are using <code>UIPercentDrivenInteractiveTransition</code> for your interaction controller, your animation controller must use the <code>UIView</code> block-based animation API to set up the animation timeline (see WWDC 2013 session 218 slide 151). Animations you set up with Core Animation directly or even with the <code>+[UIView transitionFromView:​toView:​duration:​options:​completion:]</code> method will not be managed by the <code>UIPercentDrivenInteractiveTransition</code>.</p>
<p>Another point to keep in mind is that the animation completion blocks do not get called until the end of the interactive portion of the transition, so you cannot chain animation blocks.</p>
<h3>Return the Interaction Controller</h3>
<p>Next, the navigation controller will ask its delegate for an interaction controller. Take care to only return an interaction controller if the transition will actually be interactive. In this example, an interactive pop transition can be triggered with a pinch gesture or a non-interactive one can be triggered by tapping the back button. At one point during development, I accidentally returned the interaction controller in both cases, and in the non-interactive case, there was nothing in place to ever update the interaction controller's <code>percentComplete</code>. This left the app frozen in a state that did not allow any user interaction.</p>
<h3>Update the Interaction Controller</h3>
<p>After the animation and interaction controllers are supplied to the navigation controller, it's off to the races. The animation controller's <code>‑animateTransition:</code> method will set up the animation timeline, and the pinch gesture recognizer will continue to report updates with state <code>UIGestureRecognizerStateChanged</code>. I set up the pinch gesture recognizer's handler so that as the user input changes it updates the interaction controller's <code>percentComplete</code> property. The interaction controller takes the percent complete value and scrubs to the appropriate point along the animation timeline. The percent complete does not have to change in a single direction; it can increase or decrease at each update.</p>
<p>In TWTPopTransitionController.m:</p>
<pre><code>#!objc
- (void)handlePinchGesture:(UIPinchGestureRecognizer *)pinchGestureRecognizer
{
    CGFloat scale = pinchGestureRecognizer.scale;
    CGFloat velocity = pinchGestureRecognizer.velocity;

    switch (pinchGestureRecognizer.state) {
        …
        case UIGestureRecognizerStateChanged:
        {
            CGFloat percentComplete = 1.0 - scale;
            [self updateInteractiveTransition:percentComplete];
            break;
        }
        …
    }
}
</code></pre>
<h3>Complete the Transition</h3>
<p>Eventually, the pinch gesture will report that its state has changed to either <code>UIGestureRecognizerStateEnded</code> or <code>UIGestureRecognizerStateCancelled</code>. I made the handler determine whether the gesture is progressing in a way that indicates that the user wants to complete the transition or cancel the transition, and I call the appropriate method on the interaction controller. The interaction controller then animates the views from their current positions on the animation timeline to either the end (in the case of completing the transition) or back to the beginning (in the case of canceling the transition). There is a <a href="http://openradar.appspot.com/14675246">bug</a> in the iOS Simulator that causes these final animations to play twice. Until Apple resolves this issue, you will need to test on the device to see how it is supposed to work.</p>
<p>One other unexpected behavior that I encountered was that when a transition is canceled, the properties of the views that you animate are returned to their pre-animation state before the completion block is called. This surprised me since it is the only case that I'm aware of in which the block-based animation API changes the <a href="https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html#//apple_ref/doc/uid/TP40004514-CH2-SW19">model layer</a> in ways other than adding animation objects.</p>
<p>You may be wondering, as I did, how <code>UIPercentDrivenInteractiveTransition</code> is able to scrub through an animation. Through some careful inspection with the debugger, you can see that the basic idea is that it is setting the <code>speed</code> of the container view's layer to 0 and then, as the percent complete changes, it adjusts the layer's <code>timeOffset</code> to <code>(animationDuration * percentComplete)</code> which effectively scrubs along the animation timeline.</p>
<p>I recently presented the results of my interactive transition explorations to the rest of the iOS team at Two Toasters, and another good question that was raised was "What is the purpose of the animation duration returned by the animation controller if the transition is interactive?" The answer is that it comes in to play during the animated portion of the transition after interaction is complete. If the animation duration is 1.0 second and you finish the transition when <code>percentComplete</code> is 20%, then the completion animation will last 1.0s × (100% − 20%) = 0.8s. Similarly, if instead you cancel the transition, the animation will last 0.2s.</p>
<h2>Working Toward a Reusable Implementation</h2>
<p>I like to organize code in a way that allows for easy and sensible reusability. It has been a challenge to find a good pattern for reusability when it comes to interactive transitions. The parts of this example that seemed the most valuable to reuse were the animation controller's <code>‑animateTransition:</code> method and the pinch gesture recognizer's event handling logic.</p>
<p>Initially, after trying several approaches that were less than satisfactory, I came to a design in which the animation controller was a stand-alone <code>NSObject</code> subclass and the gesture recognizer and interaction controller were managed by what I called a transition coordinator. In theory, this pattern allowed mixing and matching interaction logic with animation styles. It worked, but it was still more complicated that I thought was necessary, especially since it seemed likely that the interaction logic would usually need to be tailored to a specific animation.</p>
<p>For that reason, I tried another approach in which the animation controller, interaction controller, and interaction logic are all implemented by a single object that I named the transition controller. This approach significantly simplified the design and made the most sense of the various approaches that I tried.</p>
<p>You can find the <a href="https://github.com/ObjectiveToast/InteractiveTransitions">full example project</a> including my final implementation pattern on GitHub.</p>
<p>If you have comments on the article or examples of creative interactive transitions you've built (for example, using something other than a touch gesture as an input source) I'd love to hear about it. Get in touch on Twitter: <a href="https://twitter.com/a_hershberger">@a_hershberger</a>.</p>
