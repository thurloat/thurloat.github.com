--------------------------------------------------------------------------------
author: Thurloat
date: 2009-07-13 22:03:22
layout: page
slug: projects
status: publish
title: Projects & Codes
--------------------------------------------------------------------------------

These are some of the projects i'm working on or am involved in. This is
also a place where I keep links to useful snippets of code that I want
to share.

### Google Closure Linter TextMate Bundle

A TextMate Bundle created out of necessity to streamline my workflow
using the excellent Closure Utilities. It encapsulates both the Linter
output in a nice TextMate HTML Output window, and the fixjsstyle utility
to make those quick style changes. Blog Post:
[http://thurloat.com/2011/02/19/google-closure-linter-textmate-bundle/](http://thurloat.com/2011/02/19/google-closure-linter-textmate-bundle/)
Source Link:
[https://github.com/thurloat/GoogleClosure.tmbundle](https://github.com/thurloat/GoogleClosure.tmbundle)

### GistRSS

A fun little GitHub Gist RSS feed maker. This grew out of a want to
condense the sites I visit each day. I can now prettily follow Gists in
my Google Reader. Feel free to make use of it if you can. Website Link:
[http://gistrss.appspot.com/](http://gistrss.appspot.com/) Source Link:
[http://github.com/thurloat/gistrss/](http://github.com/thurloat/gistrss/)

### GSUtil (branch)

\*\* **NOTE:** As of June 16th my code has been merged into the official
GSUtil project (svn diff [http://goo.gl/0K9g](http://goo.gl/0K9g)), they
chose not to pull in gzip, so I'll re-implement my gzip into the latest
gsutil version later this week.**\*\*** **\*\* NOTE2:**As of July 2nd,
the gzip functionality has been pulled into trunk, and passed code
review within Google. So unless I come up with some other features, I'll
just keep this around for historical purposes :) \*\* Since the release
of Google Storage for Developers, I've seen the need to expand on the
current toolset that Google is providing for migrating data. Including
adding options for custom headers, gzipping the file contents and
settings ACLS right from the copy command. Official Source Link:
[http://code.google.com/p/gsutil](http://code.google.com/p/gsutil/) Old
Source
Link:[http://github.com/thurloat/GSUtil](http://github.com/thurloat/GSUtil)

### GAEImages

I've opened up one of my little projects which will use a Flash image
compressor for django and appengine, to compress the file down below 1mb
for uploads. Really useful for user uploaded images and avoiding the
BlobStore (if you're into that sort of thing) Source Link:
[http://github.com/thurloat/GAEImages](http://github.com/thurloat/GAEImages)

### HTTPDB

This project is a pure experiment on how deep you can dive into the
Python Datastore API, or in our case, try to completely avoid it by
building our own protocol buffers and running our RPC calls in async
mode. Really nifty stuff inside. in an apachebench test, it returns the
same json string as a django + memcached setup about 2x quicker. Source
Link:
[http://github.com/thurloat/httpdb](http://github.com/thurloat/httpdb)

### Dashboard Framework

I use the Dashboard Framework daily at my job. The best way to sum it up
is on the front page of it's website:

"Dashboard is a modular open-source web framework that incorporates CMS,
eCommerce, blogs, SEO, and many other popular features out of the box.
But best of all Dashboard was developed from the ground up to be
flexible and rapid to deploy"

I'm one of 5 contributing developers to this project, and we're working
on making it better every day, which is evident in the change log and
how quick we turn over our lighthouse tickets.
