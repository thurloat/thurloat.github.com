:author: Thurloat
:date: 2011-02-19 22:41:00
:layout: post
:slug: google-closure-linter-textmate-bundle
:status: publish
:title: Google Closure Linter TextMate Bundle
:categories: Programming, GitHub, Google
:tags: development, programming, TextMate

.. image:: http://thurloat.com/wp-content/uploads/2011/02/googleclosuretmbundle.png
    :align: center
    :alt: Google Closure TextMate Bundle

I love the `Google Closure <http://code.google.com/closure/utilities/>`_
project. As a big fan of its compiler and the linter, I use both
regularly at work and for my personal projects. However, it is not
without its flaws. The biggest hurtle to integrating the Closure Linter
into my workflow: it's strictly a command line tool. I have nothing against using
the terminal but, the workflow for running the linter over my JS looks
like this:

1.  Make some changes to the JS file.
2.  Open the terminal, run ``gsjlint source.js``
3.  Look over the results (`ugly terminal output`).
4.  Jump back to the Editor, and find where the problems were.
5.  go to 1.

This tedium quickly became frustrating. I wanted a more automated
approach to running the Closure Linter; so I hacked together a TextMate Bundle 
to make all of our lives a little easier. I was able to integrate the linter into 
my TextMate workflow without a hitch. Thus, here's my new workflow:

1.  Make some changes to the JS file.
2.  Hit "``Shift + Ctl + Opt + A``", results appear in a TextMate Browser
    window, next to the code.
3.  Keep making changes, and using keyboard shortcut to update the
    linter window.

Here's a screenshot of this new workflow in action:

.. image:: http://thurloat.com/wp-content/uploads/2011/02/ClosureLinter-1024x672.jpg
    :align: center
    :alt: Closure Linter Workflow
    :width: 625px

*Pretty neat*! However, some of the changes that the Closure Linter wants
you to make are monotonous. I don't want to waste time putting a space
before my ``=`` sign, or converting double quotes ``"`` to single quotes ``'``.
Luckily, part of the Closure Linter utilities is a "``fixjsstyle.py``"
utility that will run over your JS and perform all of those little
corrections for you. I've included this in the Bundle with the shortcut
"``Shift + Ctl + Opt + Z``". This will run the style fixer over your
current file and display what it fixed in a tooltip.

.. image:: http://thurloat.com/wp-content/uploads/2011/02/Google-Closure-Style-Fixer-1024x409.jpg
    :align: center
    :alt: Google Closure Style Fixer
    :width: 625px

If you're interested in giving it a try you can
`download <https://github.com/thurloat/GoogleClosure.tmbundle/tarball/master>`_
the bundle directly. The 
`Closure Utilities <http://code.google.com/closure/utilities/docs/linter_howto.html>`_
are required for this bundle to work. Alternatively, the source code is available on
`GitHub <https://github.com/thurloat/GoogleClosure.tmbundle>`_. The README
contains installation instructions for the git version; you can keep the
bundle up to date using the
`GetBundles <https://github.com/adamsalter/GetBundles.tmbundle>`_ manager
(which you should be using if you aren't already). If you find any bugs,
or have any feature requests, GitHub has a great Issue Tracker that I'd
love if you used. This way I can keep everything in one spot. Feel free
to fork and contribute any patches; all are welcome.

Good Luck !
===========
