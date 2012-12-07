:author: Thurloat
:date: 2011-02-25 14:44:00
:layout: post
:slug: advanced-app-engine-bulk-downloading
:status: publish
:title: Advanced App Engine Bulk Downloading
:categories: Google, App Engine, Python 
:tags: appengine, bulkloader, development, python

.. image:: http://commondatastorage.googleapis.com/thurloat/appenginedump.png
    :align: center

The new Bulkloader data exporter is much easier and more automated than
the nasty old
way of exporting. As good as it is, there were a lot of additional
requirements for one export script that I was tasked to write which
looked beyond the scope of the Bulkloader YAML configuration file.
Through reading the documentation and diving into the
I was able to find some creative solutions and for the most part -- keep
using the YAML file.

The long and winding road
=========================

I'm going to go through the problems I faced, and how I was able to
solve them using primarily the YAML config file with the new Bulkloader.

CSV is out, Pipe separated is in
--------------------------------

One of the requirements of the exported data was that it wasn't
comma-separated. It needed to be delimited by a *pipe*, "**|**",
character. The options for exporting data via the Bulkloader are *CSV*,
*simpletext* and *xml*. I don't see "*PSV*" as an option.

Here's what I did to fix this:

.. code-block:: yaml

    - kind: model_name
      connector: csv
      connector_options:
        export_options:
            delimiter: "|"

I was able to re-use the Bulkloader CSV connector by passing through
additional arguments by proxy to the Python CSV module. The outcome was
a nice clean pipe separated data format. This was *much*, *much* better
than the original thought: to try and use the Python module after the
CSV connector generated the file to re-write the CSV as pipe separated.

The two parameters that you can pass to the CSV module are:
**delimeter** and **dialect**. These options can be found in the
**ConnectorSubOptions** class in

Massaging Data
--------------

There were additional data formatting changes that needed to occur to
meet the requirements of the "*PSV*" formatted file. By default, the
Bulkloader auto-generated configuration writes out some default
export\_transform properties for objects who don't convert nicely to
strings. They cover some use-cases, but not enough to help with these
requirements.

Here's a few simple examples of how I leveraged the power of lambda
expressions to massage the data into place.

Yes / No instead of True / False
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,

This wasn't a big deal, but it opened my brain to using lambdas to tweak
the exported data. It simply reads in the True / False value and outputs
a 'yes' / 'no' string as the formatted value.

.. code-block:: yaml

    - property: boolean_field
      external_name: Boolean Field
      import_transform: transform.regexp_bool('true', re.IGNORECASE)
      export_transform: "lambda x: 'yes' if x else 'no'"

Convert list to Comma-Separated String
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,

I used this one often in my exporter. It's common to want to store data
in a list but python string formats a list as *"[u'item1', u'item2']"*.
Clearly un-acceptable for a pretty data export; this new transformer
will output the list as *"item1,item2"*.

.. code-block:: yaml

    - property: list_field
      external_name: List Field
      export_transform: "lambda x: ','.join(x) if type(x) is list else x if x else ''"

Convert a list of values into single values split in Multiple Columns
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,

This example is a little more specific to my use case, however it can
still serve as a good example for having a single column in the database
act as *multiple* columns in the exported file. Here, I have a
de-normalized list of person details so I can leverage App Engine's
exact list equality matching function, for example: I'm looking for an
object who's person is "Adam", "Thurlow" or, "thurloat@gmail.com". Here,
we're separating each of those list items out into their own column.

.. code-block:: yaml

    - property: person_details
      export:
          - external_name: First Name
            export_transform: "lambda x: x[1] if type(x) is list and len(x) > 1 else ''"
          - external_name: Last Name
            export_transform: "lambda x: x[2] if type(x) is list and len(x) > 2 else ''"
          - external_name: Email
            export_transform: "lambda x: x[0] if type(x) is list and len(x) > 0 else ''"

Quoted Printables
-----------------

One of the problems discovered early on was that **db.Text** fields
longer than 80 characters ended a line with '=\\n' or '=20\\n'. The
cause of this problem is that when you POST form data to the Blobstore:
the Blobstore encodes all large text as *MIME quoted-printable*. The
simplest way that I found to get around this was to take advantage of
the python **quopri** module.

.. code-block:: yaml

    python_preabmle:
    ...
    - import: quopri
    ...

    transformers:

    - kind: model_name
      property_map:
        - property: message_body
          import_transform: db.Text
          export_transform: quopri.decode_string


De-normalizing related data
---------------------------

This was by far the biggest challenge for the exporter. Reading through
the
It's mentioned that to do more complicated things, such as adding
columns, or modifying the file in "*arbitrary*" ways you should use the
**post\_import\_function\_** option for the property. This way seemed
overly complicated, so here's how I dove into discovering the right way
to do this:

Using Django non-rel on App Engine proved ineffective when it came to
using the models with the remote\_api. In order to pull the additional
related data into the exported file, I had to re-write small portions of
my model using **google.appengine.ext.db.Model** rather than the Django
models due mostly to un-resolved imports (unless I want to pollute my
Bulkloader python\_preamble with a ton of Django imports).

Here's some *hopefully* over commented code on how I was able to do this
with the YAML configuration file.

derefr.py
,,,,,,,,,

.. code-block:: python

    # Import the App Engine DB module
    from google.appengine.ext import db

    # Skeleton App Engine compatible models
    class user(db.Model):
        id = db.IntegerProperty()

    class user_alt_info(db.Model):
        country = db.StringProperty()
        state = db.StringProperty()


    # Data Transformation functions to Output the user's related Country and State data.

    def get_user_info(user_key):
        q = db.GqlQuery("SELECT * FROM user_alt_info WHERE user_id = :1", key.id()).fetch(1)
        return q[0] if len(q) > 0 else None

    def get_user_country(user_key):
        u = get_user_info(user_key)
        return u.country if u else ""

    def get_user_state(user_key):
        u = get_user_info(user_key)
        return u.state if u else ""

bulkloader.yaml
,,,,,,,,,,,,,,,

.. code-block:: yaml

    python_preamble:
    ...
    # Import the new denormalizing script.
    - import: derefr
    ...


    - kind: user
      ...
      property_map:

        # Here, I make extra use of the __key__ property for the model. I am 
        # able to resolve references to the user_info model through using this
        # key.
        - property: __key__
          export:

            # Using the KEY as an argument, we can pull in the related values.
            - external_name: User Country
              derefr.get_user_country

            - external_name: User State
              derefr.get_user_state
      ...

That about sums it up. It ended up taking me a lot less time than
originally anticipated, and appears to be void of bugs. I contribute
this to the fact that in no way do I ever try to manually interact with
the "*PSV*" / "*CSV*" that is generated by my script.

I hope this can shed some insight into how powerful this new YAML
configuration exporter is, and help overcome the lacklustre
documentation, and examples on the Official Bulkloader project page.

Cheers!
=======
