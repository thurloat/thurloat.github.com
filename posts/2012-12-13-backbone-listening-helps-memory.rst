:author: Thurloat
:date: 2012-12-13 22:30:00
:layout: post
:slug: backone-listening-helps-memory 
:status: publish 
:title: Listening with Backbone helps your Memory 
:categories: Backbone.js, Coffeescript 
:tags: backbone.js, coffeescript 

`Backbone`_ 0.9.9 was `released`_ today! With it came the inclusion of a long-time
requested feature to help views manage the binding and garbage collection of
the events bound to the objects within the view. The new methods that I'm going
on about are:

- ``listenTo(object, events, callback)``: Takes the same arguments as
  Backbone's **Event.on** 
  with the addition of having the view remember which object is bound to which
  events and callbacks to unbind them later on.
- ``stopListening(object, events, callback)``: Calling it by yourself with the
  original object, events and callback will remove it from the registry, and it
  gets called for you during **remove** with no arguments
  to unbind all events for the view.

This is big news for folks developing single page applications. Especially where
you are building up and tearing down many views without page reloads. If you're not
careful about managing object event listeners, you end up with many 
detached objects that will never get garbage collected. The outcome is that
your application continues to grow in memory usage, and performance
degrades rapidly as time goes on.

Taking it Further
-----------------

Being able to ``listenTo`` events on objects is great, but we can take it
an additional step in the right direction. One of my favourite parts of
Backbone is how declarative you can keep your view classes. This makes it
immediately obvious what a view is responsible for and what it's connected to
just by reading the first few lines of code. Backbone is already set
up nicely for this with properties like ``className``, ``tag``, ``events`` like
the following simple class:

**myview.coffee**

.. code-block:: coffeescript

  class MyView extends Backbone.View
    tag: "div"
    className: "myView"
    events:
      "click a.btn": "handleLinkClick"

As I outlined above, it's being blatant about how it behaves. Now imagine that
you've planned your view out well enough that you can describe how other
objects are going to interact within it in the same form.

**myview2.coffee**

.. code-block:: coffeescript

  class MyView extends BaseView
    tag: "div"
    className: "myView"
    events:
      "click a.btn": "handleLinkClick"

    objectEvents:
      "reset mycollection": "refreshList"
      "change:field someModel": "refreshLabel"

    # ...

Pretty awesome, amirite? With this set up, it forces you to think about your
views beforehand, and doesn't let you forget about calling ``listenTo`` to
models and collections within a view thus even further protecting you and your
application from memory leaks. Here's the little snippet from my **BaseView** class to 
take care of the declarative event binding using the new ``listenTo`` method
in the latest Backbone.

**baseview.coffee**

.. code-block:: coffeescript

  class BaseView extends Backbone.View
    # Automatically hook up events in objectEvents to their respective objects
    # using the fancy new `@listenTo`.
    delegateEvents: (events) ->
      super

      @objectEvents or= {}
      for compositeValue, handler of @objectEvents
        x = compositeValue.split " "
        if x.length is 2 and @[x[1]]?
          obj = @[x[1]]
          @listenTo obj, x[0], _.bind @[handler], @
        else
          console.log "Tried to bind event #{x}, but the view doesn't have that object."

I think I can say with confidence that we're all excited about this feature
inclusion into Backbone core. Many of us have implemented this on our own, as I
have, and just this evening was able to remove from my own **BaseView**. I'm 
equally as excited for those who have not implemented this on their own and 
hope that it will help keep everyone's memory usage in check.

I highly suggest checking out this `presentation`_ by Google on finding memory
leaks in your application. This upgrade in Backbone should help get you on the
right track, however it's not going to magically solve all of your problems.

Always think about how your events are going to be unbound, and good luck with 0.9.9!

.. _`presentation`: https://docs.google.com/presentation/d/1wUVmf78gG-ra5aOxvTfYdiLkdGaR9OhXRnOlIcEmu2s/pub
.. _`released`: https://github.com/documentcloud/backbone/compare/0.9.2...0.9.9
.. _`Backbone`: http://backbonejs.com
