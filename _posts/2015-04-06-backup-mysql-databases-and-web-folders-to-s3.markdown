---
layout: post
title:  "Backup MySQL Databases and Web Folders to Amazon S3"
date:   2015-04-16 12:22:26
categories: Amazon S3
---

This post will outline how to create and schedule backups to Amazon S3.

Add IAM User
===

As recommended by Amazon, create an IAM group, say S3Admin, and assign it with the S3 full access permission. Then, create an IAM user, say S3User, and add it to the S3Admin group. Download the user credentials keeping note of the access and secret keys.

Install Tools
===

Install the s3cmd package and run it with the --configure option. It is important to provide the access key, secret key, and encryption password when prompted. Verify the configuration and if it runs fine, save it.

<p>
{% highlight bash %}
sudo apt-get install s3cmd
s3cmd --configure
{% endhighlight %}
</p>

A warning pertaining to python-magic being missing may show up when running s3cmd. Resolve this by installing python-pip, which is the Python package manager, and the python-magic package.

<p>
{% highlight bash %}
sudo apt-get install python-pip
pip install python-magic
{% endhighlight %}
</p>

Write Shell Script
===

Below is the shell script that creates a backup and uploads it to S3. This script can be run manually or can be scheduled to run through cron.

{% gist jpasilan/abe9397d0c1e5b41c3dc s3backup.sh %}

Make the file executable and move it to /usr/local/bin or any directory that your user shell account access to.

<p>
{% highlight bash %}
chmod +x s3backup
mv s3backup /usr/local/bin/
{% endhighlight %}
</p>

Schedule Backups Through Cron
===

To schedule the backups to S3 through cron, add a crontab entry like so.

<p>
{% highlight bash %}
@weekly /usr/local/bin/s3backup s3://[bucket] [full_directory_path] > /dev/null 2>&1
@daily /usr/local/bin/s3backup s3://[bucket] [db_user] [db_pass] [db_name] > /dev/null 2>&1
{% endhighlight %}
</p>

The first entry is for backing up a directory while the second backs up a database. Just replace the paths and database information with your own.
