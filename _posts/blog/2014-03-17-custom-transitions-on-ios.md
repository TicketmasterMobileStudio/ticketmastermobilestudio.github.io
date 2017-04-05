---
layout: post
status: publish
published: true
title: Custom Transitions on iOS
author:
  display_name: Andrew Hershberger
  login: andrew
  email: andrew@twotoasters.com
  url: ''
author_login: andrew
author_email: andrew@twotoasters.com
wordpress_id: 68
wordpress_url: http://objectivetoast.com/?p=68
date: '2014-03-17 21:50:04 -0400'
date_gmt: '2014-03-18 01:50:04 -0400'
categories:
- User Interface
tags:
- UIViewController
- transitions
- animations
---
<p><!--<br />
    Author: Andrew Hershberger<br />
    Categories: User Interface<br />
    Tags: UIViewController, transitions, animations<br />
--></p>
<p><em>This article is the first in a series that shows how to customize view controller transitions on iOS. In part one, the focus is on creating custom animated (non-interactive) transitions.</em></p>
<p>When using typical iOS apps we navigate from screen to screen frequently. Previously, if you wanted to do anything other than the standard transition animations, you were on your own, but in iOS 7 Apple has provided a new API to let us customize these animations.</p>
<p><!--more--></p>
<p>iOS provides several built-in types of transitions. Navigation controllers push and pop to navigate through an information hierarchy, tab bar controllers switch between sections by changing tabs, and any view controller can present and dismiss another view controller modally for specific tasks.</p>
<h2>API Introduction</h2>
<p>Every custom transition involves three main objects:</p>
<ul>
<li>The from view controller (the one that's going away)</li>
<li>The to view controller (the one that's appearing)</li>
<li>An animation controller</li>
</ul>
<p>Starting a custom transition works the same way as before the transition was customized. For push and pop, it means calling one of the push-, pop-, or set- methods on <code>UINavigationController</code> to modify its stack of view controllers. For changing tabs, it means setting the <code>selectedIndex</code> or <code>selectedViewController</code> property on a <code>UITabBarController</code>. For a modal transition, it means calling <code>‑[UIViewController presentViewController:animated:completion:]</code> or <code>‑[UIViewController dismissViewControllerAnimated:completion:]</code>. In each case, this step determines which view controller will be the "from view controller" and which one will be the "to view controller."</p>
<p>To use a custom transition, you need to say which object should play the role of the animation controller. For me, this was the most confusing part of setting up a custom animated transition because each type of transition looks for the animation controller in a different place. The following table shows how to provide the animation controller for each case. Just remember that you always return the animation controller from a delegate method.</p>
<table style="border: 1px solid #aaa; border-collapse: collapse;">
<tr>
<td style="min-width:8em; border: 1px solid #aaa; border-collapse: collapse; padding: 2px 5px;">
      Transition
    </td>
<td style="border: 1px solid #aaa; border-collapse: collapse; padding: 2px 5px;">
      Animation controller provided by
    </td>
</tr>
<tr>
<td style="border: 1px solid #aaa; border-collapse: collapse; padding: 2px 5px;">
      Push and Pop
    </td>
<td style="border: 1px solid #aaa; border-collapse: collapse; padding: 2px 5px;">
      The navigation controller's <code>delegate</code> implements <code>‑navigationController:animationControllerForOperation:fromViewController:toViewController:</code>
    </td>
</tr>
<tr>
<td style="border: 1px solid #aaa; border-collapse: collapse; padding: 2px 5px;">
      Changing Tabs
    </td>
<td style="border: 1px solid #aaa; border-collapse: collapse; padding: 2px 5px;">
      The tab bar controller's <code>delegate</code> implements <code>‑tabBarController:animationControllerForTransitionFromViewController:toViewController:</code>
    </td>
</tr>
<tr>
<td style="border: 1px solid #aaa; border-collapse: collapse; padding: 2px 5px;">
      Present
    </td>
<td style="border: 1px solid #aaa; border-collapse: collapse; padding: 2px 5px;">
      The presented view controller's <code>transitioningDelegate</code> implements <code>‑animationControllerForPresentedController:presentingController:sourceController:</code>. The presented view controller's <code>‑modalPresentationStyle</code> must be set to <code>UIModalPresentationCustom</code> before the transition is started.
    </td>
</tr>
<tr>
<td style="border: 1px solid #aaa; border-collapse: collapse; padding: 2px 5px;">
      Dismiss
    </td>
<td style="border: 1px solid #aaa; border-collapse: collapse; padding: 2px 5px;">
      The presented view controller's <code>transitioningDelegate</code> implements <code>‑animationControllerForDismissedController:</code> The presented view controller's <code>‑modalPresentationStyle</code> must be set to <code>UIModalPresentationCustom</code> before the transition is started.
    </td>
</tr>
</table>
<p>The animation controller can be any object that conforms to the <a href="https://developer.apple.com/library/ios/documentation/uikit/reference/UIViewControllerAnimatedTransitioning_Protocol/Reference/Reference.html"><code>UIViewControllerAnimatedTransitioning</code> protocol</a>. The protocol defines two required methods. One provides the animation duration, and the other performs the animation. Each of these methods is passed a context when it is called. The context provides access to the information and objects you need to create your custom transition. Here are some of the highlights:</p>
<ul>
<li>The from view controller</li>
<li>The to view controller</li>
<li>The initial and final frames for the from and to view controllers' views</li>
<li>The container view, which, according to the <a href="https://developer.apple.com/library/ios/documentation/uikit/reference/UIViewControllerContextTransitioning_protocol/Reference/Reference.html#//apple_ref/occ/intfm/UIViewControllerContextTransitioning/containerView">docs</a>, "acts as the superview for the views involved in the transition."</li>
</ul>
<p><strong>Important:</strong> The context also implements <code>‑completeTransition:</code> which you must call once your custom transition is finished.</p>
<p>That is everything you need to know to make a custom transition. Let's try some examples!</p>
<h2>Examples</h2>
<p>All of these examples are <a href="https://github.com/ObjectiveToast/CustomTransitions">available on GitHub</a>, so clone the repo and experiment with it as you follow along.</p>
<p>All three examples use <code>TWTExampleViewController</code> either directly or by subclassing. All it does on its own is set the background color for its view and make it so that you can tap anywhere inside of it to end the example and go back to the main menu.</p>
<h3>Push and pop with flips</h3>
<p>In this example, the goal is to make push and pop use flip animations instead of the standard slide animation. I started by setting up a navigation controller with an instance of <code>TWTPushExampleViewController</code> as the root. <code>TWTPushExampleViewController</code> adds a button to the right side of the navigation bar titled "Push". When it is tapped, a new instance of <code>TWTPushExampleViewController</code> is pushed on to the navigation stack:</p>
```objc
- (void)pushButtonTapped
{
    TWTPushExampleViewController *viewController = [[TWTPushExampleViewController alloc] init];
    viewController.delegate = self.delegate;
    [self.navigationController pushViewController:viewController animated:YES];
}
```
<p>The setup for the navigation controller happens in <code>TWTExamplesListViewController</code> (the view controller that runs the main menu for the demo app). Notice that it sets itself as the navigation controller's delegate:</p>
```objc
- (void)presentPushExample
{
    TWTPushExampleViewController *viewController = [[TWTPushExampleViewController alloc] init];
    viewController.delegate = self;

    UINavigationController *navigationController = [[UINavigationController alloc] initWithRootViewController:viewController];
    navigationController.delegate = self;

    [self presentViewController:navigationController animated:YES completion:nil];
}
```
<p>This means that when a navigation controller transition is about to start, <code>TWTExamplesListViewController</code> will receive the delegate message and have an opportunity to return an animation controller. For this transition, I am using an instance of <code>TWTSimpleAnimationController</code> which is just a wrapper around <code>+[UIView transitionFromView:toView:duration:options:completion:]</code>:</p>
```objc
- (id&lt;UIViewControllerAnimatedTransitioning&gt;)navigationController:(UINavigationController *)navigationController
                                  animationControllerForOperation:(UINavigationControllerOperation)operation
                                               fromViewController:(UIViewController *)fromVC
                                                 toViewController:(UIViewController *)toVC
{
    TWTSimpleAnimationController *animationController = [[TWTSimpleAnimationController alloc] init];
    animationController.duration = 0.5;
    animationController.options = (  operation == UINavigationControllerOperationPush
                                   ? UIViewAnimationOptionTransitionFlipFromRight
                                   : UIViewAnimationOptionTransitionFlipFromLeft);
    return animationController;
}
```
<p>If the transition is a push, I use a flip from the right, otherwise, I use a flip from the left.</p>
<p>Here's what the implementation of <code>TWTSimpleAnimationController</code> looks like:</p>
```objc
- (NSTimeInterval)transitionDuration:(id&lt;UIViewControllerContextTransitioning&gt;)transitionContext
{
    return self.duration;
}


- (void)animateTransition:(id&lt;UIViewControllerContextTransitioning&gt;)transitionContext
{
    UIViewController *fromViewController = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
    UIViewController *toViewController = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
    toViewController.view.frame = [transitionContext finalFrameForViewController:toViewController];
    [toViewController.view layoutIfNeeded];

    [UIView transitionFromView:fromViewController.view
                        toView:toViewController.view
                      duration:self.duration
                       options:self.options
                    completion:^(BOOL finished) {
                        [transitionContext completeTransition:YES];
                    }];
}
```
<p>Remember, these two methods are part of the <code>UIViewControllerAnimatedTransitioning</code> protocol. They're called by UIKit when it's time for the animation controller to run the custom transition.</p>
<p>Here are a couple things to notice about <code>‑animateTransition:</code>:</p>
<ul>
<li>The from view controller, to view controller, and final frame for the to view controller's view are all extracted from the transition context. There are other pieces of information that are available, but in this case, not all of them were needed.</li>
<li><code>+[UIView transitionFromView:toView:duration:options:completion:]</code> takes care of adding and removing the view controllers' views to the view hierarchy. In a later example, I will show a case in which it is done manually.</li>
<li>In the transition's completion block, I call <code>[transitionContext completeTransition:YES];</code> to tell the system that the transition is over. If you forget to do this, you will not be able to interact with the app. If that happens, check this first.</li>
</ul>
<p>That's all there is to it! Here are some ideas for things to try:</p>
<ul>
<li>Change the animation duration and notice how it affects the animation in the navigation bar.</li>
<li>Change the animation options from flips to page curls.</li>
<li>Find a way to let each view controller in the navigation stack specify its own animation controller. See the recommended patterns section at the end of the article for a suggested approach.</li>
</ul>
<h3>Change tabs using a cross dissolve</h3>
<p>This example should be familiar. It uses the same ideas as before, but with a tab bar controller instead of a navigation controller.</p>
<p>Here's the setup in <code>TWTExamplesListViewController</code>:</p>
```objc
- (void)presentTabsExample
{
    NSMutableArray *viewControllers = [[NSMutableArray alloc] init];

    for (NSUInteger i=0; i&lt;3; i++) {
        TWTChangingTabsExampleViewController *viewController = [[TWTChangingTabsExampleViewController alloc] init];
        viewController.delegate = self;
        viewController.index = i;
        [viewControllers addObject:viewController];
    }

    UITabBarController *tabBarController = [[UITabBarController alloc] init];
    [tabBarController setViewControllers:viewControllers animated:NO];
    tabBarController.delegate = self;

    [self presentViewController:tabBarController animated:YES completion:nil];
}
```
<p>The <code>index</code> in the <code>TWTChangingTabsExampleViewController</code> is just a way to customize the tab for each view controller. Just like before, the <code>TWTExamplesListViewController</code> is the delegate and will provide the animation controller when it is time to change tabs:</p>
```objc
- (id&lt;UIViewControllerAnimatedTransitioning&gt;)tabBarController:(UITabBarController *)tabBarController
           animationControllerForTransitionFromViewController:(UIViewController *)fromVC
                                             toViewController:(UIViewController *)toVC
{
    TWTSimpleAnimationController *animationController = [[TWTSimpleAnimationController alloc] init];
    animationController.duration = 0.5;
    animationController.options = UIViewAnimationOptionTransitionCrossDissolve;
    return animationController;
}
```
<p>Look familiar? That's all there is to it.</p>
<h3>Present an Overlay</h3>
<p>One of my favorite uses of custom transitions is the ability to present overlay view controllers. Previously, if you wanted to mimic the behavior of one of the social sharing sheets or do anything else that required the presenting view controller to remain visible behind the presented content, you were left to choose between a number of unfortunate options (adding views directly to the window was the most common approach I had seen). Happily, this is no longer necessary.</p>
<p>The key reason why this approach works is that when the presented view controller's <code>‑modalPresentationStyle</code> is set to <code>UIModalPresentationCustom</code>, the presenting view controller's view is not automatically removed from the view hierarchy. To build an overlay, it can simply be left as-is and the presented view controller's view can be displayed above it.</p>
<p>Here's the setup in <code>TWTExamplesListViewController</code>:</p>
```objc
- (void)presentPresentExample
{
    TWTOverlayExampleViewController *viewController = [[TWTOverlayExampleViewController alloc] init];
    viewController.delegate = self;
    viewController.modalPresentationStyle = UIModalPresentationCustom;
    viewController.transitioningDelegate = self;

    [self presentViewController:viewController animated:YES completion:nil];
}
```
<p>The two important things to note here are that <code>‑modalPresentationStyle</code> is set to <code>UIModalPresentationCustom</code>, and the presented view controller's <code>‑transitioningDelegate</code> is set to the <code>TWTExamplesListViewController</code>. When the present starts, <code>TWTExamplesListViewController</code> receives the transitioning delegate message:</p>
```objc
- (id&lt;UIViewControllerAnimatedTransitioning&gt;)animationControllerForPresentedController:(UIViewController *)presented
                                                                  presentingController:(UIViewController *)presenting
                                                                      sourceController:(UIViewController *)source
{
    return [[TWTPresentAnimationController alloc] init];
}
```
<p>As you can see, I created a custom animation controller class for this example. I am going to focus on the implementation of <code>‑animateTransition:</code> since it is the part that is different from before.</p>
<p>To start off, I extract the to view controller (the presented view controller) and the container view from the transition context.</p>
```objc
UIViewController *toViewController = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];

UIView *containerView = [transitionContext containerView];
```
<p>The container view is the superview for the from and to view controllers' views during the transition. Initially, the to view controller's view has not been added to the container view, and its frame has not been set. I am setting the frame directly here, but you could also use auto layout.</p>
```objc
CGRect frame = containerView.bounds;
frame = UIEdgeInsetsInsetRect(frame, UIEdgeInsetsMake(40.0, 40.0, 200.0, 40.0));

toViewController.view.frame = frame;

[containerView addSubview:toViewController.view];
```
<p>For this transition, I want the view to pop on to the screen. To do this, I animated the view's scale from 0.3 to 1 using a UIView spring animation.</p>
<p>As an aside, starting from a scale value greater than 0 and simultaneously animating the view's alpha from 0 to 1 lets you make the animation faster than you would want to make it if you simply scaled from 0 to 1. Try removing the alpha animation and starting the scale animation from 0 to compare.</p>
```objc
toViewController.view.alpha = 0.0;
toViewController.view.transform = CGAffineTransformMakeScale(0.3, 0.3);

NSTimeInterval duration = [self transitionDuration:transitionContext];

[UIView animateWithDuration:duration / 2.0 animations:^{
    toViewController.view.alpha = 1.0;
}];

CGFloat damping = 0.55;

[UIView animateWithDuration:duration delay:0.0 usingSpringWithDamping:damping initialSpringVelocity:1.0 / damping options:0 animations:^{
    toViewController.view.transform = CGAffineTransformIdentity;
} completion:^(BOOL finished) {
    [transitionContext completeTransition:YES];
}];
```
<p>In the completion block, I call <code>‑completeTransition:</code>, and that's it! Now for dismiss…</p>
<p>When the dismiss starts, <code>TWTExamplesListViewController</code> receives the transitioning delegate message:</p>
```objc
- (id&lt;UIViewControllerAnimatedTransitioning&gt;)animationControllerForDismissedController:(UIViewController *)dismissed
{
    return [[TWTDismissAnimationController alloc] init];
}
```
<p>Another custom animation controller. Let's take a look at <code>‑animateTransition:</code>:</p>
```objc
UIViewController *fromViewController = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];

NSTimeInterval duration = [self transitionDuration:transitionContext];

[UIView animateWithDuration:3.0 * duration / 4.0
                      delay:duration / 4.0
                    options:UIViewAnimationOptionCurveEaseIn
                 animations:^{
                     fromViewController.view.alpha = 0.0;
                 }
                 completion:^(BOOL finished) {
                     [fromViewController.view removeFromSuperview];
                     [transitionContext completeTransition:YES];
                 }];

[UIView animateWithDuration:2.0 * duration
                      delay:0.0
     usingSpringWithDamping:1.0
      initialSpringVelocity:-15.0
                    options:0
                 animations:^{
                     fromViewController.view.transform = CGAffineTransformMakeScale(0.3, 0.3);
                 }
                 completion:nil];
```
<p>As you can see, the idea is that it is the reverse of the present animation. The important difference to notice is that the from view controller's view is removed from the view hierarchy before the transition is complete.</p>
<p>Before going on, try making some changes to the dismiss animation.</p>
<h2>Recommended Patterns</h2>
<p>By now, you have probably noticed that this API relies heavily on protocols and delegate methods. Because of this, it is easy to end up with a custom transition implementation that is disorganized and impossible to reuse. Here are some patterns that help me keep things clean:</p>
<ol>
<li>
<p>Use dedicated objects for animation controllers</p>
<p>While it is possible for a view controller to double as the animation controller, doing so will almost certainly reduce the reusability of your custom transition. Besides, view controllers are already notorious for doing too much. You can create reusable animation controllers by simply creating NSObject subclasses that conform to <code>UIViewControllerAnimatedTransitioning</code> and then returning instances of them when needed.</p>
<p>We did this in <a href="https://github.com/twotoasters/Toast">Toast</a> with <a href="https://github.com/twotoasters/Toast/blob/master/UIKit/View%20Controller%20Transitions/TWTSimpleAnimationController.h"><code>TWTSimpleAnimationController</code></a> which is easy to reuse and include as a CocoaPod.</p>
</li>
<li>
<p>Prevent <code>UINavigationControllerDelegate</code> from needing to know about specific transitions</p>
<p>Returning the animation controller from the navigation controller's delegate is fine as long as you want all push and pop transitions to work the same way. In some cases, though, you might only want to customize a single push or pop.</p>
<p>One messy way to do it would be to temporarily change the navigation controller's delegate to an object that knows about the transition. Another messy way to do it would be to make the navigation controller's delegate have logic specific to certain combinations of from and to view controllers. Clearly, neither of those approaches is a good option.</p>
<p>A pattern that solves this problem cleanly is to add properties to <code>UIViewController</code> called <code>‑pushAnimationController</code> and <code>‑popAnimationController</code>. Then the navigation controller's delegate can be made to return the animation controller as specified by the pushed or popped view controller. This allows the navigation controller's delegate to remain generic and avoids the need to change which object is the delegate. <code>TWTNavigationControllerDelegate</code> implements this pattern and is also included in Toast.</p>
</li>
</ol>
<p>That wraps up this introduction to creating custom animated view controller transitions. Be sure to take a look at our <a href="https://github.com/twotoasters/Toast/tree/master/UIKit/View%20Controller%20Transitions">view controller transitions module in Toast</a>, and stay tuned for part two, which will cover <a href="http://objectivetoast.com/2014/04/14/interactive-transitions/">custom interactive transitions</a>.</p>
