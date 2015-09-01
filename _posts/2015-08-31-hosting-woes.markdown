---
layout: post
title: Hosting Woes
date: 2015-08-31
---

Now that I can build my blog via continuous integration, I want to push the
digital envelope and go to full continuous deployment.

### GitHub Pages

Apparently, the [most][ghpages1] [common][ghpages2] [way][ghpages3] to deploy
Jekyll applications is via [GitHub Pages][ghpages].  Put your Jekyll configuration
in GitHub and magically you can have your own blog complete with subdomain and
automated deployment.

Well that **sounds perfect!**  It's simple.  Lots of people are doing it.  Why
shouldn't I?

And then I ran across _this_ article:

> [F%#ck Github Pages for my Jekyll blog. Why I decided to use Digital Ocean
instead?][fckgh]

<!--more-->

Welp, looks like I'm going to probably run into just as many problems as this
poor folk had.  Better look for an alternative.

### Digital Ocean

This wasn't the first time I heard of [Digital Ocean][digitalocean].  A bunch
of my friends use it for things like [Mumble servers][mumble],
[Wedding websites][wedding], and more.  Honestly, I've been looking for a good
reason to try something other than [Rackspace][rackspace] (which I've been using
for the last 5+ years).

Using Digital Ocean, I can get my own server (aka "droplet"), run a web service
on it, deploy my "built" html to it, and then be able to browse my new blog.  

**Felt straight forward and very familiar.**

But as I perused the distributions and applications made available to me,
each of the options felt far too complex for my simple static HTML.

Do I pick the comfortable Ubuntu server where I still need to manage the
accounts, security, updates, etc?  What about the Docker application?  I could
build all my sites as simple Docker containers, deploy them to this server, and
use some fancy Nginx configuration to route between them.

All of this made me very uncomfortable.  I felt like nothing has changed since
I last did this 5+ years ago.

[digitalocean]: https://www.digitalocean.com/?refcode=eba200569184
[ghpages]: https://pages.github.com/
[ghpages1]: https://help.github.com/articles/using-jekyll-with-pages/
[ghpages2]: http://jekyllrb.com/docs/github-pages/
[ghpages3]: http://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/
[jekyll]: http://jekyllrb.com
[fckgh]: http://gon.to/2015/03/03/f-percent-number-ck-github-pages-for-jekyll-why-i-decided-to-use-digital-ocean/
[mumble]: http://wiki.mumble.info/wiki/Main_Page
[wedding]: http://francesanddavid.com/
[rackspace]: http://www.rackspace.com/
