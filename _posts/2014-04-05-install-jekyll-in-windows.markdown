---
layout: post
title:  "Install Jekyll in Windows"
date:   2014-04-05 12:49:00
categories: jekyll
---

As indicated in the documentation, Jekyll does not yet support Windows. Alas, after a couple of
hours of searching on Google, I found the correct help, packages, and support tools that will make Jekyll run well
on Windows. Note that the whole installation process was done in Windows 7. That's in case this will not work for any
other version of Windows.

Required packages and tools:

1. RailsInstaller
2. Python 2.7
3. ez_setup.py

Download and install Ruby using the [RailsInstaller] package. You can just use its default installation settings.

Once done, open up Git Bash and install the Jekyll Ruby gem and its dependencies.

{% highlight bash %}
gem install jekyll
{% endhighlight %}

The installation should be going smoothly. In fact, you can start creating your blog.

{% highlight bash %}
jekyll new blog
{% endhighlight%}

Now, the problem starts when you try to run *__jekyll serve__*. This most likely has something to do with the
pygment.rb gem. 

To solve this, you can run Python's setuptools. That said, download and install first [Python 2.7] and add its executable path to Windows PATH environment variable. Then download and run [ez_setup.py]. You may need re-open Git Bash so that the changes to the Window's PATH variable will take effect.

{% highlight bash %}
python ez_setup.py 
{% endhighlight %}

There will still be a problem with pygment.rb which can be solved by installing a lower version of it. So run the following gem commands to replace pygment.rb 0.5.4 with 0.5.0.

{% highlight bash %}
gem uninstall pygment.rb 0.5.4
gem install pygment.rb 0.5.0
{% endhighlight %}

Verify if everything's well already by running the jekyll server and opening up http://localhost:4000 in your preferred browser.

{% highlight bash %}
jekyll serve
{% endhighlight %}

<br /><br />
*Sources:*

* [Jekyll Documentation]
* [madhur.co.in]
* [yizeng.me]

[RailsInstaller]: http://railsinstaller.org/en
[Python 2.7]: https://www.python.org/download/
[ez_setup.py]: https://pypi.python.org/pypi/setuptools
[yizeng.me]: http://yizeng.me/2013/05/10/setup-jekyll-on-windows
[Jekyll Documentation]: http://jekyllrb.com/docs/installation/
[madhur.co.in]: http://www.madhur.co.in/blog/2011/09/01/runningjekyllwindows.html