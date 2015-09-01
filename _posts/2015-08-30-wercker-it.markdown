---
layout: post
title: Wercker It
date: 2015-08-30
---

I work on the central Continuous Delivery team at Yahoo.  So it goes without
saying that I want my personal website to have the same awesomeness.

My first thought was, this is [Jekyll][jekyll], so it should be pretty easy to just build
the static HTML and host it somewhere.  So I jumped to my go-to CD tool,
[Wercker][wercker].

<!--more-->

It has been a while since I used them.  Apparently, they got rid of their custom
boxes in favor of Docker images direct from the registry.  So after a bit of
fiddling, I pulled the ruby image, installed my dependencies, and ran a Jekyll
build.

{% gist stjohnjohnson/201d8b252eb8f9872e45 wercker.yml %}

Whoops.  Caught by a simple error.

{% gist stjohnjohnson/201d8b252eb8f9872e45 error1.log %}

Hmm, so it appears I need to pull in some sort of JavaScript runtime.  On my Mac
I already have Node.JS installed, so obviously I came up with the bad idea of
installing that in the container...

{% gist stjohnjohnson/201d8b252eb8f9872e45 node_wercker.yml %}

Ugh.  That added at least another minute to my build, but at least it passed.

After some discussions with myself, I determined that I was crazy and should
probably just click the link provided in the error message.  First item on that page
gave me a solution that just required another gem, [therubyracer][therubyracer],
to be added to my Gemfile.

{% gist stjohnjohnson/201d8b252eb8f9872e45 Gemfile %}

*Build Passed!*  

[![wercker status](https://app.wercker.com/status/24650efb0fd8d95cf920b12927980244/m/master "wercker status")](https://app.wercker.com/project/bykey/24650efb0fd8d95cf920b12927980244)

Awesome!!  And I got my build time to under a minute.

Next I need to figure out deployments.

[wercker]: https://wercker.com
[jekyll]: http://jekyllrb.com
[therubyracer]: https://github.com/cowboyd/therubyracer
