:author: Thurloat
:date: 2012-01-21 18:15:00
:layout: post
:slug: backbonejs-readable-modular-testable-views
:status: publish
:title: Backbone.js - Modular, Readable and Testable Views
:categories: programming, backbone
:tags: coffeescript, backbone, development, javascript

After more work with Backbone on projects at `Sheepdog`_, we are starting
to develop a solid foundation that will allow us to easily develop,
**test**, and maintain our Backbone.js applications. Having these properties in
our JS / CoffeeScript is very important, as we're striving toward better test
coverage from server to client in order to best deliver the highest quality 
product to our clients.

The approach that I'm taking to solve our problem is a simple implementation 
of two common design patterns found in MVP applications. The Observer Pattern,
and the Factory Pattern.

I, Observer
===========

Obtaining much of my MVP knowledge from one of our products written in GWT 
(`gTrax`_) using the awesome GWT-Presenter and EventBus
libraries, it's been ingrained into my being that all independant pieces of 
the application should  communicate exclusively over an agreed-upon API. 
Rather than touting the benefits of modular code, I'll assume that since you're
still reading this you understand.

EventBus helps consolidate communication while developing in java and GWT, 
and I didn't have to look far to find a flexible solution already available. 
Backbone already mixes in the **Backbone.Events** class into all core pieces 
of the framework so it's the perfect candidate (*shaving a yak, and all that*).
While I've seen this pattern implemented a few times, at `Los Techies`_ most
notably, it has been missing a key piece of the puzzle. Either people are 
binding an instance of **Backbone.Events** directly to the window, and operate
using that, or manually pass an instance of it to each vew they create. I have 
two main problems with these methods which I'll outline in my review of the 
following code example.

**problem.coffee**

.. code-block:: coffeescript

    class MyView extends Backbone.View
      
      tagName: "div"
      
      events:
        "click .close_button": "closeView"
        "change input": "formChanged"
        
      initialize: ->
        # ... some post-constructor stuff
        # bind on our eventBus object
        @options.eventBus.bind "globalEvent", @respondToGlobal
        
      render: ->
        # ...
        
    $ ->
      eventBus = _.extend {}, Backbone.Events
      
      # Create a view, pass the bus
      myView = new MyView
        eventBus: eventBus
        
      # Create a second view, and pass the event bus to it as well
      mySecondView = new MyView
        eventBus: eventBus
        
My first problem with the above code is that it doesn't follow the IMO 
"Awesomeness" that is Backbone.js style. I am of the opinion that one of the
best parts of Backbone is how they suggest you structure your View classes.
To me it is immediately obvious that, in the above class declaration:

- The View will render in a ``<div></div>``.
- It respondes to a click event on the ``.close_button`` class.
- It watches for changes on any form inputs, then calls ``formChanged``.

There is a key piece of information is being tucked away in ``MyView``'s
declaration. It registers its most important event ``globalEvent`` which
gets exposed via the global eventBus. It should be a first class citizen 
along with ``events`` and ``tagName`` which clearly outlines how the View
will behave.

The second problem I have with the above code is that you're required in your
implementation to create your own instance of **Backbone.Events** and pass
it to each view you want to create, anywhere in the application. This isn't
the **worst** thing, however, what happens when your views have 2 dependancies?
3? 4? ...

Enter F%#tories
===============

So many programmers cringe when they hear the word **Factory**. Factory, 
factory, factory, factory. Now that you've said it 5 times it just sounds 
silly and we can talk about them seriously and without cringing.

This following ViewFactory solves the second problem that I had with our
initial introduction to the Observer pattern above, and I'll address the first
and most serious problem a little later. 

**factory.coffee**

.. code-block:: coffeescript

    class ViewFactory
      constructor: ->
        @eventBus = _.extend {}, Backbone.Events
        @registry =
          factory: @
      register: (key, value) ->
        @registry[key] = value
      create: (ViewClass, options) ->
        options = options or {}
        passedOptions = _.extend options, @registry, pubSub: @pubSub
        klass = ViewClass
        klass.prototype.eventBus = @eventBus
        new klass(passedOptions)
   
**usage.coffee**

.. code-block:: coffeescript

    $ ->
      factory = new Factory
      myView = factory.create MyView
      
    # or
    
    $ ->
      factory = new Factory
      factory.register "foo", "bar"
      ...
       
In 10 lines of coffeescript, I've shown how you can have a simple factory that 
is useful in a few key ways that are adventageous for structuring a large 
Backbone.js application.

1. It will automatically bind the eventBus (Observer) object to any class 
   we create.
2. It has a registry that will bolt onto Backbone's ``@options`` for any View 
   built using the factory for when your views become more complext and require
   additional dependancies.
   
The Final Countdown
===================

Now you're ready for the good stuff. Using the ``ViewFactory`` we can guarantee
that our views will be built with the dependencies injected, and use it to 
build a class that looks like this.

**awesome.coffee**

.. code-block:: coffeescript

    class MyView extends ModularView
      
      tagName: "div"
      
      events:
        "click .close_button": "closeView"
        "change input": "formChanged"
      
      globalEvents:
        "globalEvent": "respondToGlobal"
        
      initialize: ->
        # ... some post-constructor stuff
        
      render: ->
        # ...
        
Notice 2 big changes to the ``MyView`` class that I've defined in awesome.coffee.
It now extends ``ModularView``, which I'll link to at the end, and it's 
**immediately** obvious what the API for this View looks like after promoting 
``globalEvents`` to being a first class citizen. The outcome? we just blew the 
readability, modularity, and testability of ``MyView`` out of the water.

Readability
-----------

Any new developer that needs to know how your View responds to global events 
simply needs to look right at the top of your declaration for ``globalEvents``
to see which ones the View has registered. This has already improved the workflow
on our team by reducing confusion, as well as getting rid of horrid documentation
in favour of people actually understanding what the View is supposed to to
simply by reading the code.

**docstring.coffee**

.. code-block:: coffeescript

    # MyView Class
    # This View is responsible for frobbing the fizzbar, as well as
    # bazarating the foonatch.
    #
    # ATTENTION: This View listens to "globalEvent" over the EventBus
    #
    class MyView extends Backbone.View
      ...


Modularity
----------

Two months down the road, you might want to implement an alternate view that 
works on the same API. An example is starting with implementing an edit form, 
and later on, it becomes a requirement to have additionally, a compact / 
simplified view to do the same operation. Better still you want to create a 
read-only view who should still respond in the same manner as the edit form, 
but display and act in a totally different way.

Using ``ModularView``, any programmer just needs to look over the 
``globalEvents`` declaration of the initial view, and ensure their new view can
respond to these same events. This makes it possible to swap out any view in 
your application ensuring that you're not forgetting pieces, or creating 
new bugs in the process.

Testability
-----------

This is one of the more exciting benefits of using ``globalEvents`` for me. Now, 
rather than writing tests that depend on other views you only need to test the 
interface that the View needs. If you're doing BDD with tools like 
`jasmine`_ and `sinon.js`_, you can write your new View's test suite first based
on the agreed upon ``globalEvents`` interface that has been defined.

**myview_spec.coffee**

.. code-block:: coffeescript

    describe "MyView", ->
      beforeEach ->
        @viewFactory = new ViewFactory

        # Create the view using the factory so it gets the pubSub.
        @view = @viewFactory.create MyView
      
      it "should be able to respond to 'globalEvent'", ->
      
        # spy on the globalEvent listener to make sure it gets called.
        eventBusSpy = sinon.spy @view, "respondToGlobal"
      
        # fire the event over eventBus from an off-view location.
        @viewFactory.eventBus.trigger "globalEvent"

        # Ensure the event was caught
        expect(eventBusSpy.calledOnce).toBeTruthy()
      
        # put the view back together.
        do eventBusSpy.restore
        

Hopefully you're as convinced as I am that this is a solid pattern for creating
badass Views in Backbone. If you're interested in seeing how the ModularView 
is implemented, I created it as a gist on GitHub: 
https://gist.github.com/1655106 . There's a Docstring on the class with a more
condense example.

I'd be happy to answer any questions in comments if you have them.

.. _`gTrax`: http://gtraxapp.com/
.. _`jasmine`: http://pivotal.github.com/jasmine/
.. _`sinon.js`: http://sinonjs.org/
.. _`Sheepdog`: http://sheepdoginc.ca
.. _`Honza`: http://honza.ca
.. _`Los Techies`: http://lostechies.com/derickbailey/2011/07/19/references-routing-and-the-event-aggregator-coordinating-views-in-backbone-js/