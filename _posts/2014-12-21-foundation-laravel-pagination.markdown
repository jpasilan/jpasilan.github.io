---
layout: post
title:  "Foundation as Laravel Paginator"
date:   2014-12-21 04:03:00
categories: Laravel
---

[Laravel], by default, uses [Bootstrap] as its pagination utility. Developers, however, are often agnostic on their use of frameworks. That they may choose to use [Foundation] with [Laravel] on their projects. So in this blog I'll outline how one can switch to [Foundation] and use it as [Laravel]'s pagination utility.

Extend the Presenter
====

First, extend the *Illuminate\Pagination\Presenter* class and override its methods. In overriding the methods, you may only need to replace the [Bootstrap]-specific CSS classes with its [Foundation] equivalent. It is not necessary to override all of the Presenter's methods. 

Also, do not forget to add the namespace (in the case of the snippet below, Libraries, but it can be anything else) in the autoload directive in *composer.json*. When choosing to autoload in PSR-4, note that it is case-sensitive such that "Libraries" is not the same as "libraries".

{% gist jpasilan/f2e6a8f4b2dcb8924271 FoundationPresenter.php %}

Create the Pagination Template
====

Next is to create the pagination template and add the HTML markup and CSS classes required by [Foundation] for the pagination to work. Add a line too that will instantiate a FoundationPresenter object and call its render() method.

{% gist jpasilan/f2e6a8f4b2dcb8924271 foundation.blade.php %}

Set the Pagination View
====

Lastly, using the *pagination* array key, set the view configuration to use the template above as the application's pagination view template.

{% gist jpasilan/f2e6a8f4b2dcb8924271 view.php %}


[Laravel]: http://laravel.com
[Foundation]: http://foundation.zurb.com
[Bootstrap]: http://getbootstrap.com

