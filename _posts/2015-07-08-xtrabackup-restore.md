---
layout: post
title: Xtrabackup Restore
categories: [ linux ]
comments: true
---

When restoring a mysql backup from [Percona's Xtrabackup](https://www.percona.com/software/percona-xtrabackup), there's a few things to remember.  First, be sure to include the *-i* flag while extracting the tar archive.  Second, the backup needs apply the xtrabackup_logfile to play back the binary log and catch back up to the time of the snapshot.  

<!--more-->

{% highlight bash %}
pv backup.tar.gz | tar xivf - -C /var/lib/mysql-incoming
innobackupex --use-memory=1G --apply-log /var/lib/mysql-incoming
{% endhighlight %}

Now replace the current mysql directory with this backup and change the ownership accordingly.

{% highlight bash %}
mv /var/lib/mysql /var/lib/mysql.old
mv /var/lib/mysql-incoming /var/lib/mysql
chown -R mysql. /var/lib/mysql
{% endhighlight %}
