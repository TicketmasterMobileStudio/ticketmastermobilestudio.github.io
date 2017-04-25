---
layout: post
status: publish
published: true
title: Localizing with Plurals and Genders
author:
  display_name: Josh Johnson
  login: jnjosh
  email: josh@twotoasters.com
  url: ''
author_login: jnjosh
author_email: josh@twotoasters.com
wordpress_id: 113
wordpress_url: http://objectivetoast.com/?p=113
date: '2014-04-21 12:00:27 -0400'
date_gmt: '2014-04-21 16:00:27 -0400'
categories: iOS Development
tags:
- Internationalization
- Localization
- Plural Rules
- Gender Rules
---
<p>You’ve just finished your latest application. Congrats! Even though you’re only releasing it in English, you’ve used <code>NSLocalizedString</code> to internationalize your user-facing strings. With strings that require dynamic quantities—“5 items” vs. “1 item”—you handled both the singular and plural cases. You shipped the app and moved on to your next project.</p>
<p>Suddenly your app is a hit… in Russia? You get a lot of feedback that your users want a Russian translation. Easy! You’ll just ship a Russian version of your strings file. Then you realize the problem: in English you can get away with that quick one/not-one check when pluralizing nouns; those are the only two plural categories in the language. Other languages aren’t so simple. Your Russian users have four different rules for pluralizing their nouns. Arabic speakers have six! Thankfully, there are a few tools to help you pluralize nouns for all of them.</p>
<p><!--more--></p>
<h3>Plural Rules</h3>
<p>The <a href="http://cldr.unicode.org">Unicode Common Locale Data Repository (CLDR) Project</a> provides tables, charts, and other resources to help software developers correctly internationalize their apps. For this article, we are most interested in <a href="http://www.unicode.org/cldr/charts/latest/supplemental/language_plural_rules.html">Language Plural Rules</a>. This chart provides a guide for how each language uses plurality with nouns for cardinal numbers (1 thing, 2 things) and with ordinal numbers (1st thing, 2nd thing). It does this by defining some short mnemonic categories and associated rules. <a href="http://cldr.unicode.org/index/cldr-spec/plural-rules">The category names</a>—<code>zero</code>, <code>one</code>, <code>two</code>, <code>few</code>, <code>many</code>, and <code>other</code>—are common across all languages; the rules define the mapping between a quantity and the plural category to use. For example, in English, the rules are:</p>
<p><code>one</code> ⇒ <em>i</em> = 1 and <em>v</em> = 0 (<a href="http://unicode.org/reports/tr35/tr35-numbers.html#Operands">where <em>i</em> = integer value and <em>v</em> = number of visible fraction digits</a>)<br />
<code>other</code> ⇒ No rule, or <em>any other number</em>.</p>
<p>With this rule we now know that, in English, we'd say 0 <em>things</em>, 1 <em>thing</em>, 1.5 <em>things</em>, or 2 <em>things</em>.</p>
<p>Russian rules are a little more complicated:</p>
<p><code>one</code> ⇒ <em>v</em> = 0 and <em>i</em> % 10 = 1 and <em>i</em> % 100 ≠ 11<br />
<code>few</code> ⇒ <em>v</em> = 0 and <em>i</em> % 10 = 2..4 and <em>i</em> % 100 ≠ 12..14<br />
<code>many</code> ⇒ (<em>v</em> = 0 and <em>i</em> % 10 = 0) or (<em>v</em> = 0 and <em>i</em> % 10 = 5..9) or (<em>v</em> = 0 and <em>i</em> % 100 = 11..14)<br />
<code>other</code> ⇒ No rule, or <em>any other number</em>.</p>
<h3>Gender Rules</h3>
<p>Supporting gender in languages—that is, properly handling pronouns—follows a similar pattern to the Plural Rules. English has three options: Male, Female, and Neutral. Polish has five. Because there's so much variation across languages, systems like the CLDR just assign each gender a number—0 for male, 1 for female, and 2 for neutral— instead of a mnemonic category.</p>
<h3>Supporting these Rules in your App</h3>
<p>Now that you know how the rules are defined, how do you make use of them to correctly internationalize your iOS app? There are a few open source solutions and even one from Apple as of iOS 7 and OS X 10.9, but none are perfect. Choosing one really depends on your needs and tastes.</p>
<h4>TTTLocalizedPluralString</h4>
<p>One of the many open source libraries by <a href="http://twitter.com/mattt">Mattt Thompson</a> is <a href="https://github.com/mattt/TTTLocalizedPluralString">TTTLocalizedPluralString</a> It works with your existing Localizable.strings file. For each plural noun, just include a separate string for each plural category mentioned above. For example:</p>
<p><code>%d Person (plural rule: one) = "One Person";</code><br />
<code>%d Person (plural rule: other) = "%d People";</code></p>
<p>When you need to display that string, use <code>TTTLocalizedPluralString</code> instead of using <code>NSLocalizedString</code>, passing in the quantity.</p>
```objc
return TTTLocalizedPluralString(count, @"Person", nil);
```
<p>The library then decides which category-tagged string to load based on the rules in the <a href="http://www.unicode.org/cldr/charts/latest/supplemental/language_plural_rules.html">CLDR</a>.</p>
<p>Unfortunately, this library does not support gender rules, so you'll still have to do that on your own.</p>
<h4>Smartling iOS-i8n</h4>
<p>Smartling provides another library for this called <a href="https://github.com/Smartling/ios-i18n">iOS-i8n</a>. Similar to <a href="https://github.com/mattt/TTTLocalizedPluralString">TTTLocalizedPluralString</a>, you provide options in your existing Localizable.strings file for each category.</p>
<p><code>"%d songs found##{one}" = "One song found";</code><br />
<code>"%d songs found##{other}" = "%d songs found";</code></p>
<p>Then use <code>SLPluralizedString</code>, or one of it's variants, instead of <code>NSLocalizedString</code>.</p>
```objc
return SLPluralizedString(@"%d songs found", 4, nil);
```
<p>We’ve used this library on several projects at <a href="http://twotoasters.com">Two Toasters</a> as it provides a simple interface to the plural rules.</p>
<p>Unfortunately, this library also has no support for gender rules.</p>
<h4>Apple's Localizable.stringsdict</h4>
<p>Last year, Apple announced built-in support for plural and gender rules in the <a href="https://developer.apple.com/library/ios/releasenotes/Foundation/RN-Foundation/#//apple_ref/doc/uid/TP30000742-CH2-SW56">Foundation Release Notes</a> for iOS 7 and OS X 10.9. It's a little more complicated than the open source libraries above. To get started, you need to:</p>
<ol>
<li>Create a Localizable.stringsdict file (this is simply a plist). <em>Note: you must still have a <code>.strings</code> file of the same name.</em> </li>
<li>Add the localizable dictionary details to the plist (more on this in a moment). </li>
<li>In your application, use <code>+[NSString localizedStringWithFormat:]</code> with the result of <code>NSLocalizedString</code> as the format.</li>
</ol>
<p>This is all pretty straightforward. <a href="https://github.com/ObjectiveToast/LocalizeTesting">Download a sample</a> based on the examples below.</p>
<p>Say you are writing an app that tracks and displays how many David Hasselhoff movies you’ve watched. We can set this up in our <code>.stringsdict</code> file. Again, this is just a plist, so we can edit it with either a plist editor or your text editor of choice.</p>
<p>First, we’ll create a new key with a dictionary value:</p>
```xml
&lt;key&gt;%lu out of %lu Hasselhoff movies watched&lt;/key&gt;
&lt;dict&gt;&lt;/dict&gt;
```
<p>This key is the value you’ll request later with <code>NSLocalizedString</code>.</p>
<p>Next we’ll add data to the dictionary to identify what we want to provide as a replacement.</p>
```xml
&lt;key&gt;%lu out of %lu Hasselhoff movies watched&lt;/key&gt;
&lt;dict&gt;
    &lt;key&gt;NSStringLocalizedFormatKey&lt;/key&gt;
    &lt;string&gt;%1$#@lu_hasselhoff_viewings@&lt;/string&gt;
&lt;/dict&gt;
```
<p>Here we supply <strong>another</strong> format as the value for our initial key. <code>@lu_hasselhoff_viewings@</code> is the new key, but where is it defined? We’ll add another key-value pair with this definition.</p>
```xml
&lt;key&gt;%lu out of %lu Hasselhoff movies watched&lt;/key&gt;
&lt;dict&gt;
    &lt;key&gt;NSStringLocalizedFormatKey&lt;/key&gt;
    &lt;string&gt;%1$#@lu_hasselhoff_viewings@&lt;/string&gt;
    &lt;key&gt;lu_hasselhoff_viewings&lt;/key&gt;
    &lt;dict&gt;
        &lt;key&gt;NSStringFormatSpecTypeKey&lt;/key&gt;
        &lt;string&gt;NSStringPluralRuleType&lt;/string&gt;
        &lt;key&gt;NSStringFormatValueTypeKey&lt;/key&gt;
        &lt;string&gt;lu&lt;/string&gt;
        &lt;key&gt;zero&lt;/key&gt;
        &lt;string&gt;No Hasselhoff Movies? You are hassling the Hoff.&lt;/string&gt;
        &lt;key&gt;one&lt;/key&gt;
        &lt;string&gt;Only one? You can do better.&lt;/string&gt;
        &lt;key&gt;other&lt;/key&gt;
        &lt;string&gt;%lu of %lu movies watched.&lt;/string&gt;
    &lt;/dict&gt;
&lt;/dict&gt;
```
<p>Now we are getting to those plural rule categories I mentioned earlier. This dictionary identifies the <code>NSStringFormatSpecTypeKey</code>, which in this case is <code>NSStringPluralRuleType</code>. I define the value type of the format value passed in, in this case the formatter is expecting <code>%lu</code> so the value is <code>lu</code>. Finally I define the different values to use depending on what the supplied value is. If it is <code>0</code>, then it will load the <code>zero</code> key.</p>
<p>This is obviously much more complicated than the open source libraries above, but it's also more flexible. In the <code>stringsdict</code> above, I created a new key called <code>lu_hasselhoff_viewings</code> and the rules for replacing it. With this format, you can place keys inside of other keys and build much more complicated patterns. This also means you can change the string replacement rules in the <code>.stringsdict</code> without changing the code requesting a string.</p>
<p>The <code>stringsdict</code> option even ostensibly supports gender rules via <code>NSStringGenderRuleType</code>. Unfortunately, as of iOS 7.1, the gender rule type only results in <code>(null)</code> being returned from the strings dictionary. If you’d like to see more about this, there is a <a href="http://openradar.appspot.com/radar?id=5824363005739008">radar submitted to Apple</a> (or if you are an Apple engineer, &lt;rdar://16670931>).</p>
<h3>Summary</h3>
<p>As you can see, there are a lot of options for properly internationalizing your application. Whether you go with the Apple solution or work with something available in the open source community, the most important thing is that you think ahead and build this into your application from the beginning. Retroactively internationalizing all your app’s strings is a lot more work than doing it incrementally throughout development. Just like using <code>NSLocalizedString</code> even before you need it, doing just a little more work to properly handle plurals and genders will help you reach a global audience faster.</p>
