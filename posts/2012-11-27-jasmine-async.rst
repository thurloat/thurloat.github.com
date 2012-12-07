:author: Thurloat
:date: 2012-12-05 19:30:00
:layout: post
:slug: jasmine-async
:status: publish 
:title: Jasmine Asynchronous Specs
:categories: Coffeescript, Testing
:tags: coffeescript, testing

I'll start out by saying that I'm a huge `Jasmine`_ fan. I have been 
working with it exclusively for testing during the last few months. However, 
it has become common knowledge that Jasmine's built-in methods for testing async 
code are pretty weak and make for either lots of boilerplate, or fragile tests.

Let's go on a journey together from a small greenfield test suite through to
a full(*er*) set of specs as the project grows. I will walk through how I came 
to implement a jasmine test extension for simplifying asynchronous specs 
which I believe is a more holistic approach than I've seen anywhere else. 

The few key differences you should note about the final solution are that it:

- Allows for fail-fast tests without breaking the ``waitsFor`` callback when an
  exception is thrown,
- Provides a simple mechanism to wrap async methods so their exceptions are
  caught and reported correctly,
- Doesn't require any different syntax than a Jasminaut is used to (less
  refactoring).

Affairs
=======

I wrote a `post`_ a few months ago which, at the time, dealt with my 
asynchronous woes enough to keep me going. To quickly re-cap check out the 
small snippet below.

**runs-spec.coffee**

.. code-block:: coffeescript

  describe "My Functionality", ->

    it "should behave as expected", ->
      collection = new Backbone.Collection()
      collection.fetch
        success: -> runs -> # <-- the magic.
          expect(true).toBeTruthy()
          
          # something ridiculous
          do candy.eat

  
As you can see, I am now executing the callback function as part of a jasmine
``runs()`` bloc. This causes the test to run the callback within the
scope of the jasmine environment, thus being able to report on the failed
specs. This solution worked for me for a while until I needed to write some
integration tests for a complex data controller. This process allowed me to find
find a weakness in the ``runs`` method: if the callback is never called the 
test **will pass anyway** -- false positives make it tough to track down 
problems, and reduce your confidence in the suite, making you test less, 
pushing you to break your software.

The first iteration of the solution was to implement an ``itMust`` method for
Jasmine. This provides a ``complete`` argument to the spec which it must call in
order for the test to pass. Here's the runner, and a sample of what the specs 
look like with ``itMust`` in use.

**itmust.coffee**

.. code-block:: coffeescript

  itMust = (desc, test) ->
    # create a new spec
    spec = jasmine.getEnv().it desc

    # run the test function passing it a callback that indicates success
    spec.runs ->
      complete = new sinon.spy
      test.apply spec, [ complete ]

      runs ->
        expect(complete).toHaveBeenCalled()

**itmust-spec.coffee**

.. code-block:: coffeescript

  describe "My DataController", ->
    itMust "return results to the dataset", (complete) ->
      # the application is actually automatically injected and torn down
      # but that's another post.
      app = new TestApplication

      # create a query
      query = app.datacontroller.createQuery(...)

      # fetch the query results from the datacontroller
      query.fetch()

      ds = query.dataSet
      # when the datSet is reset, there should be results.
      ds.on 'reset', -> runs ->
        expect(ds.recordCount).toBeGreaterThan 0
        complete()
      
      # respond leverages sinon's fake xmlhttprequest mocking so we can control
      # server responses.
      app.respond()


From here I was able to build out the test suite for the data controller without
running into too much trouble. However, it still required that you use the
``runs()`` block syntax on any asynchronous function, and leaving one out could
cause unfortunate results. Again, having tests pass when they should fail makes
the test suite useless. It was time to look into extending or *replacing* this
testing solution.

Examining The Options
=====================

After being frustrated by my situation, I stumbled across `Derick Bailey's`_ 
solution: `Jasmine.Async`_. He tries to solve this problem in the suite with a
new optional testing object called ``AsyncSpec``. In a nutshell, his approach
was to instantiate an ``AsyncSpec`` object for each test and wrap the spec in 
a ``waitsFor`` -> ``runs`` sequence that causes the spec to stall until the 
callback is called -- then allows it to continue on to the next jasmine block.

I had a few problems with this solution which enticed me to create a new/better 
one which is loosely based on Derick's approach **and** works for me. The 
biggest pain points for me with Jasmine.Async were: 

- I had to re-write all of my tests to use his approach of wrapping async 
  actions in the ``beforeEach`` function,
- It introduced a new ``AsyncSpec`` object that I didn't want to have to 
  instantiate for all of the specs, thus creating additional boilerplate,
- And when a test failed, it would wait the full 5000ms timeout that 
  ``waitsFor`` defaults to before a test failed if the ``done`` callback was 
  never fired. 
  
This slowed testing to a crawl because running a failing test
suite went from taking 1 second, up to (5x + 1)s. This can take 
entire minutes in some cases. 

Further information on why slow tests are bad, and good testing in general: 
`Mark Seemann`_ && `Gary Bernhardt`_

The other option which many have suggested, is migrating tests over to
`Mocha`_. Since Mocha has async testing nailed down, not to mention some
really sexy test runners (*more on this in another blog post*). Switching testing
utilities at this point in the project wasn't really an option for me, as it
would require rewriting hundreds of specs, as well as all of the development
tools that compile the test runner and run them in `phantomjs`_ (my headless
webkit of choice). If I were to greenfield a new project, this will definitely be a
project to look at, but some of us have to make the best with we have.

Without further adieu ...

The Codes
=========

Now that the journey has been laid out, I'll share the destination as well. My
solution is (I believe) a more natural approach to writing these specs than
other options allow.

**jasmine-async-example-spec.coffee**

.. code-block:: coffeescript

  describe "My DataController", ->
    
    # Just use `asyncIt` instead of `it`
    asyncIt "should return results to the dataset", (complete) ->
      app = new TestApplication
      query = app.datacontroller.createQuery(...)
      query.fetch()

      ds = query.dataSet
      ds.on 'reset', ->
        expect(ds.recordCount).toBeGreaterThan 0
        complete()
      
      app.respond()

Now that's what I'm talking about -- looks just like a regular jasmine spec
should, no hacks or anything. Specs run and fail just as they should. It also
provides an ``asyncBlock`` function generator that you can use to wrap a callback
the same way that the ``runs()`` block does which will provide additional details
on exceptions that get thrown while the test gets run. Otherwise, exceptions
are reported generally and immediately.

Below, I have included my `jasmine-async.coffee` testing utility which is a
pruned version of what I am using in production so I can demonstrate the
simplest version possible. You can take the code below and run with it; hopefully 
making your asynchronous testing a little smoother than it was before. <`Gist`_>


**jasmine-async.coffee**

.. code-block:: coffeescript

  @asyncIt = (desc, test) ->

    spec = jasmine.getEnv().it desc
    __spy = new sinon.spy
    __done = no
    _NATIVE_ERR = window.onerror

    spec.runs ->
      
      # any async test must call this function in order to pass. This function is
      # passed in as the first argument to the spec that gets run.
      complete = (failed) ->

        # restore the error handler to allow for the chrome console to catch the
        # errors once everyone has been restored.
        window.onerror = _NATIVE_ERR
        
        __done = yes
        if failed is undefined
          __spy()
        else
          # continue bubbling the error to the error console
          throw failed

      # Fail specs when async internals fail.
      window.onerror = (e, _, line) ->
        er = new Error "#{e}" 
        er.stack = "Trace Unavailable, Check console for detailed message. (use @asyncBlock for async callbacks in tests)" 
        spec.fail er
        complete e
    
      # latch an async block function that will allow for better error reporting 
      # within tests themselves, as well as failing quickly when things go wrong
      # by calling out to complete()
      spec.asyncBlock = (f) ->
        ->
          try
            f.apply spec, arguments
          catch e
            spec.fail e
            complete e

      # start off the test suite, reduce test boilerplate by kicking off the
      # session loading before the test starts, and wrapping the insides in a
      # spec failer and reporter.
      try
        test.apply spec, [ complete, __app ]
      catch e
        spec.fail e
        complete e 
        
      # wait for done to be set to `yes` (happens when complete() is called.
      waitsFor -> __done

      # finally, make sure the test ended up calling the complete() function. It
      # times out after ~5 sec if none of the other error handling catches a
      # problem with the test.
      runs ->
        (expect __spy).toHaveBeenCalled "Async Complete() never called."

.. _`Jasmine`: http://pivotal.github.com/jasmine/
.. _`Derick Bailey's`: https://twitter.com/derickbailey
.. _`Jasmine.Async`: http://lostechies.com/derickbailey/2012/08/18/jasmine-async-making-asynchronous-testing-with-jasmine-suck-less/
.. _`Mocha`: http://visionmedia.github.com/mocha/
.. _`phantomjs`: http://phantomjs.org/ 
.. _`Gist`: https://gist.github.com/4199060 
.. _`Gary Bernhardt`: http://pyvideo.org/video/631/fast-test-slow-test
.. _`Mark Seemann`: http://blog.ploeh.dk/2012/05/24/TDDTestSuitesShouldRunIn10SecondsOrLess.aspx
.. _`post`: /2012/07/06/jasmine-swallowing-async-exceptions

