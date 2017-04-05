---
layout: post
status: publish
published: true
title: Introducing TTAlertView
author:
  display_name: Duncan Lewis
  login: duncan
  email: duncan@twotoasters.com
  url: ''
author_login: duncan
author_email: duncan@twotoasters.com
wordpress_id: 30
wordpress_url: http://objectivetoast.com/?p=30
date: '2012-11-21 22:35:26 -0500'
date_gmt: '2012-11-22 03:35:26 -0500'
categories:
- User Interface
tags:
- open-source
- two toasters
---
<p><a href="https://github.com/twotoasters/TTAlertView">TTAlertView</a> is a drop-in replacement for <a href="http://developer.apple.com/library/ios/#documentation/uikit/reference/UIAlertView_Class/UIAlertView/UIAlertView.html">UIAlertView</a> that allows the developer to customize the presentation of an alert. TTAlertView uses the familiar interface of UIAlertView, so you don't have to worry about rewriting any of your code. Just drop it in, add some assets, and <em>bam!</em>: you have a unique, customized alert view for your app!</p>
<h2>Using TTAlertView</h2>
<p>Using TTAlertView is simple. TTAlertView uses the familiar <code>initWithTitle:​message:​delegate:​cancelButtonTitle:​otherButtonTitles:</code> and <code>show</code> methods to create and display your alert view. From there TTAlertView handles laying out and animating the view.</p>
<p>Lets see some code:</p>
```
- (void)simpleAlert 
{ 
    TTAlertView *alert = [[TTAlertView alloc] initWithTitle:@"A Simple TTAlertView" 
                                                    message:@"... with the default layout!" 
                                                   delegate:self 
                                          cancelButtonTitle:@"Dismiss" 
                                          otherButtonTitles:nil];
    [alert show];
}
```
<p>... which gives you this:</p>
<p><img src="http://cl.ly/UDBV/download/Screen%20Shot%202012-10-19%20at%2011.23.32%20AM.png" alt="A real simple TTAlertView" /></p>
<p>Of course, this alert hasn't been customized at all, so it looks lame. Let's see what adding in some custom images can do for us...</p>
<h2>Customizing TTAlertView</h2>
<p>To customize this alert view, we're going to set the background image for <code>containerView</code> (the box containing the title, message, and buttons). Since TTAlertView handles the layout and, most importantly, the sizing of the <code>containerView</code>, using a resizable UIImage here is best practice -- this guarantees that no matter the size of the <code>containerView</code>, the image will be stretched appropriately to fit.</p>
<p>We're also going to add some button images using TTAlertView's <code>setButtonBackgroundImage:​forState:​atIndex:</code> method. With this method we will set images for the button's normal and highlighted states, again using resizable UIImages. (If you're using textured button images or button images with text baked in, worry not – we'll cover how to use these types of assets in a future blog post!)</p>
<p>Here's some more code:</p>
```
- (void)fancyAlert 
{
    TTAlertView *alert = [[TTAlertView alloc] initWithTitle:@"A Fancy TTAlertView" 
                                                    message:@"... with images and designs!" 
                                                   delegate:self 
                                          cancelButtonTitle:@"Dismiss" 
                                          otherButtonTitles:nil];

    // we don't want the background peeking through 
    [alert.containerView setBackgroundColor:[UIColor clearColor]];

    // resizable images work best 
    UIImage *backgroundImage = [bgImage resizableImageWithCapInsets:bgCapInsets]; 
    UIImage *buttonOffImage = [btnOffImage resizableImageWithCapInsets:buttonCapInsets];
    UIImage *buttonOnImage = [btnOnImage resizableImageWithCapInsets:buttonCapInsets];

    [alert.containerView setImage:backgroundImage]; 
    [alert setButtonBackgroundImage:buttonOffImage forState:UIControlStateNormal atIndex:0]; 
    [alert setButtonBackgroundImage:buttonOnImage forState:UIControlStateHighlighted atIndex:0];    

    [alert show];
}
```
<p>... and voilá!</p>
<p><img src="http://cl.ly/UDum/download/Screen%20Shot%202012-10-19%20at%2011.41.56%20AM.png" alt="A fancy, styled TTAlertView" /></p>
<h2>We're done here</h2>
<p>A quick shout-out to one of our favorite clients: <a href="http://www.gotryiton.com/">Go Try It On</a>. GTIO provided the assets used in the stylized alert example seen above -- the assets were taken from the custom alert view in GTIO's app, which is where TTAlertView was pioneered!</p>
<p>This is just a taste of TTAlertView's potential. In future posts we will cover more ways to use TTAlertView, like using fixed-size images, tweaking the layout, and supplying custom view hierarchies. Stay tuned!</p>
