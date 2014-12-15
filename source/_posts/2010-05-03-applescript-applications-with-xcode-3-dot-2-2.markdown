---
layout: post
title: "AppleScript applications with Xcode 3.2.2"
date: 2010-05-03
comments: true
categories: macosx
---
{% img left /images/blog_posts/xcode.png %}

Xcode 3.2.2 on Snow Leopard does not support building new AppleScript applications anymore. It does allow you to edit pre-builded AppleScript projects, but you need to enable the AppleScript Studio palette in Xcode, which is hidden... I've build an installer package that will take care of it all and restore this feature in Xcode 3.2.2.

What it does is to automatically install "AppleScript Application", "AppleScript Automator Action" and "AppleScript Droplet"<!--more-->new project templates to "`/Developer/Library/Xcode/Project Templates/Application/`". This package will also automatically enable the hidden AppleScript Studio palette with the following command:

`defaults write com.apple.InterfaceBuilder3 IBEnableAppleScriptStudioSupport -bool YES`

[Download](http://www.mediafire.com/?2zyzmoojjgz) the Installer Package

##Quick start AppleScript application guide:

Open Xcode and click "Create a new Xcode project". You will be presented with the following screen:

{% img center /images/blog_posts/applescript1.png %}

On the right, click on "AppleScript Application" and then the "Choose" button. Give your project a name and click "Save". You will be presented with the following screen:

{% img center /images/blog_posts/applescript2.png %}

Double click the `MainMenu.xib` file for the Interface Builder App to open up. In the Library pane, as shown below, type "button" in the search area. All button options will be shown:

{% img center /images/blog_posts/applescript3.png %}

Drag your button to the blank Window UI that you are designing:

{% img center /images/blog_posts/applescript4.png %}

We now need to link the button to our scripting code. Single click on the button and go to the Inspector AppleScript tab as shown below. Make the changes where marked in red:

{% img center /images/blog_posts/applescript5.png %}

In the Interface Builder main menu, click File and then Save. Close Interface Builder. You will now be back at the following screen:

{% img center /images/blog_posts/applescript6.png %}

Single click on `yourProject.applescript` to reveal the code. Add your code to the area where it says `"(Add your script here.*)"`. Then lastly click the "Build and Run" button. You will now have an App that will do what the code tells it to when the button is clicked!
