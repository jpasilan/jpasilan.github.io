---
layout: post
title:  "Masking Digits In A Phone Number"
date:   2014-10-03 01:44:00
categories: PHP
---

A couple days ago, part of the feature that I was working on was to mask the digits in a phone number except for the last 2 digits and leaving the rest of the characters (e.g spaces, dashes, etc.) as is. Since I was working with a PHP project, I knew then that this will have to be done with regex and PHP's ***preg_replace()*** function.

After a few minutes of browsing through [regular-expressions.info], I was able to come up with the following regular expression:

{% highlight php %}
/\d(?!\d{0,1}$)/
{% endhighlight %}

Basically, what this expression means is that after the first delimiter (/) there is a *\d* which matches all the digits in a string. However, it is followed by a expression group containing regex's negative lookahead assertion *?!*, *\d{0,1}* which matches to 2 digits, and the end of string character *$*. This grouping should mean not to match, through the negative lookahead, the 2 digits before the end of the string.

The following is a sample code using this expression and PHP's ***preg_replace()***:

{% highlight php %}
<?php
$phoneNumber = '(555) 123-4567';
$maskedPhoneNumber = preg_replace('/\d(?!\d{0,1}$)/', '*', $phoneNumber);
?>
{% endhighlight %}

And this results in *$maskedPhoneNumber* assigned with the string:

{% highlight php %}
(***) ***-**67
{% endhighlight %}

<br /><br />
*Source:*

* [regular-expressions.info]

[regular-expressions.info]: http://www.regular-expressions.info/
[regular-expressions-lookaround]: http://www.regular-expressions.info/lookaround.html