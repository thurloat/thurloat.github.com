:author: Thurloat
:date: 2012-05-05 17:45:00
:layout: post
:slug: jackson-deserializing 
:status: publish
:title: Jackson Deserializing Polymorphic JSON
:categories: programming, jackson, JSON
:tags: JSON, jackson, java

Recently I've been spending time writing a REST API using a new stack.
Java AppEngine + Objectify using Jersey to build the API, and Jackson to 
handle (de)serialization into/outof JSON. Needless to say I was running 
into **many** problems that just don't come up in the Python world.

There was one problem that our team recently ran into that I thought I'd 
share, as it took a long time to find the correct solution and there was 
a the lack of help online in this area.

Problem
-------

Reading and writing polymorphic datatypes using Jackson. 
For example a ``List<Key<Entity>>`` at runtime can only be recognized as
``List<?>`` due to Type Erasure with Generics so Jackson can really only
try to searialize and deserialize that List as a list of POJOs. We already
have a serializer that writes out the Long value of a Key as ``"[1, 2, 3]"``.

However Jackson has no idea how to turn ``"[1, 2, 3]"`` back into a List of 
Entity Keys and complains with something along the lines of "*Jackson 
doesn't know how to turn type List<Integer> into List<Key<Entity>*".

There are other documented solutions to this problem that didn't fit.
The most common was to embed type metadata into the JSON and then tell Jackson 
about it using configuration. However after looking at the JSON users would 
have to POST to the API, this was obiously a no-start.

.. code-block:: javascript

  /** Embedded Type Information **/
  {
    "id": 1,
    "myEntities": [
      ["com.googlecode.objectify.Key<com.thurloat.models.Entity>",
        {
          "id": 2
        }
      ],
      ["com.googlecode.objectify.Key<com.thurloat.models.Entity>",
        {
          "id": 3
        }
      ],
    ]
  }
  
  /** All that should need to be posted **/
  {
    "id": 1,
    "myEntities": [2, 3]
  }


Solution
--------

The solution that we stumbled across was perfect for our problem.

First, create a simple custom deserializer. The part that clicked here and
made this problem easy to solve was realizing that I could create an 
``ObjectMapper`` class inside the deserializer and work with the data in a 
way which is familiar to me. The ObjectMapper is used extensively throughout
our test suite to read in API results and test their values using types 
rather than simply checking strings. In this case it was used to read the 
JsonParser value in as the basic type in which it is serialized 
a ``List<Integer>``, then create the Key objects out of them and return that.

.. code-block:: java

  public class EntityKeyListDeserializer extends JsonDeserializer<List<Key<Entity>>> {
    static final TypeReference<List<Long>> listType = new TypeReference<List<Long>>() {};

    @Override
    public List<Key<Entity>> deserialize(JsonParser jp, DeserializationContext arg1)
        throws IOException, JsonProcessingException {
      List<Long> simpleList = jp.readValue(listType);
      List<Key<Entity>> l = new ArrayList<Key<Entity>>();
      for (int i = 0; i < simpleList.size(); i++) {
        l.add(new Key<Entity>(Entity.class, simpleList.get(i));
      }
      return l;
    }
  }
  
Then you simply add the ``@JsonDeserialize`` annotation to the model attribute
which needs it and Jackson will go back to working flawlessly.

.. code-block:: java

  public class OtherEntity {
      @Id 
      private Long id;

      @JsonDeserialize(using = EntityKeyListDeserializer.class)      
      private List<Key<Entity>> myEntities;
  }
  
Boom goes the dynamite.
