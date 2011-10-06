:author: Thurloat
:date: 2010-06-07 21:08:28
:layout: post
:slug: google-storage-and-app-engine
:status: publish
:title: Google Storage for Developers on App Engine Python
:categories: Programming, Google App Engine
:tags: appengine, development, Google Storage, python


I've been using `Google Storage <http://code.google.com/apis/storage/>`_ 
since I/O 2010, and I'm impressed with it so far. At `work <http://sheepdoginc.ca>`_,
we've converted some of our application resources from using S3 as a CDN to
Google Storage this past week.

Being early adopters has its downfalls for sure; some of which usually
include system instability, a lack of published tools, and proper or 
extensive documentation. I'll try to help address the latter in this 
short tutorial. 

Configuring Google Storage for App Engine
=========================================

This problem has come up a couple of times in the Google Storage for Developers
group. After looking at the 
`Python Library Docs for GS <http://code.google.com/apis/storage/docs/gspythonlibrary.html>`_, 
the biggest hurdle looks like getting the boto library properly configured. 
Once boto is set up, you should be able to run all of the sample code provided by 
Google. *Lets get you there*.

The Problem
###########

.. code-block:: python

    # Bring in the boto library
    import boto
    
    # Load the configuration variables from your .boto file
    config = boto.config
    
    """ FAIL. """

As you can see by my fail comment, App Engine has no way to access your
``~/.boto`` file where your Secret Key is located. We can solve this by
diving into boto.config. When the boto configuration is initialized, It
checks for the initial file and tries to parse it and load it into
memory by using ``add_section(section_name)`` and
``set(section_name,key,value)``. Knowing this, you're able to call
those functions yourself after loading the boto lib and it runs
flawlessly. It only took a little bit of hacking around to find a
solution, but it works great for me. I'd love to hear your experience,
or how you got it rolling differently.

My Solution
###########

pay close attention to the 3 lines following *config = boto.config*

.. code-block:: python

    """"
    Python App Engine + Google Storage.
    Based heavily from examples given by Google
    Instructions:
    just rip the boto lib from the gsutil project, drop it in your python 
    App Engine project. 
    @author: Adam Thurlow <Thurloat>
    """"
    
    import boto
    from google.appengine.ext import webapp
    
    config = boto.config
    config.add_section('Credentials')
    config.set('Credentials', 'gs_access_key_id', 'YOURACCESSKEY')
    config.set('Credentials', 'gs_secret_access_key', 'YOURSECRETKEY')
    
    FLAGS = ['CREATE', 'LIST_BUCKETS', 'LIST_CONTENTS','DOWNLOAD_FILE']
    DOWNLOAD_PATH = '/path/to/file.ext'
    DOWNLOAD_FILENAME = 'file.ext'
    BUCKET_NAME = 'testbucket'
    
    class DownloadHandler(webapp.RequestHandler):
    
        def get(self):
            srow = self.response.out.write
            if 'DOWNLOAD_FILE' in FLAGS:
                uri = boto.storage_uri("%s/%s" % (BUCKET_NAME, DOWNLOAD_PATH), 'gs')
                self.response.headers['Content-Type'] = 'application/octet-stream'
                self.response.headers.add_header('content-disposition', 
                    'attachment', 
                    filename = DOWNLOAD_FILENAME)
                srow(uri.get_contents_as_string())
                
    class MainHandler(webapp.RequestHandler):
    
        def get(self):
            srow = self.response.out.write
            srow("<h4> Testing the Google Storage API </h4>")
            if 'CREATE' in FLAGS:
                srow("<h2>Creating Bucket</h2>")
                uri = boto.storage_uri(BUCKET_NAME, 'gs')
                uri.create_bucket()
            if 'LIST_BUCKETS' in FLAGS:
                srow("<h2>Listing Buckets</h2>")
                uri = boto.storage_uri('', 'gs')
                buckets = uri.get_all_buckets();
                for bucket in buckets:
                    self.response.out.write(bucket.name + '<br />')
            if 'LIST_CONTENTS' in FLAGS:
                srow("<h2>Listing Contents of %s bucket</h2>" % BUCKET_NAME)
                uri = boto.storage_uri(BUCKET_NAME, 'gs')
                objs = uri.get_bucket()
                for obj in objs:
                    srow("gs://%s/%s <br />" % (BUCKET_NAME,obj.name))
                    
    _URLS = [
        ('/', MainHandler),
        ('/download',DownloadHandler),
    ]
    
    def main(argv):
        application = webapp.WSGIApplication(_URLS, debug = True)
        wsgiref.handlers.CGIHandler().run(application)
        
    if __name__ == '__main__':
        main(sys.argv)
