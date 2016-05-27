# Eelzee case study: D7 Testing with Behat

Functional testing in Drupal 7 using Behat 3, Mink, Selenium, DrupalExtension, and some other homegrown tools

<aside class="notes" data-markdown>
This presentation is on using Behat 3 as the basis for testing a Drupal 7 installation, using Mink, Selenium, the Behat Drupal Extension, and some other home grown tools we built to streamline the testing process. 

This presentation briefly covers installation of said tools.  I've also included links throughout the presentation to resources for that. I'll be giving a quick overview of the listed tools and the roles they serve, and then I'll talk about the customizations and lessons we at Eelzee found most useful for our particular setup. I've tried to limit this talk to elements of our experience that will be generally useful. I'm hoping a discussion of our journey might help those of you who are considering the same.
</aside>

---------------------------------------------
## Why not Simpletest?

- [meta] Replace the testing framework with PHPUnit and possibly rewrite Simpletest1
- "Since http://drupal.org/node/1567500 Drupal 8 started to use PHPUnit as it's unit test framework. One advantage of PHPUnit is that there are tools around which support it already." - Daniel Wehner2

<footer>
<p>Sources:

	<ul><li>https://www.drupal.org/node/1567500</li>
		<li>https://blog.erdfisch.de/2013/04/run-phpunit-drupal-8-phpstorm</li>
	</ul>
	</p>
</footer>

<aside class="notes" data-markdown>
The tools I rattled off on the prior page might be foreign to many here, but SimpleTest shouldn't be.  Before we start talking about what those listed tools are and how they'll help us, I should mention why we leaned away from SimpleTest.

On this slide I've got a thread title and a quote in this page, with sources footnoted below.  The first is the meta thread titled "Replace the testing framework with PHPUnit and possibly rewrite Simpletest", whose url is in the footnotes below.  The long term suggestions for this thread include using PHPUnit for unit tests, and Mink+Guzzle+Goutte for web/integration tests, and possibly Behat for  BDD tests.  This thread is the primary rationale for our choice, although it was backed up by other sources like the quote that follows it.  We adopted these tools primarily so that we can write testing that is more or less forward compatible with the future plans of Drupal.

We've been focusing entirely on the functional side of things, so I'll only be addressing that in the talk today.  I'm a long way from being able to give a talk on PHPUnit with any feeling like I'm passing on a best practice, so I'll defer that talk for others.
</aside>

---------------------------------------------

#Concepts

---------------------------------------------
##What is Behat

- (B)ehavior (D)riven (D)evelopment. Essentially, acceptance testing

<div class="flexbox-container">
    <div class="col">
        <ul>
        	<li>Uses client language to define tests</li>
        	<li>Top layer of testing pyramid</li>
        </ul>
    </div>
    <figure class="col polaroid">
		<img src="img/pyramid.png" alt="Martin Fowler's testing pyramid."/>
		<figcaption>Martin Fowler's Testing Pyramid</figcaption>
	</figure>
</div>
<footer>
    Further reading:

    - [The top 5 challenges of BDD with Behat/Mink](http://www.ymc.ch/en/the-top-5-challenges-of-bdd-with-behatmink)
</footer>

<aside class="notes" data-markdown>
Behat is a Behavior Driven Development, or "BDD" framework, a type of functional or acceptance testing.  It provides a means to translate english descriptions of desirable behavior into testable php code.  This is primarily distinguished from Unit testing - where that would test only an isolated bit of functionality, such as a function or method,  Functional Testing tests a slice of functionality for a site, and Acceptance Testing takes it further, to ensure the software properly satisfies the business requirements it was built for.  Behat most accurately falls in this last camp, because of how the tests are written.

BDD tests from the perspective of the client, using the client's language as part of the test, and optimally getting the client to be part of the process of writing the tests.  The primary goal in writing tests in a BDD system like Behat is engaging with the client.  By using the client's language, it can get the client engaged in the process of defining precisely how the software should behave, and has the side effect of helping the developers better understand the client's business by learning how they describe it.

Agile software development guru Martin Fowler provides a pyramid graphic that portrays how much of each type of test should comprise one's total testing efforts.  The higher a layer resides in the pyramid, the more expensive its tests are in respect of the effort for creating and maintaining them.   Here, we see "UI" at the top of the pyramid, which translates to testing that ensures all our visual components are there correctly.  This invariably involves the use of a browser, virtual if you are lucky, real if not, and some way of running the test on said browser, automated if you are lucky, manually if not.
</aside>


##What is Behat
###The functional testing anti-pattern

- Inversion of testing efforts: Too much effort in functional testing
	- Functional tests are fragile
	- Testing speed is much slower

<figure class="col polaroid">
	<img class="rot180" src="img/pyramid.png" alt="Martin Fowler's testing pyramid."/>
	<figcaption>Testing Pyramid Anti-Pattern</figcaption>
</figure>

<aside class="notes" data-markdown>
There are quite a few variants of the testing pyramid on the internet these days, but they all generally correspond to the notion of testing with a browser, either real or virtual, as being one of the most expensive tests to run.  When you end up with this situation, you've inverted this pyramid, ending up with an "ice cream cone" instead.

The ice cream cone is a visualization of an inversion of the correctly applied testing effort - an anti-pattern.  This is in part due to the development time required to write BDD and functional tests versus unit tests, but also due to the speed of execution, possibly the requirement of licenses, and fragility. In a test-driven-development environment, developers would be executing regression test frameworks frequently, which won't happen if they take a long time to execute, and system enhancements can break these sorts of tests, which then must be rewritten.  Nevertheless, they form a necessary part of the testing structure - there's some things you just can't do without them, and they're the most easily understandable by the client. They're designed to be so.

I mentioned before that Behat serviced the UI layer of the pyramid, but it also can serve for the service layer when coupled with DrupalExtension.  This is an intermediary layer, performing testing through an API, which is part of what DrupalExtension provides.  We'll get to that in a moment.
</aside>

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

<aside class="notes" data-markdown>
What are Mink and Selenium?   Mink is an open source browser controller/emulator for web applications.  The distinction being drawn here is between browser emulators, which request pages and process them in code (but don't do javascript or ajax), and browser controllers, which interact with an actual browser.  Mink is meant to unify the api differences between emulators and controllers, as both will likely eventually be needed in testing.

Selenium is a browser controller leveraged by mink to do the actual driving of the browser during testing that requires javascript or ajax.  Selenium has drivers for all the major browsers, and many third party drivers as well, although you may have to download and install some of those drivers separately.
</aside>

---------------------------------------------
##What is DrupalExtension?

- Provides Step definitions common to Drupal testing scenarios
- Execution through:
	- Blackbox: no privileged access
    - DrupalApi: most powerful.  Same machine
	- Drush: partial service layer, different machines possible.

<footer>
Further reading:
- [DrupalExtension docs](http://behat-drupal-extension.readthedocs.org/en/3.1/drupalapi.html)
</footer>

<aside class="notes" data-markdown>
The last piece is the Behat Drupal Extension.  It's an integration layer between Behat and Drupal.  It provides step definitions for common scenarios specific to Drupal sites.  Common actions like logging in, creating and deleting users, nodes, taxonomy terms, and so forth

The extension provides testing access via the blackbox, Drush, or Drupal drivers.  Here's a rough summation of the differences:
The blackbox driver skips the service layer entirely, and assumes no privileged access to the site. You can run the tests on a local or remote server, but all the actions must take place by using the site's user interface.

The Drupal API Driver is the fastest and the most powerful of the three drivers. Its biggest limitation is that the tests must run on the same server as the Drupal site.  This was not a limitation for us, so that's mostly what I'll be focusing on here.
The Drush API driver serves as a partial service layer implementation, but one that can't use javascript. An advantage of the Drush driver is that it can work when your tests run on a different server than the site being tested, but still offers some of the Drupal API functionality.

You can start functional testing by using the steps this module defines without worrying about writing your own custom step definitions.  It's worthwhile trying to accomplish your testing simply using what is on offer there, but we've found that it's best to start writing your own definitions almost immediately, as it gives you far greater control as to how your tests are written.  Before that, though, let's look an example Behat test:
</aside>

---------------------------------------------
#Behat Feature Examples


##Feature Examples

- Stored in .feature files
- Default english in implementation

<aside class="notes" data-markdown>
Behat tests are written in plain english by default, and stored in .feature files.  As I mentioned, your tests are optimally written using the client's business terminology, which can form a contract that translates the expected outcome in English into verifiable code tests that are executed by a simulated or controlled browser.
</aside>


##Feature Examples

<pre><code class="gherkin" data-trim>
Feature: this test is to simply to get a single working feature.
	In order to create a basis to work from
	As a system developer
	I want to have a standard by which I can draw from for future reference.

	@api
	Scenario:	Get one working test
		Given I am logged in as an "authenticated user"
		And I am on "/"
		Then I should not see the text "Login Required"
</code></pre>

<aside class="notes" data-markdown>
Here is a very basic example of what a feature file might look like.  Some areas of interest:

The feature section at the top here is a general descriptor of the test.  While the words in the description have no programmatic significance, it's going to be the best place to store what both you and your client expect out of that particular test.

The word 'api' with the at symbol in front of it is a tag, which can serve as run-time signifiers to Behat, or as categorization for the test writer.  For example, @javascript indicates to Behat to leverage Selenium to control an actual browser for the test, whereas @api alone would be a signal to use the Drupal or Drush API driver to control a virtual browser with greater speed but less functionality.  There can be additional tags in there, including ones you can add yourself, like '@current' or '@broken', which you can use to select a particular test or set of tests within your suite.

The Scenario declaration is a more granular subsection of the Feature above it - a single test within the feature.  Programmatic state primarily exists here, within scenarios, and is cleaned up between them. Users and nodes you create, whom you're logged in as, that sort of thing.

Below the scenario description is a series of steps, which begin with highlighted words like "Given", "When", and so forth.  These are the "steps" of the test, and are mapped to function invocations, defined by Behat, Mink, the Behat Drupal Extension, or by you yourself.
</aside>


##Feature Examples

<pre><code class="gherkin" data-trim>

Feature: this test is to show the usage of Scenario Outlines
	In order to create a basis to work from
	As a system developer
	I want to have an example of using Behat Scenario Outlines.

	@api
	Scenario Outline: User Role authentication
	  Given I am logged in as a "&lt;role&gt;"
	  When I am on "&lt;path&gt;"
	  Then the response status code should be &lt;status_code&gt;

	  Examples:
	    | role                       | path             | status_code |
	    | administrator              | /admin/config    | 200         |
	    | content creator            | /admin/config    | 403         |
	    | authenticated user         | /admin/config    | 403         |

</code></pre>

<footer>
<p>Further reading:
<ul><li>[Behat scenario outlines](http://docs.behat.org/en/v3.0/guides/1.gherkin.html#scenario-outlines)
	</li><li>[Behat backgrounds](http://docs.behat.org/en/v3.0/guides/1.gherkin.html#backgrounds)</li></ul></p>
</footer>

<aside class="notes" data-markdown>
Here is another feature example.  This demonstrates the use of Behat Scenario outlines, which is one of the ways Behat allows efficient testing.  Most of this is the same as before, with the exception of the "Scenario Outline" keywords, which trigger Behat to look for the table "Examples".   I've got three columns here, with the first row being a header row corresponding to the variable names used in the scenario outline text.

Behat considers each row in this table to be a separate scenario.  This means you cannot create a structure in one row, and expect it to be there in the next.  If you need this functionality, Behat supports something called "Backgrounds" in features, for taking actions that live across all scenarios in a feature.  I've linked to that, and the Scenario Outline docs in the footer.

There's more examples of feature construction in the documentation, but for now, I'd like to move on now to which ones we have available out of the box, and how these are defined.
</aside>

---------------------------------------------
#Installation and Configuration


##Demo
<div id="asciicast-install"></div>

<!-- <iframe width="560" height="315" data-src="https://www.youtube.com/embed/X4_ctRB3ipk?rel=0" frameborder="0" allowfullscreen></iframe> -->

<footer>
- [DrupalExtension installation docs](https://behat-drupal-extension.readthedocs.io/en/3.1/localinstall.html)
</footer>

<aside class="notes" data-markdown>
The first step of the install process is relatively straightforward.  You create a folder, install composer (either locally to the folder, or in the case of this video, globally), add a composer.json file that requires drupalextension, and run composer install.
</aside>


##Configuration file

Add the following behat.yml file:

<pre><code class="yaml" data-trim data-noescape>
default:
  suites:
    default:
      contexts:
        - FeatureContext
        - Drupal\DrupalExtension\Context\DrupalContext
        - Drupal\DrupalExtension\Context\MinkContext
        - Drupal\DrupalExtension\Context\MessageContext
        - Drupal\DrupalExtension\Context\DrushContext
  extensions:
    Behat\MinkExtension:
      goutte: ~
      selenium2: ~
      base_url: http://drupal.dev
    Drupal\DrupalExtension:
      blackbox: ~
    <mark>
    #api_driver: one of 'drush', or 'drupal'
      api_driver: drupal
      drupal:
        drupal_root: /Users/aronbeal/Sites/drupal
    </mark>
</code></pre>

<aside class="notes" data-markdown>
After the install completes, they ask you to add a behat.yml file with the contents shown here, and reconfigure the base_url to point to your local drupal installation.  I've slightly modified what they provide in the documentation, by adding the section at the bottom for 'api_driver', which with the setting shown here, will invoke the Drupal API driver for scenarios with the @api tag.  I've also added the 'drupal' and 'drupal_root' settings pointing to the filesystem path of the instance I'm going to test.
</aside>


<div id="asciicast-init"></div>
Run `./bin/behat --init`

<footer>
- [DrupalExtension installation docs](https://behat-drupal-extension.readthedocs.io/en/3.1/localinstall.html)
</footer>

<aside class="notes" data-markdown>
Once that is done, just run the above 'behat init' command from the folder where this is all installed to, and it'll generate a FeatureContext file for you, under the features/bootstrap subfolder.  We'll talk about that context file in detail in a bit, but since we're talking about it, I'll just mention that without the drupal_root declaration, you'll probably end up extending the wrong base class.

Let's look at what's become available to us at this point without doing any further configuration.
</aside>

---------------------------------------------
##Determining available steps
<div id="asciicast-show-commands"></div>
- `./bin/behat -di`

<aside class="notes" data-markdown>
Running the command at the top of the page here, you'll get a great deal of output.  The output is a list of the step definitions that are provided out of the box by DrupalExtension for you to use in your scenarios.

Each line starting with default is a step definition. This is the crux of what DrupalExtension does for us - it provides these definitions, along with an API substructure to create more. The parts containing colon prefixes are variables, and parentheticals are optional portions of text.  Only the first line is the step text - the subsequent lines are simply the description of the operation.
As you can see, there are quite a few commands right out of the box.
</aside>

---------------------------------------------
#Our first feature file


Add the following to your 'features' folder, as a file with a `.feature` suffix:
<pre><code class="gherkin">
Feature: this test is to simply to get a single working feature.
	In order to create a basis to work from
	As a system developer
	I want to have a standard by which I can draw from for future reference.

	Scenario:	Get one working test
		Given I am logged in as an "authenticated user"
		And I am on "/"
		Then I should not see the text "Login Required"
	</code></pre>

<aside class="notes" data-markdown>
Let's get started and create a simple Behat feature.  Behat knows out of the box to look for this within the feature folder it created earlier, so for now, if you're repeating this later, you should add it there.  Doesn't matter what you call it, as long as it ends with dot feature.

We save that, and run it.
</aside>


##<span class="fail">Our first feature file failure</span>
<div id="asciicast-feature1-fail"></div>

<aside class="notes" data-markdown>
We get back the above output. What went wrong?  Well, the default configuration in behat.yml gives you the configuration assuming you're going to do black box testing.  Our scenario is set up for that, but black box can't do very much.  I can't tell you its limits precisely, because we moved on from it almost immediately.
</aside>


##Add the @api tag to engage the api driver

<pre><code class="gherkin" data-trim data-noescape>

Feature: this test is to simply to get a single working feature.
	In order to create a basis to work from
	As a system developer
	I want to have a standard by which I can draw from for future reference.

	<mark class="fragment">@api</mark>
	Scenario:	Get one working test
		Given I am logged in as an "authenticated user"
		And I am on "/"
		Then I should not see the text "Login Required"

</code></pre>

<aside class="notes"
	We reconfigure our test, adding the @api tag.  This signifies that we are wanting to use the drush or drupal api driver instead of Blackbox.  We run it again.</aside>


##<span class="pass">Feature: PASSED</span>

<div id='asciicast-feature1-pass'></div>

<footer>
Further reading: [Our current behat.yml file](https://bitbucket.org/eelzeedev/testing/src/ab82ffce4105a71d5790927e09d247ce78888dc4/behat/default.behat.yml?at=master&fileviewer=file-view-default)
</footer>

<aside class="notes" data-markdown>
Running our feature now, after adding the api flag, and properly setting the drupal_root in the DrupalExtension section, and the base_url in the MinkExtension section, we see that it has finally passed muster.
</aside>


##Reverse condition to fail intentionally

<pre><code class="gherkin" data-trim data-noescape>

Feature: this test is to simply to get a single working feature.
	In order to create a basis to work from
	As a system developer
	I want to have a standard by which I can draw from for future reference.

	@api
	Scenario:	Get one working test
		Given I am logged in as an "authenticated user"
		And I am on "/"
		Then I should <mark class='fragment fade-out'>not</mark> see the text "Login Required"

</code></pre>

<aside class="notes" data-markdown>
Just to show that this is actually doing something, let's reverse the condition.  We'll remove the 'not' from the above test [click], so we are testing something that should be false.
</aside>


<div id='asciicast-feature1-fail2'></div>
<footer>
Further reading: [Behat full config example](https://gist.github.com/aronbeal/49e66e5865ac80677751138668834cb1)
</footer>

<aside class="notes" data-markdown>
And ... wait for it...

There we go, failing as expected.

There are going to be multiple hiccups for you in the early stages here while you write tests.  The presentation will be linked to later - I'll leave you with the configuration file linked in the footer.  That file that serves as the basis of what we're currently using, which I hope will be useful as a reference of a known good value when you're monkeying with this stuff.
</aside>

---------------------------------------------
#Behat Context files

<aside class="notes" data-markdown>
At this point, we've installed the drupal extension, pointed it at our drupal installation, and run a feature test based on the step definitions offered by the Behat Drupal Extension.  I'd like to move now into the classfiles that are responsible for performing the translation between the text you've seen and the executable PHP code.
</aside>


<pre><code class="php full" data-trim>
use Drupal\DrupalExtension\Context\RawDrupalContext;
use Behat\Behat\Context\SnippetAcceptingContext;

/**
 * Defines application features from the specific context.
 */
class FeatureContext extends RawDrupalContext implements SnippetAcceptingContext {

  /**
   * Initializes context.
   *
   * Every scenario gets its own context instance.
   * You can also pass arbitrary arguments to the
   * context constructor through behat.yml.
   */
  public function __construct() {
  }

}

</code></pre>

<aside class="notes" data-markdown>
First, we'll look at the Context file generated for us when we executed "behat init".  You can see here that the file extends a class called RawDrupalExtension.  If you see something else instead, you've called init without proper settings in your behat.yml file, or possibly without a working Drupal installation installed at the drupal_root config location.  You should delete the entire features folder, fix your config, and run the init step again.

 This file by itself doesn't actually have any step definitions yet. We'll add one now, but we're going to use Behat to help us with the function signature.
 </aside>


##New step: Lipsum article
<pre><code class="gherkin" data-trim data-noescape>
Feature: this test is to simply to get a single working feature.
	In order to create a basis to work from
	As a system developer
	I want to have a standard by which I can draw from for future reference.

	@api
	Scenario:	Create a new feature
	<mark class="fragment">Given I have a lipsum article</mark>
</code></pre>

<aside class="notes" data-markdown>
Here's our modified feature.  I've removed the login test for clarity, and added the line [click] "Given I have a lipsum article".  We're going to use this to create an article with preformatted content, presuming it's the sort of thing we need a lot in our testing.
</aside>


Executing provides the method signature:

<pre><code class="bash" data-trim data-noescape>
Arons-MacBook-Air:my_behat_test aronbeal$ ./bin/behat features/my_second_feature.feature
Feature: this test is to simply to get a single working feature.
  In order to create a basis to work from
  As a system developer
  I want to have a standard by which I can draw from for future reference.

  @api
  Scenario: Create a new feature  # features/my_second_feature.feature:7
    Given I have a lipsum article

1 scenario (1 undefined)
1 step (1 undefined)
0m0.44s (26.35Mb)
<mark>
--- FeatureContext has missing steps. Define them with these snippets:

    /**
     * @Given I have a lipsum article
     */
    public function iHaveALipsumArticle()
    {
        throw new PendingException();
    }
</mark>
</code></pre>

<aside class="notes" data-markdown>
Running this new scenario, I get the output shown here.  We see that Behat has noticed we have an undefined step, and has [click] given us a basic method signature that throws a Behat-specific exception by default, indicating we've got unfinished business in the method body.</aside>


Copy generated method into context file...
<pre><code class="php" data-trim data-noescape>
/**
 * @file
 */

use Behat\Behat\Context\SnippetAcceptingContext;
use Drupal\DrupalExtension\Context\RawDrupalContext;

/**
 * Defines application features from the specific context.
 */
class FeatureContext extends RawDrupalContext implements SnippetAcceptingContext {

  /**
   * Initializes context.
   *
   * Every scenario gets its own context instance.
   * You can also pass arbitrary arguments to the
   * context constructor through behat.yml.
   */
  public function __construct() {

  }
  <mark>
  /**
   * @Given I have a lipsum article
   */
  public function iHaveALipsumArticle() {

    throw new PendingException();
  }</mark>
}
</code></pre>

<aside class="notes" data-markdown>
We copy that signature over to our FeatureContext class.  If we run the test again now...
</aside>


TODO: indicates test has throw a PendingException
<pre><code class="bash full" data-trim data-noescape>
Feature: this test is to simply to get a single working feature.
  In order to create a basis to work from
  As a system developer
  I want to have a standard by which I can draw from for future reference.

  @api
  Scenario: Create a new feature  # features/my_second_feature.feature:7
    Given I have a lipsum article # FeatureContext::iHaveALipsumArticle()
    <mark>TODO: write pending definition</mark>

1 scenario (1 pending)
1 step (1 pending)
0m0.40s (26.40Mb)
	</code></pre>

<aside class="notes" data-markdown>
We'll see output indicating that the step was found, and is in a "TODO" status.
</aside>

---------------------------------------------
#What's next
##Shortcomings, customizations, and future directions

<aside class="notes" data-markdown>
That's the bare bones of how all this works.  I'm going to need to step it up a notch in terms of complexity in order to talk about the next section.  It's unfortunately unavoidable - I need to talk about the file heirarchy, some optimizations we've done at our shop to make testing easier and more powerful, and some of the pain points with this tool, especially in terms of Behat 3.  Remember, I'll have this presentation online for revisiting later.
</aside>

---------------------------------------------
##Pain point #1
###Lack of inter-context communication

<aside class="notes" data-markdown>
When you start digging into this, you're probably going to hit the same issue we did fairly quickly - that of Context files not being able to communicate with each other, and lack of shared state.

Let's look at an example of that first point.
	</aside>

vendor/drupal/drupal-extension/src/Drupal/DrupalExtension/Context/:
- RawDrupalContext.php
- <mark>DrupalContext.php</mark>:

<pre><code class="php" data-trim>

/**
* @Given I am an anonymous user
* @Given I am not logged in
*/
public function assertAnonymousUser() {
    // Verify the user is logged out.
    if ($this->loggedIn()) {
      $this->logout();
    }
}

</code></pre>

<aside class="notes" data-markdown>
RawDrupalContext is what you extend when a context file is generated directly for you, and the above directory is where it's found.  You'll find another class in this directory, DrupalContext.  Where RawDrupalContext defines some shared functionality, this class is responsible for actually defining many of the steps you see when you execute behat with the 'di' flag.
If we look inside DrupalContext, we see that it has the following step definition (among many others).  The step leverages its parent class method to enact a a logout during the current scenario.</aside>


###Pain point #1.1
- In Behat 3, step defining classes are non-inheritable.
- In Behat 3, context classes cannot communicate with each other directly.
<footer>
Further (optional) reading: [SubContexts](https://behat-drupal-extension.readthedocs.io/en/3.1/subcontexts.html)
</footer>

<aside class="notes" data-markdown>
Some things you need to know at this point: In Behat 3, whenever a context class defines a step, it immediately becomes non-inheritable.  Defining a step simply consists of adding one of those @Given step tags within the docblock of one of the class' methods. The minute you try this, you'll get errors from php complaining that a function is already defined.  This occurs because Behat internals add any step definition tests to the environment, and then tries to add them again with any class that extends the step-defining one.  Step-defining context classes are therefore de-facto final.  Even as much as one step in the class is enough to make it so.

Additionally, context classes have no default means to communicate with each other.  Contexts have been expected to work in isolation up to this point. This is a problem. If I want my custom context to be able to perform some task that another context already implements, I have to recreate it in my own context.  This isn't great. I need to have a way to invoke the context defined by the other function as part of my context operation.

In the past, you would just extend DrupalContext as a Subcontext.  Unfortunately, subcontexts, which were part of Behat 2 and which this extension was built around, have been removed.  Now everything is a Context, which usually means step-defining and non-inheritable.

The author of DrupalExtension purports to get around this communication issue by continuing to support Subcontexts within his codebase.  This might be enough, were it not for pain point #2
</aside>

---------------------------------------------
##Pain point #2
###Lack of shared state

- Side note: state isn't always cleaned up well - DON'T run this on a production site.

<aside class="notes" data-markdown>
The second pain point is lack of shared state between running contexts.  Why is this a problem?  As it currently stands, when a class that extends RawDrupalExtension does something like create a user, it stores reference to that user internally on the context instance, so that it can be removed from the database at the completion of a scenario.  The implementation, however, was done in a Behat 2 world with subcontexts in mind.  Under the new paradigm, if a context creates something for testing purposes, other contexts have no idea that node exists - it's in the database, but they have no means of referencing it.  Similarly, if another context logs in a user, you can't determine anything about that logged in user from your custom context.

One other thing I'll mention while here - state isn't always cleaned up well.  In particular, if a scenario crashes with a fatal PHP exception, or if you create something as a side effect of leveraging the mink api to interact with browser fields, the context classes won't know about it, and it'll remain in the database after the scenario ends.  Usually, this hasn't presented a problem, other than database cruft, but I would never want to run tests directly on a production site as a result of this.
</aside>

---------------------------------------------
#Our custom work at Eelzee

- [https://github.com/aronbeal/drupalextension](https://github.com/aronbeal/drupalextension)

<aside class="notes" data-markdown>
This segues into our custom work at Eelzee.  We've been trying to solve these issues at Eelzee.  On this page, you can see the url of a fork of the DrupalExtension we've been working from.  We very much hope to get the changes we're making folded back into the original project at some point, but it's still early days.  The maintainer is very busy and often hard to reach, but he's been enthusiastic about the direction of our work.

Here's some of the 'eureka' moments we've had so far, the lessons of which will hopefully be helpful to you even if you end up working just with the original.
</aside>


##Optimization 1 
###Storing context references during runtime

- Capture reference to external context in the `@BeforeScenario` hook
- store internally in a custom class (NOT a step-defining class)
- Invoke directly from custom code

<footer data-trim>

<p>Further reading:

<ul><li>[How to deal with "Step is already defined"](https://www.drupal.org/node/2685951#comment-10961257)</li>
<li>[Add a cookbook about accessing contexts from each other](https://github.com/Behat/docs/pull/65)</li></ul>
</p>
</footer>

<aside class="notes" data-markdown>
The first optimization we undertook was storing references to other contexts.  This is possible by using the @BuildScenario hook in the context bootstrap process to capture the other contexts as they are being set up prior to the scenario, and stores internal copies of them.  It then can use those copies to ask questions of the other contexts in order to get work done.  While you can invoke another step directly in the feature file, what we're talking about here is gaining the ability to invoke another context's method (or series of methods) directly.
</aside>


<pre><code class="php" data-trim>
  /**
   * @BeforeScenario
   *
   */
  public function beforeScenario(\Behat\Behat\Hook\Scope\BeforeScenarioScope $scope) {

	  $environment = $scope->getEnvironment();
	  $settings    = $environment->getSuite()->getSettings();
	  foreach ($settings['contexts'] as $context_name) {
	    $context = $environment->getContext($context_name);
	    self::$contexts->add($context, array('key' => $context_name));
	  }
  }
	</code></pre>
<pre><code class="php" data-trim>
  /**
   * Convenience method.  Invokes a method on another context object.
   */
  public function callContext($context_name, $method) {
    $other_context = self::$contexts->get($context_name);
    $args = array_slice(func_get_args(), 2);
    return call_user_func_array(array($other_context, $method), $args);
  }
	</code></pre>

<aside class="notes" data-markdown>
Here is a couple code snippets showing this in action.  In the interests of clarity, I've trimmed these functions almost down to the bare bones.

The first uses the @BeforeScenario hook, one of many hooks into the testing process, to capture references to the other contexts during scenario spool-up.  The second retrieves the stored context, and invokes one of its functions on this context's behalf.

DrupalExtension still does not do this out of the box.  You'll need some sort of context capture like this if you want this level of reuse.
</aside>


##Optimization 2: Refactoring of means of internal storage to allow shared state

<aside class="notes" data-markdown>
The second optimization we performed deals with shared state.  We wanted to be able to have ANY context class running during a scenario be able to know what other nodes had been created by any other context class.  In order to do this, we rewrote the internals of the RawDrupalContext class to be a more advanced static version - more advanced, because we wanted more flexibility with retrieving nodes we had created later during the scenario, and static, so that the containing data structures would no longer live on the instances of the contexts, with no knowledge of each other.  Our revamping has all contexts adding their creations to a common pool.
	</aside>


##Optimization 3: Divestment of functionality from step definitions
<figure class="col polaroid">
	<img class="rot180" src="img/therapy_ball.png" alt="Martin Fowler's testing pyramid."/>
	<figcaption>moving the functionality away from the steps</figcaption>
</figure>

<aside class="notes" data-markdown>
The third optimization came upon the heels of a realization.  You cannot extend step-defining context classes - if you try, you'll get a 'function is already defined' error, as Behat tries to bring the same method call in twice - once for the parent, and once for the inheriting child.

We therefore began removing all significant functionality from the steps themselves, and moving it into the parent classes.  Unlike step-defining context classes, these parent classes could be extended, and moving the functionality into the parent would maximize the possibility for function inheritance and reuse.  I liken it to the therapy ball shown above - the functionality lives centrally, with the steps themselves being reduced to mere nubs on the surface.

In practice, this was a little bit overkill - it turned out that only the functionality that deals with stored state needs to live in non-step defining classes.  Still, this approach has proven very effective so far - we've taken this to the point where any context class that doesn't define steps, we declare as abstract, so we know immediately there won't be any steps in it, and any step defining class we declare as final, so as to take the inherit restrictions in the structure and make them formal.
</aside>

##Optimization 4: "Polling" functions

<pre><code class="php" data-trim>
public function poll($method_name, $wait)
{
    // Everything after the second argument is arguments for the method.
    $args = array_slice(func_get_args(), 2);

    for ($i = 0; $i < intval($wait); $i++) {
      try {
        // TODO: note - relies on called method throwing exception in the case
        // of a problem.  This is generally true with assertion functions,
        // but I may need to revisit this for the general case.

        if (!call_user_func_array(array($this, $method_name), $args)) {
          return false;
        }
        return true;
      } catch (\Exception $e) {
        if (($i + 1) === intval($wait)) {
          throw new \Exception(sprintf("%s::%s: %s (after %s seconds)", get_class($this), __FUNCTION__, $e->getMessage(), $wait));
        }
      }
      // One second between test iterations.
      sleep(1);
    }
}
</code></pre>

<aside class="notes" data-markdown>
The next optimization I'm going to mention tonight had to do with where context files and feature files live.  Adding features and context files requires changing of the Behat yaml configuration file every time it happens.  We wanted that to happen automatically - we wanted feature and context auto-discovery.  We expect our feature and context files to move in rough synchronization with the code they are intended to test, and we wanted a separation between the custom work we did that could apply to any Drupal site, and the work that would apply to a particular site.

I ended up building a small cli tool using nodejs that would accomplish this for us.  It's not yet ready for general release, unfortunately - it hasn't been tested on anything but my local development platform so far, and I'm currently focusing my efforts on expanding the Behat Extension Driver code, so it may be awhile before it does.  Nevertheless, I mention it here, because I believe it to be an important realization that will benefit others who are considering how to structure their projects.
</aside>

##Optimization 5: Let custom steps and features live with the site repo

<aside class="notes" data-markdown>
The next optimization I'm going to mention tonight had to do with where context files and feature files live.  Adding features and context files requires changing of the Behat yaml configuration file every time it happens.  We wanted that to happen automatically - we wanted feature and context auto-discovery.  We expect our feature and context files to move in rough synchronization with the code they are intended to test, and we wanted a separation between the custom work we did that could apply to any Drupal site, and the work that would apply to a particular site.

I ended up building a small cli tool using nodejs that would accomplish this for us.  It's not yet ready for general release, unfortunately - it hasn't been tested on anything but my local development platform so far, and I'm currently focusing my efforts on expanding the Behat Extension Driver code, so it may be awhile before it does.  Nevertheless, I mention it here, because I believe it to be an important realization that will benefit others who are considering how to structure their projects.
</aside>

---------------------------------------------

#Questions?

- [Original](https://github.com/jhedstrom/drupalextension)
- [Fork](https://github.com/aronbeal/drupalextension)
<aside class="notes" data-markdown>
That's about all I had.  Like I said, this whole project is a work in progress, and I'm sorry that I can't tell you the enhancements we have are polished and mature, or even that they'll be merged back into the original once they become so.  Nevertheless, my forked variant is freely available for you to download and try out if you like, and it's very usable even without the cli tooling I mentioned earlier.  I welcome thoughts and comments on the approach I've taken, and I hope it is useful to folks, whether it's merged back or not.

Thanks!  [Questions]
</aside>