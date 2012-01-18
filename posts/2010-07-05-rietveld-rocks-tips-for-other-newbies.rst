:author: Thurloat
:date: 2010-07-05 21:20:00
:layout: post
:slug: rietveld-rocks-tips-for-other-newbies
:status: publish
:title: Rietveld Rocks. Tips for other newbies!
:categories: Programming, Google, Code Review
:tags: appengine, code review, development, programming, python, rietveld, sheepdog

The Potatoes
============

Peer code review has become one of my favorite feedback processes by
far. At the SheepDog office, we're forming the habit of reviewing any
code that comes out of a sprint. In order to move a task from tested to
verified, it must get signed off by an engineer who did not write
the code. This process was quickly becoming cumbersome when done manually. 
For example; A large feature would be created, then tested, and no one had the 
time to look it over because the review process was: check out the source code
at the correct location, comb through it, and provide manual feedback to
the other developer -- had it not slipped your mind to review it in the
first place. 

We installed the code review tool Rietveld on our production domain this week. 
Getting it installed on our domain was incredibly easy; however it's not the 
easiest option to find. *PROTIP*: Once you're in the Google Apps Domain Control 
Panel, click ``Add Services``, Then you'll see this little link at the bottom 
where you can install it..

.. image:: http://commondatastorage.googleapis.com/thurloat/domainlabs.png
    :alt: Domain Labs

Once inside, you can install the application as you would any other to
the domain. Of the many cool features, one of the things we quickly fell
in love with was the ability to leave a review without touching the
mouse. This made reviewing code much easier and seemingly familiar with
vi-ish key bindings; for example:

* ``j`` next diff,
* ``k`` previous diff,
* ``n`` next line,
* ``p`` previous line,
* ``m`` commit and mail review.

The Meat
========

On top of the slick navigation, the Google team has made it even more brainless to
actually post your diff online for review using the *git* commands that we
already knew! One of the things that isn't so well documented in the
wiki is the ``Upload.py`` usage. There's a fun little operator on the end
that gets no documentation [-- diff\_options] which allows for more
advanced diff patches.

simply upload all un-commited changes to be reviewed

.. code-block:: bash

    $ upload.py

these are some cooler usages
----------------------------

upload all changes to a file from a certain commit point to the
present for review

.. code-block:: bash

    $ upload.py -- GIT_COMMIT_ID..HEAD -- src/main.py

upload all changes for a span commits to a module for review

.. code-block:: bash

    $ upload.py -- GIT_COMMIT_ID..GIT_COMMIT_ID2 -- src/gisthub/

append a new diff patch to an existing issue, useful if someone has asked for a 
change in review

.. code-block:: bash

    $ upload.py -i ISSUE_ID src/main.py

We're currently working on layering our own python script on top of the
upload one in order to better match our newly defined internal
processes. Any other shortcuts and time-savers we discover; I'll be sure
to share them here. Hopefully that'll get your wheels spinning and into
a better habit of doing peer code review. Happy Coding!
