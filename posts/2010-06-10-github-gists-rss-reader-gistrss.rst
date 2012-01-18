:author: Thurloat
:date: 2010-06-10 14:50:00
:layout: post
:slug: github-gists-rss-reader-gistrss
:status: publish
:title: GitHub Gists + RSS Reader = GistRSS
:categories: programming, GitHub, Google, Personal
:tags: Adam Thurlow, appengine, development, GitHub, programming, python

I am pleased to announce `GistRSS <http://gistrss.appspot.com/>`_
(a.k.a "Pretty GitHub Gist RSS Feed App") to the world. The idea for this
application came about after a frustrating conversation with a friend of mine.
The problem that we exposed was that GitHub currently has mediocre
Gist atom feeds. GistRSS will allow you to subscribe to any GitHubber's
Gist feed and their Gist content will be sent straight to your RSS
Reader fully syntax highlighed.

The History
===========

After scouring the internet for some great tech RSS feeds, and bugging
my friends for help, Erik Kastner came to the rescue with an idea to get
fresh content into my RSS Reader.

The Transcript
##############

.. code-block:: text

    Erik: I have a programming section but don't really have much in there 
    Erik: I also follow defunkt's gists ;) http://gist.github.com/defunkt 
    Me: i never thought of following gists, thats a neat idea 
    Me: publish a new gist, so i can see what reader does when you post a new one 
    Erik: it's not very great :/ 
    Me: sounds like a cool project, github gist pretty rss maker 
    Erik: that would be fantastic 
    Me: \*adds to to-do list\* 
    Erik: ttyl :)

A few hours later, I had a working prototype. Since then I've added some
caching and syntax highlighting to the raw Gist content. I'm
pretty excited to have people using GistRSS and it's become something
that I use every day. There's no charge and never will be for this
service, thanks for Google's App Engine for hosting my tiny apps for nil
without coming anywhere near my free quota. According to my control
panel, there are somewhere around 10-15 folks subscribed to 3-4 feeds
each. Which is pretty impressive to me, for something that I may have
only mentioned once or twice on Twitter.

.. image:: http://img.skitch.com/20100610-nhdugxi6tcp4fjmwbk3jt97tt5.png
    :align: center
    :alt: GistRSS in Google Reader

The Future
==========

I'm hard at work on the next version, where I'm doing a bottom
up re-write. This includes caching of popular content in the Datastore,
and using the task queue to re-fetch any new Gists rather than
continuously pound the GistHub API when my memcache copy goes away. I'm
also working on generating a dynamic 'popularly subscribed gist feeds'
so you can find out what other people have subscribed to. If you
encounter any bugs, irregularities or have some comments on how to
improve GistRSS, head over to my open project repo:
`github.com/thurloat/gistrss <http://github.com/thurloat/gistrss>`_ and please submit an issue report. 
I'll be checking it occasionally. So now that you've reached the end; head over to
`gistrss.appspot.com <http://gistrss.appspot.com/>`_ and consume
someone's Gists.
