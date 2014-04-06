---
layout: post
title:  "Install Rails and Deploy a Rails Application"
date:   2014-04-06 10:02:30
categories: Rails
---

This post will outline the process of setting up Ruby, Rails, and Phusion Passenger. Phusion Passenger is an application server used as a middleware between a web server, e.g. Apache or Nginx, and Ruby making it easy to deploy any Rails application. Also, this document will go through the process of deploying a sample Rails application named *rails_app*. 

Here are the assumptions for this post:

* Installation is done in Ubuntu 12.10+
* Apache2 used as the web server
* A user account with sudo privileges named *user* is used to execute the commands

__Installing Ruby__

The first step is to install the required Ruby dependencies.

{% highlight bash %}
sudo apt-get update
sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev \
 libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev
sudo apt-get install libgdbm-dev libncurses5-dev automake libtool bison libffi-dev
{% endhighlight %}

Installing the core Ruby software packages can be difficult to do given the incompatibilities between the different versions of it. Fortunately, there are Ruby package managers that simplifies the installation process. A popular package manager, RVM (Ruby Version Manager), will be used to install Ruby and although a single-user installation of RVM is recommended, the server or multi-user installation is done instead.

So, download and set up RVM.

{% highlight bash %}
\curl -sSL https://get.rvm.io | sudo bash -s stable
{% endhighlight %}

An instruction will then show up requiring a user to be added to the *rvm* group and to run another command. Note too that there may be a need to re-open the shell terminal.

{% highlight bash %}
sudo usermod -a -G rvm lees
source /etc/profile.d/rvm.sh
{% endhighlight %}

Then, install Ruby using rvm. Ruby 1.9.3 will be used here but you can choose to install any other version.

{% highlight bash %}
rvm install 1.9.3
rvm use 1.9.3 --default
{% endhighlight %}

Verify the installation by running:

{% highlight bash %}
ruby -v
{% endhighlight %}

__Installing Rails__

Several Rails dependencies may require NodeJS, so add an APT repository for it and install.

{% highlight bash %}
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
{% endhighlight %}

Then, install Rails.

{% highlight bash %}
gem install rails
{% endhighlight %}

__Installing Passenger__

First, install the Passenger Ruby gem.

{% highlight bash %}
gem install passenger
{% endhighlight %}

Then, run the following command. The on-screen instructions are very intuitive and simple to follow. It will also recommend that certain packages may have to be installed first using *apt-get*.

{% highlight bash %}
passenger-install-apache2-module
{% endhighlight %}

Create Passenger’s Apache load module configuration.

{% highlight bash %}
sudo nano /etc/apache2/mods-available/passenger.load
{% endhighlight %}

Type in the following text in one line:

{% highlight bash %}
LoadModule passenger_module 
 /usr/local/rvm/gems/ruby-1.9.3-p545/gems/passenger-4.0.40/buildout/apache2/mod_passenger.so
{% endhighlight %}

And create too it’s other Apache configuration.

{% highlight bash %}
sudo nano /etc/apache2/mods-available/passenger.conf
{% endhighlight %}

That will have the following content:

{% highlight bash %}
<IfModule mod_passenger.c>
 	PassengerRoot /usr/local/rvm/gems/ruby-1.9.3-p545/gems/passenger-4.0.40
 	PassengerDefaultRuby /usr/local/rvm/gems/ruby-1.9.3-p545/wrappers/ruby
</IfModule>
{% endhighlight %}

Enable the Passenger module.

{% highlight bash %}
sudo a2enmod passenger
{% endhighlight %}

__Setting up the Rails Application__

Copy the *rails_app* folder to the user’s home directory (/home/user/rails\_app). And run bundle in it.

{% highlight bash %}
cd /home/user/rails_app
bundle install
{% endhighlight %}

Then, create the Apache site configuration.

{% highlight bash %}
sudo nano /etc/apache2/sites-available/rails_app
{% endhighlight %}

Type in the following content replacing the correct *ServerName* value:

{% highlight bash %}
<VirtualHost *:80>
  ServerName www.yourhost.com
  DocumentRoot /home/user/rails_app/public

  <Directory /home/user/rails_app/public>
     	# This relaxes Apache security settings.
     	AllowOverride all
     	# MultiViews must be turned off.
     	Options -MultiViews
  </Directory>
</VirtualHost>
{% endhighlight %}

Enable the site configuration and restart the Apache server.

{% highlight bash %}
sudo a2ensite rails_app
sudo service apache2 restart
{% endhighlight %}

Finally, open up the site in the web browser and verify if it’s running fine. And that should be all of it!
