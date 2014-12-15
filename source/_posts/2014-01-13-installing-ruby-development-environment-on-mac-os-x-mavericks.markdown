---
layout: post
title: "Installing Ruby development environment on Mac OS X Mavericks"
date: 2014-01-13 16:52:02 +0200
comments: true
categories: macosx other
---
{% img left /images/blog_posts/ruby.png %}

Installing Ruby development environment on Mac OS X Mavericks. Ruby on rails with the Ruby version manager, MySQL server and Brew.
<!--more-->
<br>
<br>
<br>

##Installing the prerequisites:

The Xcode command-line tools is required. This can be installed after you've installed the full Xcode installation or as a standalone by running the following command:

**Installing the Xcode command-line tools:**

{% codeblock lang:bash %}
xcode-select --install
{% endcodeblock %}

[Homebrew](http://brew.sh/) is my preferred package manager for Mac. It is similar to [MacPorts](http://www.macports.org/).

**Installing Homebrew:**

{% codeblock lang:bash %}
ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go/install)"
{% endcodeblock %}

...after the installation run the following command:

{% codeblock lang:bash %}
brew doctor
{% endcodeblock %}

##Installing MySQL server:

MySQL server can be installed via Homebrew or just by downloading and installing the [official DMG](http://dev.mysql.com/downloads/mysql/) file.

{% codeblock lang:bash %}
brew install mysql
{% endcodeblock %}

{% codeblock lang:bash %}
vi ~/.bashrc
{% endcodeblock %}

add `export PATH=$PATH:/usr/local/mysql/bin` for a DMG installation
or
add `export PATH=$PATH:$(brew --prefix mysql)/bin` if you installed via Homebrew.

##Installing gcc 4.9

{% codeblock lang:bash %}
brew tap homebrew/versions
brew install gcc49
brew install autoconf automake libtool libyaml readline libksba openssl
{% endcodeblock %}

{% codeblock lang:bash %}
vi ~/.bashrc
{% endcodeblock %}

Add the following three lines. Leave them commented out. You can uncomment them when you need gcc as compiler.

{% codeblock lang:bash %}
#export CC=/usr/local/bin/gcc-4.9
#export CPP=/usr/local/bin/cpp-4.9
#export CXX=/usr/local/bin/g++-4.9
{% endcodeblock %}

##Installing [RVM](http://rvm.io/), the Ruby version manager:

{% codeblock lang:bash %}
\curl -L https://get.rvm.io | bash -s stable
{% endcodeblock %}

**Testing if installation was successfull:**

After the installation is complete, restart the terminal App and type the following command:

{% codeblock lang:bash %}
type rvm | head -n 1
{% endcodeblock %}

You should get the following response:

{% codeblock lang:bash %}
rvm is a function
{% endcodeblock %}

##Installing Ruby:

{% codeblock lang:bash %}
rvm install ruby-2.0.0-p247
{% endcodeblock %}

**Setting and verifying the used Ruby version:**

{% codeblock lang:bash %}
rvm use 2.0.0-p247
{% endcodeblock %}

{% codeblock lang:bash %}
ruby -v
{% endcodeblock %}

##Installing and verifying rails:

{% codeblock lang:bash %}
gem install rails
{% endcodeblock %}

{% codeblock lang:bash %}
rails -v
{% endcodeblock %}

Other Ruby gems can now also be installed by just running:

{% codeblock lang:bash %}
gem install some_gem
{% endcodeblock %}

[Here](http://guides.rubygems.org/command-reference/) is the command reference for the `gem` command.
