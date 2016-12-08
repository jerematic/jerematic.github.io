---
layout: post
title: Working With Pass
categories: [ linux ]
comments: true
---
When it comes to password management, like most things I prefer simplicity.  The [pass](https://www.passwordstore.org/) library is a simple yet effective password management tool, consisting only of gpg encrypted text files.  I started using it for a customer project, but the flexibility and simplicity of it attracted me for my own password management needs.  This is a quick introduction that I wrote as I set this up for myself.

<!--more-->

## Installation

Ubuntu / Debian
{% highlight bash %}
sudo apt-get install pass
{% endhighlight %}

CentOS / RHEL
{% highlight bash %}
sudo yum install pass
{% endhighlight %}

Macintosh
{% highlight bash %}
brew install pass
echo "source /usr/local/etc/bash_completion.d/password-store" >> ~/.bashrc
{% endhighlight %}

## Getting Started
A GPG key is required, so be sure you have one generated.  If not, simple run:

{% highlight bash %}
gpg --gen-key
{% endhighlight %}

The default option of `RSA and RSA` is fine for most.  It's recommend to use `4096` for the maximum key size.  The key can exist for any length of time, but the default has no expiration.  Then finally enter your name and passphrase.

After the key is generated, you'll need the GPG key ID in order to initialize the pass repository.

{% highlight bash %}
gpg --list-secret-keys --keyid-format LONG
{% endhighlight %}

The output will look something like this where `3AA5C34371567BD2` is the ID you're looking for:

{% highlight bash %}
gpg --list-secret-keys --keyid-format LONG
/Users/hubot/.gnupg/secring.gpg
------------------------------------
sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
uid                          Example 
ssb   4096R/42B317FD4BA89E7A 2016-03-10
{% endhighlight %}

From here you can initialize the password store with the following command:

{% highlight bash %}
pass init 3AA5C34371567BD2
{% endhighlight %}

Then initialize a git repository to store the passwords.

{% highlight bash %}
pass git init
{% endhighlight %}

Now you can generate a new password with a set number of characters.  In this example I'm using 15 characters.

{% highlight bash %}
pass generate johndoe@example.com 15
{% endhighlight %}

You can insert an existing password.

{% highlight bash %}
pass insert amazon.com
{% endhighlight %}

Passwords can be organized into folders while creating or generating them as well.

{% highlight bash %}
pass insert Email/johndoe@example.com
{% endhighlight %}

And copying the password to your clipboard is as simple as:

{% highlight bash %}
pass -c johndoe@example.com
{% endhighlight %}

There's a lot more information available on the [pass man](https://git.zx2c4.com/password-store/about/) page and I'll follow up with more information as I integrate this into my routine.  There are a lot of community driven features so far including a [GUI](https://qtpass.org/), [Android client](https://github.com/zeapo/Android-Password-Store), and much more.  But tools like this are best when backed by a community customizing it to their needs, so maybe I'll add something useful at some point. 