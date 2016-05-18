# Eelzee case study: D7 Testing with Behat

Functional testing in Drupal 7 using Behat 3, Mink, Selenium, DrupalExtension, and some other homegrown tools

Speaker Notes:
This presentation is on using Behat 3 as the basis for testing a Drupal 7 installation, using Mink, Selenium, the Behat Drupal Extension, and some other home grown tools to get the testing done. This presentation does not cover installation of said tools, but I've included links at the end of the presentation to resources for that - please let me know afterwords if you need help and I'll assist if I can. This presentation will give a rough overview of the listed tools and the roles they serve, and then delve immediately into the customizations and lessons we at Eelzee found most useful for our particular setup. Not everything in this presentation is going to be generally useful to folks, I'm hoping a discussion of the journey might help those of you who are considering the same.

---------------------------------------------

## Why not Simpletest?

- [meta] Replace the testing framework with PHPUnit and possibly rewrite Simpletest1
- "Since http://drupal.org/node/1567500 Drupal 8 started to use PHPUnit as it's unit test framework. One advantage of PHPUnit is that there are tools around which support it already." - Daniel Wehner2

Speaker Notes:
The tools named on the prior page might be foreign to many here, but SimpleTest shouldn't be.  Before we start talking about what those listed tools are and how they'll help us, i should mention why we leaned away from SimpleTest.
On this slide I've got a thread title and a quote in this page, with sources footnoted below.  The first is the meta thread titled "Replace the testing framework with PHPUnit and possibly rewrite Simpletest", whose url is in the footnotes below.  The long term suggestions for this thread include using PHPUnit for unit tests, and Mink+Guzzle+Goutte for web/integration tests, and possibly Behat for  BDD tests.  This thread is the primary rationale for our choice, although it was backed up by other sources like the quote that follows it.  We adopted these tools primarily so that we can write testing that is more or less forward compatible with the future plans of Drupal.
We've been focusing entirely on the functional side of things, so I'll only be addressing that in the talk today.  I'm a long way from being able to give a talk on PHPUnit with any feeling like I'm passing on a best practice, so I'll defer that talk for others.

---------------------------------------------

##What is Behat?

(B)ehavior (D)riven (D)evelopment. Essentially, acceptance testing

<div class="flexbox-container">
    <div class="col">
        <ul>
        	<li>Uses client language to define tests</li>
        	<li>Top layer of testing pyramid</li>
        </ul>
    </div>
    <div class="col">
        <img class="right" src="img/pyramid.png" alt="Martin Fowler's testing pyramid."/>
    </div>
</div>
<footer>
    Further reading: 

    - [The top 5 challenges of BDD with Behat/Mink](http://www.ymc.ch/en/the-top-5-challenges-of-bdd-with-behatmink)
</footer>

Speaker Notes:
Behat is a Behavior Driven Development, or "BDD" framework, a type of functional or acceptance testing.  It provides a means to translate english descriptions of desirable behavior into testable php code.  This is primarily distinguished from Unit testing - where Unit testing tests only an isolated bit of functionality, such as a function or method,  functional testing tests a slice of functionality for a site, and Acceptance testing takes it further, to ensure the software properly satisfies the business requirements it was built for.  Behat most accurately falls in this last camp, because of how the tests are written
BDD tests from the perspective of the client, using the client's language as part of the test, and optimally getting the client to be part of the process of writing the tests.  The primary goal in writing tests in a BDD system like Behat is engaging with the client.  By using the client's language, it can get the client engaged in the process of defining precisely how the software should behave, and has the side effect of helping the developers better understand the client's business by learning how they describe it.
Agile software development guru Martin Fowler provides a pyramid graphic that portrays how much of each type of test should comprise one's total testing efforts.  The higher a layer resides in the pyramid, the more expensive its tests are in respect of the effort for creating and maintaining them.   Here, we see "UI" at the top of the pyramid, which translates to testing that ensures all our visual components are there correctly.  This invariably involves the use of a browser, virtual if you are lucky, real if not, and some way of running the test on said browser, automated if you are lucky, manually if not.
There are quite a few variants of the testing pyramid on the internet these days, but they all generally correspond to the notion of testing with a browser, either real or virtual, as being one of the most expensive tests to run.  When you end up with this situation, you've inverted this pyramid, ending up with an "ice cream cone" instead.

---------------------------------------------

##What is Behat?

//TODO

Speaker Notes:
The ice cream cone is a visualization of an inversion of the correctly applied testing effort - an anti-pattern.  This is in part due to the development time required to write BDD and functional tests versus unit tests, but also due to the speed of execution, possibly the requirement of licenses, and fragility. In a test-driven-development environment, developers would be executing regression test frameworks frequently, which won't happen if they take a long time to execute, and system enhancements can break these sorts of tests, which then must be rewritten.  Nevertheless, they form a necessary part of the testing structure - there's some things you just can't do without them, and they're the most easily understandable by the client. They're designed to be so.

I mentioned before that Behat serviced the UI layer of the pyramid, but it also can serve for the service layer when coupled with DrupalExtension.  This is an intermediary layer, performing testing through an API, which is part of what DrupalExtension provides.  We'll get to that in a moment.

---------------------------------------------
##What are Mink and Selenium?

- An open source browser controller/emulator for web applications
- Emulator = application that parses response of http requests
- Controller = software that controls an actual web browser
- Mink: Unifying api for emulators and controllers
- Selenium: browser controller

<footer>
Further reading:

- [Mink at a Glance](http://mink.behat.org/en/latest/at-a-glance.html
)
- [Selenium homepage](http://www.seleniumhq.org/)
</footer>

Speaker Notes: What are Mink and Selenium?   Mink is an open source browser controller/emulator for web applications.  The distinction being drawn here is between browser emulators, which request pages and process them in code (but don’t do javascript or ajax), and browser controllers, which interact with an actual browser.  Mink is meant to unify the api differences between emulators and controllers, as both will likely eventually be needed in testing.

Selenium is a browser controller leveraged by mink to do the actual driving of the browser during testing that requires javascript or ajax.  Selenium has drivers for all the major browsers, and many third party drivers as well.

---------------------------------------------

##What is DrupalExtension?

- Provides Step definitions common to Drupal testing scenarios
- Execution through:
	- Blackbox: no privileged access
	- Drush
	- DrupalApi

<footer>
Further reading: [DrupalExtension docs](http://behat-drupal-extension.readthedocs.org/en/3.1/drupalapi.html)
</footer>

Speaker Notes:
The last piece is the Behat Drupal Extension.  It’s an integration layer between Behat and Drupal.  It provides step definitions for common scenarios specific to Drupal sites.  Common actions like logging in, creating and deleting users, nodes, taxonomy terms, and so forth

The extension provides testing access via the blackbox, Drush, or Drupal drivers.  Here’s a rough summation of the differences:
The blackbox driver skips the service layer entirely, and assumes no privileged access to the site. You can run the tests on a local or remote server, but all the actions must take place by using the site’s user interface.
The Drupal API Driver is the fastest and the most powerful of the three drivers. Its biggest limitation is that the tests must run on the same server as the Drupal site.  This was not a limitation for us, so that’s mostly what I’ll be focusing on here.
The Drush API driver serves as a partial service layer implementation, but one that can’t use javascript. The main advantage of the Drush driver is that it can work when your tests run on a different server than the site being tested.

You can start functional testing by using the steps this module defines without worrying about writing your own custom step definitions.  It’s worthwhile trying to accomplish your testing simply using what is on offer there, but we’ve found that it’s best to start writing your own definitions almost immediately, as it gives you far greater control as to how your tests are written.  Before that, though, let’s look an example Behat test:

---------------------------------------------

#Behat Tests

- Stored in .feature files
- Default english in implementation

Speaker Notes: 
Behat tests are written in plain english by default, and stored in .feature files.  As I mentioned, your tests are written using the client’s language, which can form a contract that translates the expected outcome in English into verifiable code tests that are executed by a simulated or controlled browser.

Some areas of interest:
The feature section at the top here is a general descriptor of the test.  While it contains no programmatic significance, it’s going to be the best place to store what both you and your client expect out of that particular test.

The line with the ‘at-symbol’ words @api and @javascript are tags, which can serve as programmatic signifiers to Behat, or categorization for the test writer.  For example, @javascript indicates to behat to leverage selenium to control an actual browser for the test, whereas @api alone would be a signal to use the Drupal API driver to control a virtual browser with greater speed but less functionality.  There can be additional tags in there, including ones you can add yourself, like ‘@current’ or ‘@broken’, which you can use to select a particular test or set of tests within your suite.

The Scenario declaration is a more granular subsection of the Feature above it - a single test within the feature.  Programmatic state primarily exists here, within scenarios, and is cleaned up between them. Users and nodes you create, whom you’re logged in as, that sort of thing.

What follows the scenario is a series of steps, which begin with highlighted words like “Given”, “When”, and so forth.  These are the “steps” of the test, and are mapped to function invocations, defined by Behat, Mink, the Behat Drupal Extension, or by you yourself.

A step definition can be highly granular, or very broad in scope.  As far as I know, it’s technically possible have a step that says “Given that I run all the tests”, and then have a massive function block that ran all possible function tests.  I say this not to encourage you to do so, but mostly to underscore how your client’s language can be incorporated into the testing process, and how the power of Behat is going to probably lie in you embracing writing your own step definitions, rather than simply leveraging the ones provided out of the box.

Now let’s look at a context containing step definitions.

---------------------------------------------

##Behat Contexts and Step Definitions (1 of 3)

- Initialized via `behat --init`, or manually.
- Translates english to php code
- Contains step definitions OR extensions of functionality
- Comments of definitions define the english sentence they serve
- Behat provides base definitions for snippets it cannot find.


##Behat Contexts and Step Definitions (2 of 3)

- TODO: Discuss transformations


##Behat Contexts and Step Definitions (2 of 3)
###Installation demo

<div id="demo1-player"></div>

---------------------------------------------

##Custom step definitions

- Necessary to abstract testing language
- Difficult due to inability to extend (step-defining) context classes, and inter-context communication discouraged. 
- Need ability to leverage functionality of other contexts without extending

Speaker Notes:
Let’s look at that now.  We want to write custom step definitions that we know will conform to our clients’ language.  In older versions of Behat, this would be done by extending the appropriate drupal extension sub-context to get the relevant functionality for your test.

Unfortunately, we’re in a bit of a growth stage right now.  Behat 3 makes it harder to leverage the work done in the Behat Drupal Extension directly.  Subcontexts, which were part of Behat 2 and which this extension was built around, have been removed.  Now everything is a Context, but inter-context communication is not (directly) supported - more on this in a bit.  You also cannot extend a step-defining context class - the classes with the phpdoc english snippets in them?  The minute you try to, you’ll get errors from php complaining that a function is already defined.  This occurs because Behat internals add any step definition tests to the environment, and then tries to add them again with any class that extends the step-defining one.  Step-defining context classes are therefore de-facto final.  Even as much as one step in the class is enough to make it so.

Why is this a problem?  Because of shared state - the existing DrupalExtension stores references to objects that it creates during its execution, so they can be removed from the database at the completion of a scenario.  The way it has been written up to this point, however, is with subcontexts in mind.  Under the new paradigm, if a context creates a node for testing purposes, other contexts have no idea that node exists.  If another context logs in a user, that user doesn’t remain logged in for your custom context.

This would signal many folks to stick with Behat 2

<<<<<<< HEAD
The DrupalExtension author has compensated for the change by having custom context authors extend the class RawDrupalContext and RawMinkContext.  These classes only have minimal feature sets - all the power lies in DrupalContext and MinkContext, which cannot be directly extended in the new version (they contain step definitions). 

Those facts means you cannot leverage a function provided by another context without special work-arounds, which in our case were necessary.  Code efficiency dictated a need to gain access to other contexts at runtime in order to pass along test questions they are better equipped to answer.
=======
The DrupalExtension author has compensated for the change by having custom context authors extend the class RawDrupalContext and RawMinkContext.  These classes only have minimal feature sets - all the power lies in DrupalContext and MinkContext, which cannot be directly extended in the new version (they contain step definitions).   Those facts means you cannot leverage a function provided by another context without special work-arounds, which in our case were necessary.  Code efficiency dictated a need to gain access to other contexts at runtime in order to pass along test questions they are better equipped to answer.
>>>>>>> gh-pages
