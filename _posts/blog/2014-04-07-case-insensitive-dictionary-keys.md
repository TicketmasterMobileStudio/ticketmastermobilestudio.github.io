---
layout: post
status: publish
published: true
title: 'Case-Insensitive Dictionary Keys '
author:
  display_name: Prachi Gauriar
  login: prachi
  email: prachi@twotoasters.com
  url: http://twitter.com/prachigauriar/
author_login: prachi
author_email: prachi@twotoasters.com
author_url: http://twitter.com/prachigauriar/
wordpress_id: 24
wordpress_url: http://objectivetoast.com/?p=24
date: '2014-04-07 13:54:25 -0400'
date_gmt: '2014-04-07 17:54:25 -0400'
categories:
- Fundamentals
tags:
- Map tables
- Dictionaries
---
<p>On a recent project, I found myself needing a mutable dictionary whose keys were case-insensitive strings. A common solution to this problem is to normalize the case of each key before getting or setting its value:</p>
<p><!--more--></p>
```objc
- (id)valueForField:(NSString *)field
{
    return dictionary[field.lowercaseString];
}

- (void)setValue:(id)value forField:(NSString *)field
{
    dictionary[field.lowercaseString] = value;
}
```
<p>Unfortunately, I had an additional requirement that precluded this solution: I wanted the case of keys to be <em>preserved</em>. That is, if I added an entry using the key <code>@"Foo"</code>, I wanted to be able to get its value using <code>@"foo"</code> or <code>@"FOO"</code>; if I printed out the dictionary’s keys, I wanted to see the key as <code>@"Foo"</code>.</p>
<p>It wasn’t readily apparent how to achieve this without dropping into CoreFoundation or subclassing <code>NSMutableDictionary</code>, neither of which sounded like a lot of fun. Instead, I reached for <code>NSMutableDictionary</code>’s more flexible cousin <a href="https://developer.apple.com/library/ios/documentation/Cocoa/Reference/NSMapTable_class/Reference/NSMapTable.html" title="NSMapTable"><code>NSMapTable</code></a>. Map tables and mutable dictionaries have nearly identical interfaces, but map tables can also:</p>
<ul>
<li>Use keys that don’t conform to <code>NSCopying</code></li>
<li>Weakly reference their keys and/or values</li>
<li>Store non-objects as pointers</li>
<li>Use user-defined functions for hashing, equality checking, and memory management</li>
</ul>
<p>We can exploit this last feature to get a dictionary-like object with case-insensitive keys.</p>
<h2>Customizing Our Map Table</h2>
<p>We need to customize how our map table determines if two keys are equal, replacing strict string equality with case-insensitive comparison. Map tables use <a href="https://developer.apple.com/library/ios/documentation/cocoa/reference/foundation/classes/NSPointerFunctions_Class/Introduction/Introduction.html" title="NSPointerFunctions"><code>NSPointerFunctions</code></a> objects to determine how keys and values are compared and memory-managed. Our first task is to create a pointer-functions object with case-insensitive versions of <code>hashFunction</code> and <code>isEqualFunction</code>. We’ll start by implementing these functions:</p>
```objc
NSUInteger TWTCaseInsensitiveHash(const void *item, NSUInteger (*size)(const void *item))
{
    return [[(__bridge NSString *)item lowercaseString] hash];
}

BOOL TWTCaseInsensitiveIsEqual(const void *item1, const void *item2,  
                               NSUInteger (*size)(const void *item))
{
    NSString *string1 = (__bridge NSString *)item1;
    NSString *string2 = (__bridge NSString *)item2;
    return [string1 caseInsensitiveCompare:string2] == NSOrderedSame;
}
```
<p>Next, we need to create the pointer-functions object for our keys. Besides using our case-insensitive <code>…Hash</code> and <code>…IsEqual</code> functions, our keys should be like those of <code>NSDictionary</code>: they should be copied on insertion, use strongly referenced memory, and otherwise behave like objects.</p>
```objc
NSPointerFunctionsOptions options = NSPointerFunctionsCopyIn |
                                    NSPointerFunctionsStrongMemory |
                                    NSPointerFunctionsObjectPersonality;
NSPointerFunctions *keyFunctions = [[NSPointerFunctions alloc] initWithOptions:options];
keyFunctions.hashFunction = TWTCaseInsensitiveHash;
keyFunctions.isEqualFunction = TWTCaseInsensitiveIsEqual;
```
<p>Creating the value pointer-functions object is easier. Values should use strong memory and behave like objects.</p>
```objc
options = NSPointerFunctionsStrongMemory | NSPointerFunctionsObjectPersonality;
NSPointerFunctions *valueFunctions = [[NSPointerFunctions alloc] initWithOptions:options];
```
<p>Finally, we need to create a map table that uses these key and value pointer-functions:</p>
```objc
NSMapTable *mapTable = [[NSMapTable alloc] initWithKeyPointerFunctions:keyFunctions 
                                                 valuePointerFunctions:valueFunctions 
                                                              capacity:0];
```
<p>And that’s it. We now have a dictionary-like object with case-insensitive, case-preserving keys. Adding this as a category to <code>NSMapTable</code> is left as an exercise for the reader. While you’re at it, implement <code>‑objectForKeyedSubscript:</code> and <code>‑setObject:forKeyedSubscript:</code> so that your map table can use <a href="http://clang.llvm.org/docs/ObjectiveCLiterals.html#dictionary-style-subscripting" title="Objective-C Literals: Dictionary-Style Subscripting">dictionary-style subscripting</a>.</p>
<p>There is one important caveat with our solution: when setting objects for existing keys, <code>NSMapTable</code> reuses the existing key. For example,</p>
```objc
[mapTable setObject:@"Bar" forKey:@"Foo"];
NSLog(@"%@", [mapTable.keyEnumerator allObjects]);    // Outputs "Foo"

[mapTable setObject:@"Baz" forKey:@"FOO"];
NSLog(@"%@", [mapTable.keyEnumerator allObjects]);    // Also outputs "Foo"
```
<p>If this is a problem for you, you’ll need to remove the existing object before inserting the new one.</p>
<hr />
<p>Hopefully we’ve shown how easy it is to create interesting variations on dictionary-like objects using <code>NSMapTable</code>. Map tables are powerful, flexible objects, and becoming comfortable with them can save you a lot of time that would otherwise be spent subclassing <code>NSDictionary</code>.</p>
