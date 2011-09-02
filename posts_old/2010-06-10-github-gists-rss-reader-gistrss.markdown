--------------------------------------------------------------------------------
author: Thurloat
date: 2010-06-10 14:50:42
layout: post
slug: github-gists-rss-reader-gistrss
status: publish
title: GitHub Gists + RSS Reader = GistRSS
wordpress_id: '137'
categories:
- GitHub
- Google
- Personal
- programming
tags:
- Adam Thurlow
- appengine
- development
- GitHub
- programming
- python
--------------------------------------------------------------------------------

I am pleased to announce **[GistRSS](http://gistrss.appspot.com/)**
(a.k.a "Pretty GitHub Gist RSS Feed App") to everyone. I started working
on this application because of a problem that came up in conversation
with friends. The problem being that GitHub currently has some mediocre
Gist atom feeds. This app will allow you to subscribe to any GitHubber's
Gist feed and their Gist content will be sent straight to your RSS
Reader with syntax highlighting and all.

#### The History

Last weekend I was craving some good RSS feeds. After having a chat with
Erik Kastner, trying to get some really great new content in my Google
Reader. He sparked an idea. \*Roll the transcript\*

> .... Erik: I have a programming section but don't really have much in
> there Erik: I also follow defunkt's gists ;)
> http://gist.github.com/defunkt Me: i never thought of following gists,
> thats a neat idea Me: publish a new gist, so i can see what reader
> does when you post a new one Erik: it's not very great :/ Me: sounds
> like a cool project, github gist pretty rss maker Erik: that would be
> fantastic Me: \*adds to to-do list\* Erik: ttyl :)

A few hours later, I had a working prototype. Since then I've added some
cache control and syntax highlighting to the raw Gist content. I'm
pretty excited to have people using GistRSS and it's become something
that I use every day. There's no charge and never will be for this
service, thanks for Google's App Engine for hosting my tiny apps for nil
without coming anywhere near my free quota. According to my control
panel, there are somewhere around 10-15 folks subscribed to 3-4 feeds
each. Which is pretty impressive to me, for something that I may have
only mentioned once or twice on Twitter.

### Pretty Picture

[![image](http://img.skitch.com/20100610-nhdugxi6tcp4fjmwbk3jt97tt5.png "GistRSS in Google Reader")](http://gistrss.appspot.com)

#### The Future

I'm already hard at work on the next version, where I'm doing a bottom
up re-write. This includes caching of popular content in the Datastore,
and using the task queue to re-fetch any new Gists rather than
continuously pound the GistHub API when my memcache copy goes away. I'm
also working on generating a dynamic 'popularly subscribed gist feeds'
so you can find out what other people have subscribed to. If you
encounter any bugs, irregularities or have some comments on how to
improve GistRSS, head over to my open project repo:
[http://github.com/thurloat/gistrss](http://github.com/thurloat/gistrss)
and please submit an issue report. I'll be checking it constantly. So
now that you've reached the end; head over to
[http://gistrss.appspot.com/](http://gistrss.appspot.com/) and consume
someone's Gists.
