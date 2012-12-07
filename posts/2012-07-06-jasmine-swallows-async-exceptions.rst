:author: Thurloat
:date: 2012-07-06 23:00:00
:layout: post
:slug: jasmine-swallowing-async-exceptions
:status: publish
:title: Jasmine Tests Swallowing Asynchronous Exceptions
:categories: Javascript, Coffeescript, Testing
:tags: javascript, jasmine, testing

On a recent cross-continent programming expedition, I ran into an **exceptionally**
frustrating bug while writing tests using the `Jasmine`_ unit testing framework.
After coming up with a solution on my own, I scoured the web for additional insight
and only found one other mention of this bug ( *and have since lost it* ).
This reference proposed a solution very similar to mine.
The most disheartening aspect is that this bug is not necessarily a bug within
Jasmine but can only be worked around by following a specific code pattern
which I'll be describing later.

This bug took ... a *while* to track down. Such a long while in fact that most of the
time was spent detecting the bug in the first place. It manifested by causing
tests to pass. I'm not sure if you are aware, but one of the first rules of
good unit tests in my books is that they do **not** emit false positives. False
positives in a test suite usually indicates that your tests don't have good branch
coverage or they do not assert the correct behaviour - not that many of your
tests are naively written in a way which makes them pass no matter what kind of
junk you put between the ``{}`` in an asynchronous callback.

Steps To Reproduce
------------------

1. Write some tests examining some asynchronous behaviour.
2. Expect something reasonable.
3. Write something that will throw an exception.

Outcome: All tests pass and you believe that you write bug-free code.

Desired Outcome: Test fails and the Exception is logged.

In the code samples below I set up a simple Jasmine test suite and expect
that ``true`` should be ``truthy`` in both Coffeescript and Javascript. Both tests
demonstrate the *"bug"* in full effect.

coffeescript
============

.. code-block:: coffeescript

    describe "My Functionality", ->
        it "should behave as expected", ->
            collection = new Backbone.Collection()
            collection.fetch
                success: ->
                    expect(true).toBeTruthy()
                    # something ridiculous
                    do candy.eat

javascript
==========

.. code-block:: javascript

    describe("My Functionality", function() {
      it("should behave as expected", function() {
        var collection = new Backbone.Collection();
        collection.fetch({
          success: function() {
            expect(true).toBeTruthy();
            candy.eat();
          }
        });
      });
    });


Output
======

.. code-block:: bash

    Actual:

        My Functionality: 1 of 1 expectations passed.
        Passed : My Functionality : should behave as expected.

    Expected:

        Tests should not have passed! There was a bloody exception!

As you can see from the output block my tests are obviously not behaving
in a manner becoming of a good test. Here comes the "answer"...

The Answer-around
-----------------

The most elegant way I found to work around these swallowed exceptions in the
Jasmine tests was to use one of Jasmine's utility functions ``runs()``. A call
to ``runs()`` executes the callback as if it were a blocking operation. One
of the handy features of the runs function is that it brings along nicely the
scope of the test suite inside of the anonymous function you write
inside of it. For more information on ``runs()`` check the `Asynchronous Specs`_
documentation for `Jasmine`_.

So, the pattern is for **any** success / fail / callback that you define within a test,
to make sure to wrap the function definition inside of a ``runs()`` call. The
following snippets are examples in both Coffeescript and Javascript again.

Coffeescript
============

.. code-block:: coffeescript

    describe "My Functionality", ->
        it "should behave as expected", ->
            collection = new Backbone.Collection()
            collection.fetch
                success: -> runs -> # <-- the magic.
                    expect(true).toBeTruthy()
                    # something ridiculous
                    do candy.eat

Javascript
==========

.. code-block:: javascript

    describe("My Functionality", function() {
      return it("should behave as expected", function() {
        var collection = new Backbone.Collection();
        collection.fetch({
          success: function() {
            runs(function() {
              expect(true).toBeTruthy();
              candy.eat();
            });
          }
        });
      });
    });

Output
======

.. code-block:: bash

    Actual:

        My Functionality: 0 of 1 expectations passed.
        Failed : My Functionality : should behave as expected
            - ReferenceError: candy is not defined

**Bing!** - And there's the pattern. It is a little more verbose in plain
javascript but, it still fulfils the main goal.

Now, we can all breathe a sigh of relief that our false positives are being
caused by poor coverage, not bad tests.

Amendment
---------

It is worth noting that while coming across this test pattern, a couple of
different approaches were taken. An alternative ( yet ugly ) pattern worth
mentioning is to wrap the whole function body in a ``try {} catch {}`` block
and fail an expectation. I don't recommend this, as it pollutes your test
code with additional branches that need maintaining to do exception handling
whereas the ``runs()`` statement only needs to come before your declaration.

.. code-block:: coffeescript

    collection.fetch
        success: ->
            try
                expect(true).toBeTruthy()
                do candy.eat
            catch error
                # force Jasmine to assert badly and print the exception
                # as the failure message.
                expect(false).toBeTruthy error

.. _`Jasmine`: http://pivotal.github.com/jasmine/
.. _`Asynchronous Specs`: http://github.com/pivotal/jasmine/wiki/Asynchronous-specs
