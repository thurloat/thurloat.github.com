:author: Thurloat
:date: 2012-11-10 17:30:00
:layout: post
:slug: mocha-phantomjs-jasmine-reporting
:status: publish 
:title: Mocha Style Reporters in PhantomJS with Jasmine
:categories: Javascript, Testing, PhantomJS, Jasmine
:tags: javascript, testing

Everyone and their *cat* knows that the test reporters in `Mocha`_ are much 
more nicely put together than those in `Jasmine`_. However, in the latest 
version of `PhantomJS`_ (1.7) there is a new `API`_ that allows Jasmine console
runners to catch up. I got tired of watching a slew of bland and illegible 
``console.log`` messages after every code change, so I went ahead and 
implemented the infamous `nyan reporter`_ that comes built into Mocha using the
new API methods.

.. image:: http://f.cl.ly/items/1C1r2K1I35361a1j300o/Screen%20Shot%202012-11-30%20at%2012.08.59%20PM.png
    :align: center
    :alt: Jasmine Nyancat Runner

The general approach is to have your test runner within the headless browser
make a call out to phantom with ``window.callPhantom`` and use it as a sort of
message bus to report progress within the test reporter. And within your
PhantomJS executed Javascript, hook onto ``page.onCallback`` and pass some parameterized
objects around to notify Phantom of the child page's state.

Jasmine Reporter
----------------

Here's a stripped down version of what I have in place to do the console
reporting to help illustrate the new API methods. 

**new-console-jasmine-reporter.js**

.. code-block:: javascript

  // A simple jasmine reporter that will report over the `callPhantom` API in
  // the new PhantomJS 1.7 release.
  NewConsoleReporter = function() { };
  NewConsoleReporter.prototype = {
    reportRunnerStarting: function(runner) {
      // Announce that the suite is starting by letting the runner know how
      // many specs are going to be run.
      this.tellPhantom({"type":"init", "payload": runner.specs().length});
    },
    reportSpecResults: function(spec) {
      // ... collect the spec results

      // pass along the small payload of information to phantom so the test
      // reporter can correctly display whatever it needs to.
      this.tellPhantom({
        "type":"spec",
        "payload":{
          "didFail": spec.didFail,
          "suite": spec.suite.description,
          "description": spec.description,
          "numFailures": failures,
          "failureMessage": consoleFailure 
        }});
    },
    reportRunnerResults: function(runner) {
      // Announce that the test suite is finished and that phantom should exit.
      this.out({"type":"results", "payload": {}});
    },
    tellPhantom: function(message) {
      if (window.callPhantom) {
        window.callPhantom(message);
      }
    }
  };
  jasmine.NewConsoleReporter = NewConsoleReporter;

This ``NewConsoleReporter`` object is structured to plug directly into Jasmine
using it's default reporter interface. You can create it the same way that you
would a normal ``HtmlReporter`` or ``JasmineXmlReporter`` with the Jasmine
environment. If you open the page in a browser, you're not going to see
anything at all... so let's move onto the Phantom implementation.

**Hacker Note** - You should keep the size of the message you're passing to
``window.callPhantom`` as small as possible. Passing large objects (such as
jasmine spec or suite objects) causes serious performance issues. Try and keep
your payload as small collections of booleans, strings and integers.

PhantomJS Runner
----------------

The phantom code is pretty simple since I have distilled out most of the 
``console.log`` hacks which I had to do to animate the cat. Writing a basic 
PhantomJS script that builds off the Jasmine reporter above would look something 
like the following.

**nyancat-jasmine-phantom.js**

.. code-block:: javascript

    var page = require("webpage").create();
  
    // supress all the stdout messages from the runner so our reporter looks
    // pretty even when tests fail. 
    page.onConsoleMessage = function(msg, source, linenumber) { };
    page.onError = function(msg, trace) { };

    // ascii control character to clear the screen so you can write fresh. 
    var cls = function() {return '\u001b[2J\u001b[1;3H';};
   
    // keep some state around how the test suite is doing for future reporting.
    var ran = 0,
      totalSpecs = 100,
      totalFailures = 0,
      failureMessages = [],
      cols = 80;
      
    // Here's the new `experimental` API.
    page.onCallback = function(callbackData){
      var pld = callbackData["payload"];
      switch(callbackData["type"]){
        case "init":
          // capture the total number of tests that are going to run in the
          // suite.
          totalSpecs = pld;
          break;

        case "spec":
          // increment the number of tests that have been run
          ran++;

          // keep track of how many assertions have failed for the completed
          // spec.
          totalFailures += pld.numFailures;

          // aggregate the failure messages so they can be shown at the end.
          if (pld.didFail) {
            failureMessages.push(pld.failureMessage);
          }
          
          // draw the cat that poops rainbows ...
          var output = cls();
          output += "Running Unit Tests\n";
          output += "Running: " + ran + " of " + totalSpecs + ". Failures: " + totalFailures;
          console.log(output);
          break;

        case "results":
          setTimeout(function(){
            // print all of the test failures after the suite has finished 
            // running
            if(totalFailures > 0){
              console.log("\033[1;31mFailures: " + stopColor);
              for(var i=0; i < failureMessages.length; ++i){
                console.log(failureMessages[i]);
              }
              // ugly setTimeouts are just to make sure STDOUT is flushed
              // before the phantom process exits.
              setTimeout(function(){
                phantom.exit(totalFailures > 0 ? 1 : 0);
              }, 250);
            }
          }, 250);
          break;
      }
    };

    // Open phantom to the provided test runner.
    var htmlrunner = phantom.args[0],
      pwd = fs.workingDirectory;
    page.open("file://localhost/" + pwd + "/" + htmlrunner, function(status) {
      if (status != "success") {
        console.log("phantomjs> Could not load '" + htmlrunner + "'.");
        phantom.exit(1);
      }
    });

Above you can see the ``page.onCallback`` event that gets fired when the child
page calls out with ``window.callPhantom`` and how it can be leveraged to make
more advanced (and realtime) test runners than what is currently available in 
projects like `phantomjs-jasminexml`_.

I sincerely hope that some motivated folks get on this bandwagon and create
some nice looking console test runners for Jasmine and PhantomJS. I am working on
cleaning up the Nyancat runner that I created as a more detailed starting point
and if you need some motivation -- have a look at the `Mocha reporters`_ section 
of the documentation.

Good Luck!

.. _`phantomjs-jasminexml`: https://github.com/detro/phantomjs-jasminexml-example
.. _`API`: https://github.com/ariya/phantomjs/wiki/API-Reference
.. _`Jasmine`: http://pivotal.github.com/jasmine/
.. _`Mocha`: http://visionmedia.github.com/mocha/
.. _`PhantomJS`: http://phantomjs.org/ 
.. _`nyan reporter`: http://visionmedia.github.com/mocha/#nyan-reporter
.. _`Mocha reporters`: http://visionmedia.github.com/mocha/#reporters
