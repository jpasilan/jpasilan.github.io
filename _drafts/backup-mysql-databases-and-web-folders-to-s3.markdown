---
layout: post
title:  "Backup MySQL databases and Web Folders to Amazon S3"
date:   2014-04-06 12:22:26
categories: Amazon S3
---

This post will outline how to create and schedule backups to S3.

{% highlight bash %}
#!/bin/bash

if [ -z $1 ] || [ -z $2 ]; then
 	echo "DB Backup Usage: $0 db S3_BUCKET MYQSL_USER MYSQL_PASS"
    echo "Web Backup Usage: $0 www S3_BUCKET [WEB_DIRECTORY_PATH]"
    exit 1
fi

# Set backup type.
if [ "$1" = "db" ]; then
    if [ -z $3 ] || [ -z $4 ]; then
        echo "DB Backup Usage: $0 db S3_BUCKET MYSQL_USER MYSQL_PASS"
        exit 1
    fi
    TYPE="db"
else
    TYPE="www"
fi

# Check if S3 bucket exists.
S3_MATCH=$(s3cmd ls | grep "$2$")
if [ "$?" = "1" ]; then
    echo "Error: S3 Bucket $2 does not exist."
    echo "Exiting..."
    exit 1
fi

HOST=$(hostname)
BACKUP_DIR="/tmp/backups/$TYPE"
S3_BUCKET="s3://$2/$TYPE/$HOST"

if [ ! -d $BACKUP_DIR ]; then
    mkdir -p $BACKUP_DIR
fi

echo "Cleaning up $BACKUP_DIR..."
rm -rf $BACKUP_DIR/*

if [ $TYPE = "db" ]; then
    USER=$3
    PASS=$4
    MYSQL_DB=$(mysql --user=$USER --password=$PASS -e "SHOW DATABASES;")
        
    echo "Creating backups of databases..."
    cd $BACKUP_DIR

    for DB in $MYSQL_DB
    do
        # The Database column header is included in the array,
        # so exclude it. Exclude the databases built-in to MYSQL
        # as well.
        [ $DB = "Database" -o $DB = "information_schema" -o $DB = "performance_schema" \
        -o $DB = "mysql" -o $DB = "test" ] && continue
        FILE="$DB.sql.gz"

        echo "Backing up $DB..."
        mysqldump --user=$USER --password=$PASS $DB | gzip -5 > $FILE

        # DB dumps are backed up daily for a month's duration.
        DAY="day-$(date +%d)"

        echo "Uploading $FILE to S3..."
        s3cmd put $FILE $S3_BUCKET/$DAY/$FILE 2>&1

        echo "Done..."
    done
else
    # Check if a Web directory is supplied and it exists.
    # Otherwise, use the default.
    echo "Checking for Web directories..."
    if [ ! -z $3 ] && [ -d "$3" ]; then
        echo "Found $3"
        WEB_DIR="$3"
    elif [ -d "/var/www" ]; then
        echo "Found /var/www"
        WEB_DIR="/var/www"
    else
        echo "Error: Web directory not found."
        echo "Web Backup Usage: $0 www S3_BUCKET [WEB_DIRECTORY_PATH]"
        echo "Exiting..."
        exit 1
    fi

	# Traversing to $WEB_DIR.
    cd $WEB_DIR
    WWW_DIR=$(ls)

    echo "Creating backups of the web directories..."
    for DIR in $WWW_DIR
    do
        FILE="$DIR.tar.bz2"
        FILE_PATH="$BACKUP_DIR/$FILE"

        echo "Backing up $WEB_DIR/$DIR..."
        tar cpjf $FILE_PATH $DIR 2>&1

        # Web directories are backed up every week.
        WEEK="week-$(date +%U)"

        echo "Uploading $FILE to S3..."
        s3cmd put $FILE_PATH $S3_BUCKET/$WEEK/$FILE 2>&1

        echo "Done..."
    done
fi

echo "Removing all backups in $BACKUP_DIR..."
rm -rf $BACKUP_DIR/*
echo "All Done!"
{% endhighlight %}