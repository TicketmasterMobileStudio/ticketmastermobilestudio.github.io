---
layout: post
status: publish
published: true
title: Validating Data with TWTValidation
author:
  display_name: Prachi Gauriar
  login: prachi
  email: prachi@twotoasters.com
  url: http://twitter.com/prachigauriar/
author_login: prachi
author_email: prachi@twotoasters.com
author_url: http://twitter.com/prachigauriar/
wordpress_id: 208
wordpress_url: http://objectivetoast.com/?p=208
date: '2014-08-13 16:38:07 -0400'
date_gmt: '2014-08-13 20:38:07 -0400'
categories:
- Tools
tags:
- open-source
- TWTValidation
- Validation
- Key-Value Coding
- Key-Value Validation
---
<p>A few months ago, <a href="https://twitter.com/a_hershberger" title="Andrew Hershberger on Twitter">Andrew</a> and I were working on a project that required some validation of model objects and form data. After looking around a bit, we quickly came to the conclusion that there aren’t any widely accepted best practices for data validation in Cocoa. If you’re using Core Data, you can declare some validators for strings and numbers in your data model. If you’re not, you have to roll your own validation system, perhaps using <a href="https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/KeyValueCoding/Articles/Validation.html" title="Key-Value Validation">Key-Value Validation</a> (KVV), a part of Foundation that many experienced Cocoa developers have never even heard of.</p>
<p><!--more--></p>
<p>There’s a huge gap between these two options. Core Data’s managed object validation is declarative: you don’t describe <em>how</em> to perform validation, you just declare <em>what</em> validations need to take place. Key-Value Validation is the exact opposite: you override <code>‑validateValue:​forKey:​error:</code> (or <code>‑validate«Key»:​error:</code>) and write the code that actually performs the validations. What we set out to do was to bring the simplicity and clarity of Core Data’s declarative approach to non-Core Data classes. We think we’ve done that with <a href="https://github.com/twotoasters/TWTValidation" title="TWTValidation on GitHub">TWTValidation</a>.</p>
<h2>TWTValidation</h2>
<p>TWTValidation is a Cocoa framework that allows you to easily validate data with reusable validator objects, each of which is a subclass of <code>TWTValidator</code>. The common interface for all validators is a single method: <code>‑validateValue:​error:</code>. This method returns whether a given value is valid and, if not, indirectly returns an error describing why validation failed. With just this simple interface, we’ve been able to build some pretty interesting validators. While going over each of them would take too much time, here’s a brief summary:</p>
<ul>
<li><strong>Value validators</strong> can validate that a value is of a particular class. They can also optionally allow for values that are <code>nil</code> or the <code>NSNull</code> instance.
<ul>
<li><strong>Number validators</strong> can check if a number is within a given range. You can also optionally require that a value be an integer.</li>
<li><strong>String validators</strong> are value validators that validate strings. There are a variety of string validators for doing things like checking the length of a string or validating that it matches a particular pattern.</li>
</ul>
</li>
<li><strong>Compound validators</strong> allow you to combine multiple validators using logical operators like AND, OR, and NOT.</li>
<li><strong>Collection validators</strong> validate the count and elements of arrays and sets.</li>
<li><strong>Keyed collection validators</strong> validate the count, keys, values, and specific key-value pairs for dictionaries and map tables.</li>
<li><strong>Block validators</strong> allow you to easily create a <code>TWTValidator</code> with custom validation logic using a block.</li>
</ul>
<p>These validators are incredibly useful—see the [TWTValidation readme][Readme] for code examples—and fundamentally important to any use of TWTValidation, but my favorite class in the framework is <code>TWTKeyValueCodingValidator</code>.</p>
<h2>Key-Value Coding Validators</h2>
<p>Key-value coding (KVC) validators validate an object using validators that the object itself provides. Each KVC validator is initialized with a set of KVC keys.</p>
<pre><code>#!objc
NSSet *keys = [NSSet setWithObjects:@"property1", @"property2", nil];
TWTKeyValueCodingValidator *validator = [[TWTKeyValueCodingValidator alloc] initWithKeys:keys];
</code></pre>
<p>Later, when the validator validates an object, it iterates over each key in its key set, gets the object’s value for that key (using <code>‑valueForKey:</code>), gets the object’s validators for that key, and then validates the key’s value. In pseudo-code, this looks like:</p>
<pre><code>#!objc
for (NSString *key in self.keys) {
    id value = [object valueForKey:key];

    // Get validators from object somehow
    NSSet *validators = … ;

    // Run the validators on value, accumulating and returning any errors
}
</code></pre>
<p>How does it get object’s validators for a key? <code>TWTKeyValueCodingValidator</code> declares an informal protocol containing the method <code>+twt_validatorsForKey:</code>, which classes can override to return the set of <code>TWTValidator</code> objects for a given KVC key. For example,</p>
<pre><code>#!objc
+ (NSSet *)twt_validatorsForKey:(NSString *)key
{
    if ([key isEqualToString:@"property1"]) {
        // Create and return an NSSet containing the validators for property1
    } else if ([key isEqualToString:@"property2"]) {
        // Create and return an NSSet containing the validators for property2
    }

    // Otherwise, ask the superclass
    return [super twt_validatorsForKey:key];
}
</code></pre>
<p>Unfortunately, cascading if-else statements like this are sort of gross, particularly when you have a lot of keys you want to validate. We remedy this by borrowing a simple pattern from <a href="https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Protocols/NSKeyValueCoding_Protocol/Reference/Reference.html" title="Key-Value Coding">Key-Value Coding</a>, <a href="https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Protocols/NSKeyValueObserving_Protocol/Reference/Reference.html" title="Key-Value Observing">Key-Value Observing</a>, and Key-Value Validation. The base implementation of <code>+twt_validatorsForKey:</code> checks to see if the receiver responds to <code>+twt_validatorsFor«Key»</code>, where <em>«Key»</em> is the capitalized form of the key parameter. If so, we simply return the result of that method; otherwise, we returns <code>nil</code>. Knowing this, we can now simplify the cascading if-else into:</p>
<pre><code>#!objc
+ (NSSet *)twt_validatorsForProperty1
{
    // Create and return an NSSet containing the validators for property1
}

+ (NSSet *)twt_validatorsForProperty2
{
    // Create and return an NSSet containing the validators for property2
}
</code></pre>
<p>To my eyes, this is a lot clearer. Basically, if you implement these methods, validation of the <code>property1</code> and <code>property2</code> keys will just work. Any other keys will not be validated, i.e., they will implicitly pass, because you haven’t declared any validators for them.</p>
<p>Let’s walk through a complete example just to drive this whole idea home.</p>
<h2>Example: Validating a User Object</h2>
<p>For our example, we’ll create a simple class that models users for some imaginary online service.</p>
<pre><code>#!objc
@interface TWTUser : NSObject

@property (nonatomic, strong) NSNumber *ID;
@property (nonatomic, strong) NSString *fullName;
@property (nonatomic, copy) NSString *username;

@end
</code></pre>
<p>User IDs must be positive integers. Full names must be strings between 0 and 64 characters long. Usernames must only contain alphanumeric characters and <code>'_'</code> and must be between 1–15 characters long. To work with <code>TWTKeyValueCodingValidator</code>s, we need to implement <code>+twt_validatorsForID</code>, <code>+twt_validatorsForUsername</code>, and <code>+twt_validatorsForFullName</code>. Let’s get to it.</p>
<p>Validating the ID is really simple. We just need to create a number validator that allows a minimum value of 1, no maximum, and requires integral values:</p>
<pre><code>#!objc
@implementation TWTUser

+ (NSSet *)twt_validatorsForID
{
    TWTNumberValidator *validator = [[TWTNumberValidator alloc] initWithMinimum:@1 maximum:nil];
    validator.requiresIntegralValue = YES;
    return [NSSet setWithObject:validator];
}

@end
</code></pre>
<p>Easy enough. Validating full names is very similar. We just need a string validator with a minimum length of 0 and a maximum length of 64. We’ll go ahead and allow <code>nil</code> for good measure.</p>
<pre><code>#!objc
+ (NSSet *)twt_validatorsForFullName
{
    TWTStringValidator *validator = [TWTStringValidator stringValidatorWithMinimumLength:0 maximumLength:64];
    validator.allowsNil = YES;
    return [NSSet setWithObject:validator];
}
</code></pre>
<p>Validating usernames is only slightly trickier. TWTValidation has support for validating strings using a regular expression. We can express the validation rule for usernames with the (unescaped) regular expression <code>^\w{1,15}$</code>. This pattern only matches strings that are composed of 1–15 alphanumeric or underscore characters.</p>
<pre><code>#!objc
+ (NSSet *)twt_validatorsForUsername
{
    NSRegularExpression *usernameExpression = [[NSRegularExpression alloc] initWithPattern:@"^\w{1,15}$"
                                                                                   options:0
                                                                                     error:NULL];

    TWTStringValidator *validator = [TWTStringValidator stringValidatorWithRegularExpression:usernameExpression
                                                                                     options:0];
    return [NSSet setWithObject:validator];
}
</code></pre>
<p>Okay, so we have our validators declared. Now we just need to actually perform validation. Let’s add an <code>‑isValid</code> method to <code>TWTUser</code> for that purpose. Internally, it will use a KVC validator to actually perform the validation. We’ll store that in an internal property.</p>
<pre><code>#!objc
@interface TWTUser ()
@property (nonatomic, strong) TWTKeyValueCodingValidator *objectValidator;
@end


@implementation TWTUser

- (instancetype)init
{
    self = [super init];
    if (self) {
        NSSet *keys = [NSSet setWithObjects:@"ID", @"fullName", @"username", nil];
        _objectValidator = [[TWTKeyValueCodingValidator alloc] initWithKeys:keys];
    }

    return self;
}

// ...

@end
</code></pre>
<p>Finally, we can implement <code>‑isValid</code>:</p>
<pre><code>#!objc
- (BOOL)isValid
{
    return [self.objectValidator validateValue:self error:NULL];
}
</code></pre>
<p>And that’s it.</p>
<p>There are some important things to point out about our implementation. First, while we have our own private KVC validator, other KVC validators can be used as well. This might be desirable if you only wanted to validate the user’s username and ID, but not its full name.</p>
<p>We also aren’t limited to performing validation internally. If you have a controller that needs to perform some validation on a different key set, it can do that using its own KVC validator, which will still get the appropriate validators from the <code>TWTUser</code> object.</p>
<p>Finally, KVC validators can co-exist with KVV. For example, suppose that we didn’t implement <code>+twt_validatorsForFullName</code> and instead had an equivalent KVV method:</p>
<pre><code>#!objc
- (BOOL)validateFullName:(NSString **)ioValue error:(NSError **)outError
{
    if (!*ioValue || ([*ioValue isKindOfClass:[NSString class]] &amp;&amp; [*ioValue length] &lt;= 16)) {
        return YES;
    } else if (outError) {
        // Construct an NSError and assign it to outError
    }

    return NO;
}
</code></pre>
<p>When the KVC validators finds that you have no validators declared for <code>fullName</code>, it falls back on key-value validation to do its work. In other words, everything just works. This can be useful if you want to transition your code from KVV to TWTValidation.</p>
<hr />
<p>So, that’s a taste of TWTValidation. We think it’s a great approach to doing data validation in Cocoa. We’ve only covered a small part of the framework, so read <a href="http://cocoadocs.org/docsets/TWTValidation/" title="TWTValidation Documentation at CocoaDocs.org">the docs</a> and take a look at our <a href="https://github.com/twotoasters/TWTValidation" title="TWTValidation on GitHub">GitHub project page</a>. We’re also hard at work on the next release of TWTValidation, so be on the lookout for some updates in the next couple of weeks. If you have any questions or ideas for new validators, I’m <a href="https://twitter.com/prachigauriar" title="Prachi Gauriar on Twitter">@prachigauriar</a> on Twitter!</p>
