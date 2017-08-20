---
layout: post
title:  "My OSX Setup"
date:   2017-08-14 20:00:00 +0200
---

This post includes my OSX Developer Setup and i uses this post to setup new macs,
so the order is important for some steps.

# OSX Dock Setup:

I prefer using an automatically hiding Dock but the animation duration drives me crazy. To speed up the Dock Animation execute the following 2 commands.

{% highlight shell %}
defaults write com.apple.dock autohide-time-modifier -float 0.25;killall Dock
defaults write com.apple.dock autohide-delay -float 0;killall Dock
{% endhighlight %}

# HACK Font

> A typeface designed for source code

- [HACK](http://sourcefoundry.org/hack/)

# Xcode

- install xcode from store
- [github.com/hdoria/xcode-themes](https://github.com/hdoria/xcode-themes/archive/master.zip)
- Xcode Theme: SecondGear

*xcode download takes long.. so go ahead with homebrew, it will download the xcode command line tools for you*

# Homebrew
{% highlight shell %}
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
{% endhighlight %}

# Carthage

{% highlight shell %}
brew install carthage
{% endhighlight %}

# Ruby (rvm)

{% highlight shell %}
curl -sSL https://get.rvm.io | bash -s stable --ruby
rvm install ruby-2.4
{% endhighlight %}

# Bundler

{% highlight shell %}
gem install bundler
{% endhighlight %}

# zsh and oh my zsh

{% highlight shell %}
brew install zsh zsh-completions
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
{% endhighlight %}

{% highlight config %}
# .zshrc
ZSH_THEME="eastwood"
{% endhighlight %}

# Sublime

- [Sublime](https://www.sublimetext.com)
- [Package Control](https://packagecontrol.io/installation)

You can add a symbolic link to the binary of sublime text, to easily open files and folders from the terminal.

{% highlight shell %}
ln -s "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" /usr/local/bin/subl

# usage in terminal:
subl foo.swift
{% endhighlight %}

Packages:
- IDL-Syntax
- Swift
- Dockerfile Syntax Highlighting
- Pretty JSON
- Gitignore
- Theme - Spacefunk
- Tomorrow Color Schemes

{% highlight json %}
# Preferences.sublime-settings
{
    "color_scheme": "Packages/Tomorrow Color Schemes/Tomorrow-Night.tmTheme",
    "draw_white_spaces": "all",
    "font_size": 18,
    "highlight_line": true,
    "ignored_packages":
    [
        "Vintage"
    ],
    "rulers":
    [
        80,
        120
    ],
    "scroll_past_end": true,
    "theme": "Spacefunk (Grey Tuesday).sublime-theme",
    "draw_white_space": "all"
}
{% endhighlight %}

# Docker

- [Docker](https://docs.docker.com/docker-for-mac/install/#download-docker-for-mac)

# Docker Compose

{% highlight shell %}
curl -L https://github.com/docker/compose/releases/download/1.12.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
{% endhighlight %}

# Vapor

{% highlight shell %}
brew install vapor/tap/vapor
{% endhighlight %}
