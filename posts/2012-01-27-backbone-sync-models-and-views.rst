:author: Thurloat
:date: 2012-01-17 20:04:00
:layout: post
:slug: backbone-sync-models-and-views
:status: publish
:title: Backbone.js - Keeping a View's model in sync
:categories: programming, backbone
:tags: coffeescript, backbone, development, javascript

I'm doing a lot of experimenting with Backbone.js for the team at
Sheepdog these days. We're working on compiling a front end stack to build
our internal and client projects that require complex front end 
interactions.

One of our "wants" for Backbone is to easily bind a Backbone.View containing
many form elements directly to a Backbone.Model instance. I wanted to implement 
this in the most declarative way possible, similar to the way a view is 
structured. The solution involved creating a subclass of Backbone.View called
FormView.

The Example
===========

To explain it, I'll go through a simple example of creating a simple Model
and View to lay it all out. I'll start by creating a model for the User with
some fields fullName, foo, and bar. 

**model.coffee**

.. code-block:: coffeescript

    class User extends Backbone.Model
      defaults:
        fullName: ''
        foo: ''
        bar: ''

Then create a View to display this new User class.

**view.coffee**

.. code-block:: coffeescript

    class UserForm extends Backbone.View
      tagName: 'div'
      
      events:
        'click #test_reset': 'resetModel'
      
      resetModel: ->
        @model.set
          foo: 'bang'
          bar: 'baz'
          fullName: 'Cool Guy'
        
      render: ->
        $(@el).html """
          <input type="text" id="test_foo" />
          <input type="text" id="test_bar" />
          <input type="text" id="test_first_name" />
          <input type="text" id="test_last_name" />
          <input type="submit" id="test_reset" value="reset" />
        """
        @
        
Now, you pass a User object into the view when you create it, and now the goal
is to have the model update each time the form is changed as well as propogate
model changes back into the form when they happen. This is where things start
to get a little ugly and WET (**!DRY**). 

You'll have to add a refresh method to your View, and bind it with 
``@model.bind 'change', => do @refresh`` and to sync in the other direction 
you will need to have  ``'change input': 'toModel'`` declaration within your 
events object for the View.

This is where you'll end up writing some ugly code like the following which
implements naive ``refresh`` and ``toModel`` methods for the example.

**view.coffee**

.. code-block:: coffeescript

    class UserForm extends Backbone.View
      
      tagName: 'div'
      
      events:
        'change input': 'toModel'
        'click #test_reset': 'resetModel'
      
      initialize: ->
        @model.bind "change", => do @refresh
        
      resetModel: ->
        @model.set
          foo: 'bang'
          bar: 'baz'
          fullName: 'Cool Guy'
          
      toModel: ->
        @model.set
          foo: $('#test_foo').val()
          bar: $('#test_bar').val()
          fullName: "#{@$('#test_first_name').val()} #{@$('#test_last_name').val()}"

      refresh: ->
        $('#test_foo').val @model.get "foo"
        $('#test_bar').val @model.get "bar"
        names = @model.get('fullName').split " "
        $('#test_first_name').val names[0]
        $('#test_last_name').val names[1]

      render: ->
        $(@el).html """
          <input type="text" id="test_foo" />
          <input type="text" id="test_bar" />
          <input type="text" id="test_first_name" />
          <input type="text" id="test_last_name" />
          <input type="submit" id="test_reset" value="reset" />
        """
        @

There are many problems with the code above for keeping the form and model
in sync with eachother. To name a few:

- It's unclear which Model fields are bound to which form elements
- Lots of repetitive code
- Refactoring becomes a nightmare, jQuery seleectors all over the code
- What if you don't want to propogate ALL changes on a model? only certain fields?
  the boiler plate for this starts growing very quickly.

Not only does the code contain the afforementioned issues, but it doesn't 
really follow the backbone paradigm of defining selectors for events
during the View's declaration.

Syncing Solution - FieldMap
===========================

The solution we (read `Honza`_ and I) came up with involves creating a 
"*fieldMap*" of input ids to model field names, and have them sync in both 
directions automatically. For the most simple case this looks similar to 
the events declaration at the top of your View class; No additional legwork 
needed.

You can see that the fullName syncing isn't in the below example, I'll show 
how that can be done just as simply leveraging a slightly more complex
fieldMap declaration.

**view.coffee**

.. code-block:: coffeescript

    class UserForm extends FormView

      tagName: 'div'
  
      fieldMap:
        '#test_foo': 'foo' # Binds #test_foo to User.foo
        '#test_bar': 'bar' # Binds #test_bar to User.bar
    
      events:
        'click #test_reset': 'resetModel'
      
      resetModel: ->
        @model.set
          foo: 'bang'
          bar: 'baz'
          fullName: 'Cool Guy'
      
      render: ->
        $(@el).html """
          <input type="text" id="test_foo" />
          <input type="text" id="test_bar" />
          <input type="text" id="test_first_name" />
          <input type="text" id="test_last_name" />
          <input type="submit" id="test_reset" value="reset" />
        """
        @


Much nicer, isn't it? When the user presses the "reset" button which is 
generated by ``render``, the ``resetModel`` method will fire to set the model's 
fields and the syncing will happen immediately to those form elements that the 
fields are bound to. Also, when a user changes the content of either textbox
the foo and bar values will get set on the View's model instance.

Now how about the complex case where the form input doesn't necessarily match 
up exactly to the data going into the Model? or if the Model's field is 
separated over multiple inputs, just like the fullName field on the User 
Model? 

Complex FieldMaps
=================

When you require a more complex mapping between the View and the Model
the fieldMap object can be defined in two ways.

**simple_map.coffee**

.. code-block:: coffeescript

    fieldMap:
      '#selector': 'fieldName'

**complex_map.coffee**      

.. code-block:: coffeescript    

    fieldMap:
      '#selector':
        field: 'fieldName'
        toModel: 'functionToSyncFieldToModel'
        toForm: 'functionToSyncModelToField'
        
To give an example of how this long form declaration is useful, and allows for
flexibility & complexity within your sync logic, I'll implement the fullName 
fieldMap for the UserForm that we've been working with all along.

.. code-block:: coffeescript

    class UserForm extends FormView

      tagName: 'div'
  
      fieldMap:
        '#test_foo': 'foo'
        '#test_bar': 'bar'
        '#test_first_name, #test_last_name':
          field: 'fullName'
          toModel:  'nameToModel'
          toForm: 'nameToForm'
    
      events:
        'click #test_reset': 'resetModel'
      
      resetModel: ->
        @model.set
          foo: 'bang'
          bar: 'baz'
          fullName: 'Cool Guy'
      
      nameToModel: ->
        # Your toModel function should simply return the value that the
        # model expects.
        "#{@$('#test_first_name').val()} #{@$('#test_last_name').val()}"
    
      startToForm: ->
        # Your toForm function should just render that value out from
        # the model into the inputs where it belongs.
        names = @model.get('fullName').split " "
        @$('#test_first_name').val names[0]
        @$('#test_last_name').val names[1]
    
      render: ->
        $(@el).html """
          <input type="text" id="test_foo" />
          <input type="text" id="test_bar" />
          <input type="text" id="test_first_name" />
          <input type="text" id="test_last_name" />
          <input type="submit" id="test_reset" value="reset" />
        """
        @

You can see this is much cleaner than our initial ``UserForm`` view containing
the full two way sync. It's now easy to tell which fields are linked to which 
model fields, as well as being able to quickly spot where any complexities lie.

If you're interested in seeing how the FormView is implemented, I created
it as a gist on GitHub: https://gist.github.com/1630584 . There's a Docstring
on the class with a similar example, but more condensed. I'd be happy to answer
any questions in comments if you have them.

.. _`Honza`: http://honza.ca
