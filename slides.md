# Eelzee case study: D7 Testing with Behat

Functional testing in Drupal 7 using Behat 3, Mink, Selenium, DrupalExtension, and some other homegrown tools

<aside class="notes">
This presentation is on using Behat 3 as the basis for testing a Drupal 7 installation, using Mink, Selenium, the Behat Drupal Extension, and some other home grown tools to get the testing done. This presentation does not cover installation of said tools, but I've included links at the end of the presentation to resources for that - please let me know afterwords if you need help and I'll assist if I can. This presentation will give a rough overview of the listed tools and the roles they serve, and then delve immediately into the customizations and lessons we at Eelzee found most useful for our particular setup. Not everything in this presentation is going to be generally useful to folks, I'm hoping a discussion of the journey might help those of you who are considering the same.
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

<aside class="notes">
The tools named on the prior page might be foreign to many here, but SimpleTest shouldn't be.  Before we start talking about what those listed tools are and how they'll help us, i should mention why we leaned away from SimpleTest.
On this slide I've got a thread title and a quote in this page, with sources footnoted below.  The first is the meta thread titled "Replace the testing framework with PHPUnit and possibly rewrite Simpletest", whose url is in the footnotes below.  The long term suggestions for this thread include using PHPUnit for unit tests, and Mink+Guzzle+Goutte for web/integration tests, and possibly Behat for  BDD tests.  This thread is the primary rationale for our choice, although it was backed up by other sources like the quote that follows it.  We adopted these tools primarily so that we can write testing that is more or less forward compatible with the future plans of Drupal.
	We've been focusing entirely on the functional side of things, so I'll only be addressing that in the talk today.  I'm a long way from being able to give a talk on PHPUnit with any feeling like I'm passing on a best practice, so I'll defer that talk for others.</aside>

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

<aside class="notes">
Behat is a Behavior Driven Development, or "BDD" framework, a type of functional or acceptance testing.  It provides a means to translate english descriptions of desirable behavior into testable php code.  This is primarily distinguished from Unit testing - where Unit testing tests only an isolated bit of functionality, such as a function or method,  functional testing tests a slice of functionality for a site, and Acceptance testing takes it further, to ensure the software properly satisfies the business requirements it was built for.  Behat most accurately falls in this last camp, because of how the tests are written
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

<aside class="notes">
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

Speaker Notes: What are Mink and Selenium?   Mink is an open source browser controller/emulator for web applications.  The distinction being drawn here is between browser emulators, which request pages and process them in code (but don't do javascript or ajax), and browser controllers, which interact with an actual browser.  Mink is meant to unify the api differences between emulators and controllers, as both will likely eventually be needed in testing.

Selenium is a browser controller leveraged by mink to do the actual driving of the browser during testing that requires javascript or ajax.  Selenium has drivers for all the major browsers, and many third party drivers as well.

---------------------------------------------
##What is DrupalExtension?

- Provides Step definitions common to Drupal testing scenarios
- Execution through:
	- Blackbox: no privileged access
	- Drush
	- DrupalApi

<footer>
Further reading:
- [DrupalExtension docs](http://behat-drupal-extension.readthedocs.org/en/3.1/drupalapi.html)
</footer>

<aside class="notes">
The last piece is the Behat Drupal Extension.  It's an integration layer between Behat and Drupal.  It provides step definitions for common scenarios specific to Drupal sites.  Common actions like logging in, creating and deleting users, nodes, taxonomy terms, and so forth

The extension provides testing access via the blackbox, Drush, or Drupal drivers.  Here's a rough summation of the differences:
The blackbox driver skips the service layer entirely, and assumes no privileged access to the site. You can run the tests on a local or remote server, but all the actions must take place by using the site's user interface.
The Drupal API Driver is the fastest and the most powerful of the three drivers. Its biggest limitation is that the tests must run on the same server as the Drupal site.  This was not a limitation for us, so that's mostly what I'll be focusing on here.
The Drush API driver serves as a partial service layer implementation, but one that can't use javascript. The main advantage of the Drush driver is that it can work when your tests run on a different server than the site being tested.

You can start functional testing by using the steps this module defines without worrying about writing your own custom step definitions.  It's worthwhile trying to accomplish your testing simply using what is on offer there, but we've found that it's best to start writing your own definitions almost immediately, as it gives you far greater control as to how your tests are written.  Before that, though, let's look an example Behat test:
</aside>

---------------------------------------------
#Behat Feature Examples


##Feature Examples

- Stored in .feature files
- Default english in implementation

<aside class="notes">
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

<aside class="notes">Here is a very basic example of what a feature file might look like.  Some areas of interest:
The feature section at the top here is a general descriptor of the test.  While the words in the description have no programmatic significance, it's going to be the best place to store what both you and your client expect out of that particular test.

The word 'api' with the at symbol in front of it is a tag, which can serve as run-time signifiers to Behat, or as categorization for the test writer.  For example, @javascript indicates to Behat to leverage Selenium to control an actual browser for the test, whereas @api alone would be a signal to use the Drupal or Drush API driver to control a virtual browser with greater speed but less functionality.  There can be additional tags in there, including ones you can add yourself, like '@current' or '@broken', which you can use to select a particular test or set of tests within your suite.

The Scenario declaration is a more granular subsection of the Feature above it - a single test within the feature.  Programmatic state primarily exists here, within scenarios, and is cleaned up between them. Users and nodes you create, whom you're logged in as, that sort of thing.

	Below the scenario description is a series of steps, which begin with highlighted words like "Given", "When", and so forth.  These are the "steps" of the test, and are mapped to function invocations, defined by Behat, Mink, the Behat Drupal Extension, or by you yourself.</aside>


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

<aside class="notes">
Here is another feature example.  This demonstrates the use of Behat Scenario outlines, which is one of the ways Behat allows efficient testing.  Most of this is the same as before, with the exception of the "Scenario Outline" keywords, which trigger Behat to look for the table "Examples".   I've got three columns here, with the first row being a header row corresponding to the variable names used in the scenario outline text.

Behat considers each row in this table to be a separate scenario.  This means you cannot create a structure in one row, and expect it to be there in the next.  If you need this functionality, Behat supports something called "Backgrounds" in features, for taking actions that live across all scenarios in a feature.  I've linked to that, and the Scenario Outline docs in the footer.

There's more examples of feature construction in the documentation, but for now, I'd like to move on now to which ones we have available out of the box, and how these are defined.
</aside>

---------------------------------------------
#Installation and Configuration


##Demo

<iframe width="560" height="315" data-src="https://www.youtube.com/embed/X4_ctRB3ipk?rel=0" frameborder="0" allowfullscreen></iframe>

<footer>
- [DrupalExtension installation docs](https://behat-drupal-extension.readthedocs.io/en/3.1/localinstall.html)
</footer>

<aside class="notes">
	The first step of the install process is relatively straightforward.  You create a folder, install composer (either locally to the folder, or in the case of this video, globally), add a composer.json file that requires drupalextension, and run composer install.  This video is sped up, as this process took about 5 minutes to complete.

	While the video is running, however, I will take a moment to note a discrepany I've run into in the docs.  The composer configuration they list in the install didn't end up creating a bin/behat binary, so coming up, I'll be referencing the behat binary in vendor/bin/behat, which is where it's always reliably been available to me.  For shorthand, I might also just use 'behat', assuming a propery configured PATH in your local shell instance.</aside>


##Configuration file

Add behat.yml file:

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

Run `vendor/bin/behat --init`

<footer>
- [DrupalExtension installation docs](https://behat-drupal-extension.readthedocs.io/en/3.1/localinstall.html)
</footer>

<aside class="notes">
After the install completes, they ask you to add a behat.yml file with the contents shown here, and reconfigure the base_url to point to your local drupal installation.  I've slightly modified what they provide in the documentation, by adding the yellow higlighted lines for 'api_driver', which with the setting shown here, will invoke the Drupal API driver for scenarios with the @api tag.  I've also added the 'drupal' and 'drupal_root' settings pointing to the filesystem path of the instance I'm going to test.

Once that is done, just run the above 'behat init' command from the folder where this is all installed to, and it'll generate a FeatureContext file for you, under the features/bootstrap subfolder.  We'll talk about that context file in detail in a bit, but since we're talking about it, I'll just mention that without the yellow section, you'll probably end up extending the wrong base class.

Let's look at what's become available to us at this point without doing any further configuration.
</aside>

---------------------------------------------
##Determining available steps

- `vendor/bin/behat -di`

<pre><code class="bash">
Arons-MacBook-Air:my_behat_test aronbeal$ ./vendor/bin/behat -di
default | Given I am an anonymous user
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertAnonymousUser()`

default | Given I am not logged in
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertAnonymousUser()`

default | Given I am logged in as a user with the :role role(s)
        | Creates and authenticates a user with the given role(s).
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertAuthenticatedByRole()`

default | Given I am logged in as a/an :role
        | Creates and authenticates a user with the given role(s).
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertAuthenticatedByRole()`

default | Given I am logged in as a user with the :role role(s) and I have the following fields:
        | Creates and authenticates a user with the given role(s) and given fields.
        | | field_user_name     | John  |
        | | field_user_surname  | Smith |
        | | ...                 | ...   |
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertAuthenticatedByRoleWithGivenFields()`

default | Given I am logged in as :name
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertLoggedInByName()`

default | Given I am logged in as a user with the :permissions permission(s)
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertLoggedInWithPermissions()`

default | Then I should see (the text ):text in the ":rowText" row
        | Find text in a table row containing given text.
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertTextInTableRow()`

default | Given I click :link in the :rowText row
        | Attempts to find a link in a table row containing giving text. This is for
        | administrative pages such as the administer content types screen found at
        | `admin/structure/types`.
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertClickInTableRow()`

default | Then I (should )see the :link in the :rowText row
        | Attempts to find a link in a table row containing giving text. This is for
        | administrative pages such as the administer content types screen found at
        | `admin/structure/types`.
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertClickInTableRow()`

default | Given the cache has been cleared
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertCacheClear()`

default | Given I run cron
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertCron()`

default | Given I am viewing a/an :type (content )with the title :title
        | Creates content of the given type.
        | at `Drupal\DrupalExtension\Context\DrupalContext::createNode()`

default | Given a/an :type (content )with the title :title
        | Creates content of the given type.
        | at `Drupal\DrupalExtension\Context\DrupalContext::createNode()`

default | Given I am viewing my :type (content )with the title :title
        | Creates content authored by the current user.
        | at `Drupal\DrupalExtension\Context\DrupalContext::createMyNode()`

default | Given :type content:
        | Creates content of a given type provided in the form:
        | | title    | author     | status | created           |
        | | My title | Joe Editor | 1      | 2014-10-17 8:00am |
        | | ...      | ...        | ...    | ...               |
        | at `Drupal\DrupalExtension\Context\DrupalContext::createNodes()`

default | Given I am viewing a/an :type( content):
        | Creates content of the given type, provided in the form:
        | | title     | My node        |
        | | Field One | My field value |
        | | author    | Joe Editor     |
        | | status    | 1              |
        | | ...       | ...            |
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertViewingNode()`

default | Then I should be able to edit a/an :type( content)
        | Asserts that a given content type is editable.
        | at `Drupal\DrupalExtension\Context\DrupalContext::assertEditNodeOfType()`

default | Given I am viewing a/an :vocabulary term with the name :name
        | Creates a term on an existing vocabulary.
        | at `Drupal\DrupalExtension\Context\DrupalContext::createTerm()`

default | Given a/an :vocabulary term with the name :name
        | Creates a term on an existing vocabulary.
        | at `Drupal\DrupalExtension\Context\DrupalContext::createTerm()`

default | Given users:
        | Creates multiple users.
        | Provide user data in the following format:
        | | name     | mail         | roles        |
        | | user foo | foo@bar.com  | role1, role2 |
        | at `Drupal\DrupalExtension\Context\DrupalContext::createUsers()`

default | Given :vocabulary terms:
        | Creates one or more terms on an existing vocabulary.
        | at `Drupal\DrupalExtension\Context\DrupalContext::createTerms()`

default | Given the/these (following )languages are available:
        | Creates one or more languages.
        |
        | Provide language data in the following format:
        | | langcode |
        | | en       |
        | | fr       |
        |
        |   The table listing languages by their ISO code.
        | at `Drupal\DrupalExtension\Context\DrupalContext::createLanguages()`

default | Then (I )break
        | Pauses the scenario until the user presses a key. Useful when debugging a scenario.
        | at `Drupal\DrupalExtension\Context\DrupalContext::iPutABreakpoint()`

default | Given I am at :path
        | Visit a given path, and additionally check for HTTP response code 200.
        | at `Drupal\DrupalExtension\Context\MinkContext::assertAtPath()`

default | When I visit :path
        | Visit a given path, and additionally check for HTTP response code 200.
        | at `Drupal\DrupalExtension\Context\MinkContext::assertAtPath()`

default | When I click :link
        | at `Drupal\DrupalExtension\Context\MinkContext::assertClick()`

default | Given for :field I enter :value
        | at `Drupal\DrupalExtension\Context\MinkContext::assertEnterField()`

default | Given I enter :value for :field
        | at `Drupal\DrupalExtension\Context\MinkContext::assertEnterField()`

default | Given I wait for AJAX to finish
        | Wait for AJAX to finish.
        | at `Drupal\DrupalExtension\Context\MinkContext::iWaitForAjaxToFinish()`

default | When /^(?:|I )press "(?P&lt;button&gt;(?:[^"]|\\")*)"$/
        | Presses button with specified id|name|title|alt|value
        | Example: When I press "Log In"
        | Example: And I press "Log In"
        | at `Drupal\DrupalExtension\Context\MinkContext::pressButton()`

default | When I press the :button button
        | Presses button with specified id|name|title|alt|value.
        | at `Drupal\DrupalExtension\Context\MinkContext::pressButton()`

default | Given I press the :char key in the :field field
        | at `Drupal\DrupalExtension\Context\MinkContext::pressKey()`

default | Then I should see the link :link
        | at `Drupal\DrupalExtension\Context\MinkContext::assertLinkVisible()`

default | Then I should not see the link :link
        | Links are not loaded on the page.
        | at `Drupal\DrupalExtension\Context\MinkContext::assertNotLinkVisible()`

default | Then I should not visibly see the link :link
        | Links are loaded but not visually visible (e.g they have display: hidden applied).
        | at `Drupal\DrupalExtension\Context\MinkContext::assertNotLinkVisuallyVisible()`

default | Then I (should )see the heading :heading
        | at `Drupal\DrupalExtension\Context\MinkContext::assertHeading()`

default | Then I (should )not see the heading :heading
        | at `Drupal\DrupalExtension\Context\MinkContext::assertNotHeading()`

default | Then I (should ) see the button :button
        | at `Drupal\DrupalExtension\Context\MinkContext::assertButton()`

default | Then I (should ) see the :button button
        | at `Drupal\DrupalExtension\Context\MinkContext::assertButton()`

default | When I follow/click :link in the :region( region)
        |   If region or link within it cannot be found.
        | at `Drupal\DrupalExtension\Context\MinkContext::assertRegionLinkFollow()`

default | Given I press :button in the :region( region)
        | Checks, if a button with id|name|title|alt|value exists or not and pressess the same
        |
        |
        |   string The id|name|title|alt|value of the button to be pressed
        |
        |   string The region in which the button should be pressed
        |
        |   If region or button within it cannot be found.
        | at `Drupal\DrupalExtension\Context\MinkContext::assertRegionPressButton()`

default | Given I fill in :value for :field in the :region( region)
        | Fills in a form field with id|name|title|alt|value in the specified region.
        |
        |
        |
        |   If region cannot be found.
        | at `Drupal\DrupalExtension\Context\MinkContext::regionFillField()`

default | Given I fill in :field with :value in the :region( region)
        | Fills in a form field with id|name|title|alt|value in the specified region.
        |
        |
        |
        |   If region cannot be found.
        | at `Drupal\DrupalExtension\Context\MinkContext::regionFillField()`

default | Then I should see the heading :heading in the :region( region)
        | Find a heading in a specific region.
        |
        |
        |
        |   If region or header within it cannot be found.
        | at `Drupal\DrupalExtension\Context\MinkContext::assertRegionHeading()`

default | Then I should see the :heading heading in the :region( region)
        | Find a heading in a specific region.
        |
        |
        |
        |   If region or header within it cannot be found.
        | at `Drupal\DrupalExtension\Context\MinkContext::assertRegionHeading()`

default | Then I should see the link :link in the :region( region)
        |   If region or link within it cannot be found.
        | at `Drupal\DrupalExtension\Context\MinkContext::assertLinkRegion()`

default | Then I should not see the link :link in the :region( region)
        |   If region or link within it cannot be found.
        | at `Drupal\DrupalExtension\Context\MinkContext::assertNotLinkRegion()`

default | Then I should see( the text) :text in the :region( region)
        |   If region or text within it cannot be found.
        | at `Drupal\DrupalExtension\Context\MinkContext::assertRegionText()`

default | Then I should not see( the text) :text in the :region( region)
        |   If region or text within it cannot be found.
        | at `Drupal\DrupalExtension\Context\MinkContext::assertNotRegionText()`

default | Then I (should )see the text :text
        | at `Drupal\DrupalExtension\Context\MinkContext::assertTextVisible()`

default | Then I should not see the text :text
        | at `Drupal\DrupalExtension\Context\MinkContext::assertNotTextVisible()`

default | Then I should get a :code HTTP response
        | at `Drupal\DrupalExtension\Context\MinkContext::assertHttpResponse()`

default | Then I should not get a :code HTTP response
        | at `Drupal\DrupalExtension\Context\MinkContext::assertNotHttpResponse()`

default | Given I check the box :checkbox
        | at `Drupal\DrupalExtension\Context\MinkContext::assertCheckBox()`

default | Given I uncheck the box :checkbox
        | at `Drupal\DrupalExtension\Context\MinkContext::assertUncheckBox()`

default | When I select the radio button :label with the id :id
        | at `Drupal\DrupalExtension\Context\MinkContext::assertSelectRadioById()`

default | When I select the radio button :label
        | at `Drupal\DrupalExtension\Context\MinkContext::assertSelectRadioById()`

default | Given /^(?:|I )am on (?:|the )homepage$/
        | Opens homepage
        | Example: Given I am on "/"
        | Example: When I go to "/"
        | Example: And I go to "/"
        | at `Drupal\DrupalExtension\Context\MinkContext::iAmOnHomepage()`

default | When /^(?:|I )go to (?:|the )homepage$/
        | Opens homepage
        | Example: Given I am on "/"
        | Example: When I go to "/"
        | Example: And I go to "/"
        | at `Drupal\DrupalExtension\Context\MinkContext::iAmOnHomepage()`

default | Given /^(?:|I )am on "(?P&lt;page&gt;[^"]+)"$/
        | Opens specified page
        | Example: Given I am on "http://batman.com"
        | Example: And I am on "/articles/isBatmanBruceWayne"
        | Example: When I go to "/articles/isBatmanBruceWayne"
        | at `Drupal\DrupalExtension\Context\MinkContext::visit()`

default | When /^(?:|I )go to "(?P&lt;page&gt;[^"]+)"$/
        | Opens specified page
        | Example: Given I am on "http://batman.com"
        | Example: And I am on "/articles/isBatmanBruceWayne"
        | Example: When I go to "/articles/isBatmanBruceWayne"
        | at `Drupal\DrupalExtension\Context\MinkContext::visit()`

default | When /^(?:|I )reload the page$/
        | Reloads current page
        | Example: When I reload the page
        | Example: And I reload the page
        | at `Drupal\DrupalExtension\Context\MinkContext::reload()`

default | When /^(?:|I )move backward one page$/
        | Moves backward one page in history
        | Example: When I move backward one page
        | at `Drupal\DrupalExtension\Context\MinkContext::back()`

default | When /^(?:|I )move forward one page$/
        | Moves forward one page in history
        | Example: And I move forward one page
        | at `Drupal\DrupalExtension\Context\MinkContext::forward()`

default | When /^(?:|I )follow "(?P&lt;link&gt;(?:[^"]|\\")*)"$/
        | Clicks link with specified id|title|alt|text
        | Example: When I follow "Log In"
        | Example: And I follow "Log In"
        | at `Drupal\DrupalExtension\Context\MinkContext::clickLink()`

default | When /^(?:|I )fill in "(?P&lt;field&gt;(?:[^"]|\\")*)" with "(?P&lt;value&gt;(?:[^"]|\\")*)"$/
        | Fills in form field with specified id|name|label|value
        | Example: When I fill in "username" with: "bwayne"
        | Example: And I fill in "bwayne" for "username"
        | at `Drupal\DrupalExtension\Context\MinkContext::fillField()`

default | When /^(?:|I )fill in "(?P&lt;field&gt;(?:[^"]|\\")*)" with:$/
        | Fills in form field with specified id|name|label|value
        | Example: When I fill in "username" with: "bwayne"
        | Example: And I fill in "bwayne" for "username"
        | at `Drupal\DrupalExtension\Context\MinkContext::fillField()`

default | When /^(?:|I )fill in "(?P&lt;value&gt;(?:[^"]|\\")*)" for "(?P&lt;field&gt;(?:[^"]|\\")*)"$/
        | Fills in form field with specified id|name|label|value
        | Example: When I fill in "username" with: "bwayne"
        | Example: And I fill in "bwayne" for "username"
        | at `Drupal\DrupalExtension\Context\MinkContext::fillField()`

default | When /^(?:|I )fill in the following:$/
        | Fills in form fields with provided table
        | Example: When I fill in the following"
        |              | username | bruceWayne |
        |              | password | iLoveBats123 |
        | Example: And I fill in the following"
        |              | username | bruceWayne |
        |              | password | iLoveBats123 |
        | at `Drupal\DrupalExtension\Context\MinkContext::fillFields()`

default | When /^(?:|I )select "(?P&lt;option&gt;(?:[^"]|\\")*)" from "(?P&lt;select&gt;(?:[^"]|\\")*)"$/
        | Selects option in select field with specified id|name|label|value
        | Example: When I select "Bats" from "user_fears"
        | Example: And I select "Bats" from "user_fears"
        | at `Drupal\DrupalExtension\Context\MinkContext::selectOption()`

default | When /^(?:|I )additionally select "(?P&lt;option&gt;(?:[^"]|\\")*)" from "(?P&lt;select&gt;(?:[^"]|\\")*)"$/
        | Selects additional option in select field with specified id|name|label|value
        | Example: When I additionally select "Deceased" from "parents_alive_status"
        | Example: And I additionally select "Deceased" from "parents_alive_status"
        | at `Drupal\DrupalExtension\Context\MinkContext::additionallySelectOption()`

default | When /^(?:|I )check "(?P&lt;option&gt;(?:[^"]|\\")*)"$/
        | Checks checkbox with specified id|name|label|value
        | Example: When I check "Pearl Necklace" from "itemsClaimed"
        | Example: And I check "Pearl Necklace" from "itemsClaimed"
        | at `Drupal\DrupalExtension\Context\MinkContext::checkOption()`

default | When /^(?:|I )uncheck "(?P&lt;option&gt;(?:[^"]|\\")*)"$/
        | Unchecks checkbox with specified id|name|label|value
        | Example: When I uncheck "Broadway Plays" from "hobbies"
        | Example: And I uncheck "Broadway Plays" from "hobbies"
        | at `Drupal\DrupalExtension\Context\MinkContext::uncheckOption()`

default | When /^(?:|I )attach the file "(?P&lt;path&gt;[^"]*)" to "(?P&lt;field&gt;(?:[^"]|\\")*)"$/
        | Attaches file to field with specified id|name|label|value
        | Example: When I attach "bwayne_profile.png" to "profileImageUpload"
        | Example: And I attach "bwayne_profile.png" to "profileImageUpload"
        | at `Drupal\DrupalExtension\Context\MinkContext::attachFileToField()`

default | Then /^(?:|I )should be on "(?P&lt;page&gt;[^"]+)"$/
        | Checks, that current page PATH is equal to specified
        | Example: Then I should be on "/"
        | Example: And I should be on "/bats"
        | Example: And I should be on "http://google.com"
        | at `Drupal\DrupalExtension\Context\MinkContext::assertPageAddress()`

default | Then /^(?:|I )should be on (?:|the )homepage$/
        | Checks, that current page is the homepage
        | Example: Then I should be on the homepage
        | Example: And I should be on the homepage
        | at `Drupal\DrupalExtension\Context\MinkContext::assertHomepage()`

default | Then /^the (?i)url(?-i) should match (?P&lt;pattern&gt;"(?:[^"]|\\")*")$/
        | Checks, that current page PATH matches regular expression
        | Example: Then the url should match "superman is dead"
        | Example: Then the uri should match "log in"
        | Example: And the url should match "log in"
        | at `Drupal\DrupalExtension\Context\MinkContext::assertUrlRegExp()`

default | Then /^the response status code should be (?P&lt;code&gt;\d+)$/
        | Checks, that current page response status is equal to specified
        | Example: Then the response status code should be 200
        | Example: And the response status code should be 400
        | at `Drupal\DrupalExtension\Context\MinkContext::assertResponseStatus()`

default | Then /^the response status code should not be (?P&lt;code&gt;\d+)$/
        | Checks, that current page response status is not equal to specified
        | Example: Then the response status code should not be 501
        | Example: And the response status code should not be 404
        | at `Drupal\DrupalExtension\Context\MinkContext::assertResponseStatusIsNot()`

default | Then /^(?:|I )should see "(?P&lt;text&gt;(?:[^"]|\\")*)"$/
        | Checks, that page contains specified text
        | Example: Then I should see "Who is the Batman?"
        | Example: And I should see "Who is the Batman?"
        | at `Drupal\DrupalExtension\Context\MinkContext::assertPageContainsText()`

default | Then /^(?:|I )should not see "(?P&lt;text&gt;(?:[^"]|\\")*)"$/
        | Checks, that page doesn't contain specified text
        | Example: Then I should not see "Batman is Bruce Wayne"
        | Example: And I should not see "Batman is Bruce Wayne"
        | at `Drupal\DrupalExtension\Context\MinkContext::assertPageNotContainsText()`

default | Then /^(?:|I )should see text matching (?P&lt;pattern&gt;"(?:[^"]|\\")*")$/
        | Checks, that page contains text matching specified pattern
        | Example: Then I should see text matching "Batman, the vigilante"
        | Example: And I should not see "Batman, the vigilante"
        | at `Drupal\DrupalExtension\Context\MinkContext::assertPageMatchesText()`

default | Then /^(?:|I )should not see text matching (?P&lt;pattern&gt;"(?:[^"]|\\")*")$/
        | Checks, that page doesn't contain text matching specified pattern
        | Example: Then I should see text matching "Bruce Wayne, the vigilante"
        | Example: And I should not see "Bruce Wayne, the vigilante"
        | at `Drupal\DrupalExtension\Context\MinkContext::assertPageNotMatchesText()`

default | Then /^the response should contain "(?P&lt;text&gt;(?:[^"]|\\")*)"$/
        | Checks, that HTML response contains specified string
        | Example: Then the response should contain "Batman is the hero Gotham deserves."
        | Example: And the response should contain "Batman is the hero Gotham deserves."
        | at `Drupal\DrupalExtension\Context\MinkContext::assertResponseContains()`

default | Then /^the response should not contain "(?P&lt;text&gt;(?:[^"]|\\")*)"$/
        | Checks, that HTML response doesn't contain specified string
        | Example: Then the response should not contain "Bruce Wayne is a billionaire, play-boy, vigilante."
        | Example: And the response should not contain "Bruce Wayne is a billionaire, play-boy, vigilante."
        | at `Drupal\DrupalExtension\Context\MinkContext::assertResponseNotContains()`

default | Then /^(?:|I )should see "(?P&lt;text&gt;(?:[^"]|\\")*)" in the "(?P&lt;element&gt;[^"]*)" element$/
        | Checks, that element with specified CSS contains specified text
        | Example: Then I should see "Batman" in the "heroes_list" element
        | Example: And I should see "Batman" in the "heroes_list" element
        | at `Drupal\DrupalExtension\Context\MinkContext::assertElementContainsText()`

default | Then /^(?:|I )should not see "(?P&lt;text&gt;(?:[^"]|\\")*)" in the "(?P&lt;element&gt;[^"]*)" element$/
        | Checks, that element with specified CSS doesn't contain specified text
        | Example: Then I should not see "Bruce Wayne" in the "heroes_alter_egos" element
        | Example: And I should not see "Bruce Wayne" in the "heroes_alter_egos" element
        | at `Drupal\DrupalExtension\Context\MinkContext::assertElementNotContainsText()`

default | Then /^the "(?P&lt;element&gt;[^"]*)" element should contain "(?P&lt;value&gt;(?:[^"]|\\")*)"$/
        | Checks, that element with specified CSS contains specified HTML
        | Example: Then the "body" element should contain "style=\"color:black;\""
        | Example: And the "body" element should contain "style=\"color:black;\""
        | at `Drupal\DrupalExtension\Context\MinkContext::assertElementContains()`

default | Then /^the "(?P&lt;element&gt;[^"]*)" element should not contain "(?P&lt;value&gt;(?:[^"]|\\")*)"$/
        | Checks, that element with specified CSS doesn't contain specified HTML
        | Example: Then the "body" element should not contain "style=\"color:black;\""
        | Example: And the "body" element should not contain "style=\"color:black;\""
        | at `Drupal\DrupalExtension\Context\MinkContext::assertElementNotContains()`

default | Then /^(?:|I )should see an? "(?P&lt;element&gt;[^"]*)" element$/
        | Checks, that element with specified CSS exists on page
        | Example: Then I should see a "body" element
        | Example: And I should see a "body" element
        | at `Drupal\DrupalExtension\Context\MinkContext::assertElementOnPage()`

default | Then /^(?:|I )should not see an? "(?P&lt;element&gt;[^"]*)" element$/
        | Checks, that element with specified CSS doesn't exist on page
        | Example: Then I should not see a "canvas" element
        | Example: And I should not see a "canvas" element
        | at `Drupal\DrupalExtension\Context\MinkContext::assertElementNotOnPage()`

default | Then /^the "(?P&lt;field&gt;(?:[^"]|\\")*)" field should contain "(?P&lt;value&gt;(?:[^"]|\\")*)"$/
        | Checks, that form field with specified id|name|label|value has specified value
        | Example: Then the "username" field should contain "bwayne"
        | Example: And the "username" field should contain "bwayne"
        | at `Drupal\DrupalExtension\Context\MinkContext::assertFieldContains()`

default | Then /^the "(?P&lt;field&gt;(?:[^"]|\\")*)" field should not contain "(?P&lt;value&gt;(?:[^"]|\\")*)"$/
        | Checks, that form field with specified id|name|label|value doesn't have specified value
        | Example: Then the "username" field should not contain "batman"
        | Example: And the "username" field should not contain "batman"
        | at `Drupal\DrupalExtension\Context\MinkContext::assertFieldNotContains()`

default | Then /^(?:|I )should see (?P&lt;num&gt;\d+) "(?P&lt;element&gt;[^"]*)" elements?$/
        | Checks, that (?P&lt;num&gt;\d+) CSS elements exist on the page
        | Example: Then I should see 5 "div" elements
        | Example: And I should see 5 "div" elements
        | at `Drupal\DrupalExtension\Context\MinkContext::assertNumElements()`

default | Then /^the "(?P&lt;checkbox&gt;(?:[^"]|\\")*)" checkbox should be checked$/
        | Checks, that checkbox with specified in|name|label|value is checked
        | Example: Then the "remember_me" checkbox should be checked
        | Example: And the "remember_me" checkbox is checked
        | at `Drupal\DrupalExtension\Context\MinkContext::assertCheckboxChecked()`

default | Then /^the checkbox "(?P&lt;checkbox&gt;(?:[^"]|\\")*)" (?:is|should be) checked$/
        | Checks, that checkbox with specified in|name|label|value is checked
        | Example: Then the "remember_me" checkbox should be checked
        | Example: And the "remember_me" checkbox is checked
        | at `Drupal\DrupalExtension\Context\MinkContext::assertCheckboxChecked()`

default | Then /^the "(?P&lt;checkbox&gt;(?:[^"]|\\")*)" checkbox should not be checked$/
        | Checks, that checkbox with specified in|name|label|value is unchecked
        | Example: Then the "newsletter" checkbox should be unchecked
        | Example: Then the "newsletter" checkbox should not be checked
        | Example: And the "newsletter" checkbox is unchecked
        | at `Drupal\DrupalExtension\Context\MinkContext::assertCheckboxNotChecked()`

default | Then /^the checkbox "(?P&lt;checkbox&gt;(?:[^"]|\\")*)" should (?:be unchecked|not be checked)$/
        | Checks, that checkbox with specified in|name|label|value is unchecked
        | Example: Then the "newsletter" checkbox should be unchecked
        | Example: Then the "newsletter" checkbox should not be checked
        | Example: And the "newsletter" checkbox is unchecked
        | at `Drupal\DrupalExtension\Context\MinkContext::assertCheckboxNotChecked()`

default | Then /^the checkbox "(?P&lt;checkbox&gt;(?:[^"]|\\")*)" is (?:unchecked|not checked)$/
        | Checks, that checkbox with specified in|name|label|value is unchecked
        | Example: Then the "newsletter" checkbox should be unchecked
        | Example: Then the "newsletter" checkbox should not be checked
        | Example: And the "newsletter" checkbox is unchecked
        | at `Drupal\DrupalExtension\Context\MinkContext::assertCheckboxNotChecked()`

default | Then /^print current URL$/
        | Prints current URL to console.
        | Example: Then print current URL
        | Example: And print current URL
        | at `Drupal\DrupalExtension\Context\MinkContext::printCurrentUrl()`

default | Then /^print last response$/
        | Prints last response to console
        | Example: Then print current response
        | Example: And print current response
        | at `Drupal\DrupalExtension\Context\MinkContext::printLastResponse()`

default | Then /^show last response$/
        | Opens last response content in browser
        | Example: Then show last response
        | Example: And show last response
        | at `Drupal\DrupalExtension\Context\MinkContext::showLastResponse()`

default | Then I should see the error message( containing) :message
        | Checks if the current page contains the given error message
        |
        |   string The text to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertErrorVisible()`

default | Then I should see the following error message(s):
        | Checks if the current page contains the given set of error messages
        |
        |   array An array of texts to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertMultipleErrors()`

default | Given I should not see the error message( containing) :message
        | Checks if the current page does not contain the given error message
        |
        |   string The text to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertNotErrorVisible()`

default | Then I should not see the following error messages:
        | Checks if the current page does not contain the given set error messages
        |
        |   array An array of texts to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertNotMultipleErrors()`

default | Then I should see the success message( containing) :message
        | Checks if the current page contains the given success message
        |
        |   string The text to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertSuccessMessage()`

default | Then I should see the following success messages:
        | Checks if the current page contains the given set of success messages
        |
        |   array An array of texts to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertMultipleSuccessMessage()`

default | Given I should not see the success message( containing) :message
        | Checks if the current page does not contain the given set of success message
        |
        |   string The text to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertNotSuccessMessage()`

default | Then I should not see the following success messages:
        | Checks if the current page does not contain the given set of success messages
        |
        |   array An array of texts to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertNotMultipleSuccessMessage()`

default | Then I should see the warning message( containing) :message
        | Checks if the current page contains the given warning message
        |
        |   string The text to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertWarningMessage()`

default | Then I should see the following warning messages:
        | Checks if the current page contains the given set of warning messages
        |
        |   array An array of texts to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertMultipleWarningMessage()`

default | Given I should not see the warning message( containing) :message
        | Checks if the current page does not contain the given set of warning message
        |
        |   string The text to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertNotWarningMessage()`

default | Then I should not see the following warning messages:
        | Checks if the current page does not contain the given set of warning messages
        |
        |   array An array of texts to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertNotMultipleWarningMessage()`

default | Then I should see the message( containing) :message
        | Checks if the current page contain the given message
        |
        |   string The message to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertMessage()`

default | Then I should not see the message( containing) :message
        | Checks if the current page does not contain the given message
        |
        |   string The message to be checked
        | at `Drupal\DrupalExtension\Context\MessageContext::assertNotMessage()`

default | Given I run drush :command
        | at `Drupal\DrupalExtension\Context\DrushContext::assertDrushCommand()`

default | Given I run drush :command :arguments
        | at `Drupal\DrupalExtension\Context\DrushContext::assertDrushCommandWithArgument()`

default | Then drush output should contain :output
        | at `Drupal\DrupalExtension\Context\DrushContext::assertDrushOutput()`

default | Then drush output should not contain :output
        | at `Drupal\DrupalExtension\Context\DrushContext::drushOutputShouldNotContain()`

default | Then print last drush output
        | at `Drupal\DrupalExtension\Context\DrushContext::printLastDrushOutput()`
    </code></pre>

<aside class="notes">
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

<aside class="notes">
Let's get started and create a simple Behat feature.  Behat knows out of the box to look for this within the feature folder it created earlier, so for now, if you're repeating this later, you should add it there.  Doesn't matter what you call it, as long as it ends with dot feature.

We save that, and run it.
</aside>


##<span class="fail">Our first feature file failure</span>

<pre><code class="bash" data-trim>
Arons-MacBook-Air:my_behat_test aronbeal$ ./vendor/bin/behat
Feature: this test is to simply to get a single working feature.
  In order to create a basis to work from
  As a system developer
  I want to have a standard by which I can draw from for future reference.

  Scenario: Get one working test                    # features/my_first_feature.feature:6
    Given I am logged in as an "authenticated user" # Drupal\DrupalExtension\Context\DrupalContext::assertAuthenticatedByRole()
      No ability to generate random in Drupal\Driver\BlackboxDriver. Put `@api` into your feature and add an API driver (ex: `api_driver: drupal`) in behat.yml. (Drupal\Driver\Exception\UnsupportedDriverActionException)
    And I am on "/"                                 # Drupal\DrupalExtension\Context\MinkContext::visit()
    Then I should not see the text "Login Required" # Drupal\DrupalExtension\Context\MinkContext::assertNotTextVisible()

--- Failed scenarios:

    features/my_first_feature.feature:6
	</code></pre>

<aside class="notes">
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

<pre><code class="bash" data-trim>
Arons-MacBook-Air:my_behat_test aronbeal$ ./vendor/bin/behat
Feature: this test is to simply to get a single working feature.
  In order to create a basis to work from
  As a system developer
  I want to have a standard by which I can draw from for future reference.

  @api
  Scenario: Get one working test                    # features/my_first_feature.feature:7
    Given I am logged in as an "authenticated user" # Drupal\DrupalExtension\Context\DrupalContext::assertAuthenticatedByRole()
    And I am on "/"                                 # Drupal\DrupalExtension\Context\MinkContext::visit()
    Then I should not see the text "Login Required" # Drupal\DrupalExtension\Context\MinkContext::assertNotTextVisible()

1 scenario (1 passed)
3 steps (3 passed)
0m6.64s (27.94Mb)
	</code></pre>

<footer>
Further reading: [Our current behat.yml file](https://bitbucket.org/eelzeedev/testing/src/ab82ffce4105a71d5790927e09d247ce78888dc4/behat/default.behat.yml?at=master&fileviewer=file-view-default)
</footer>

<aside class="notes">
Running our feature now, after adding the api flag, and properly setting the drupal_root din the DrupalExtension section, and teh base_url in the MinkExtension section, we see that it has finally passed muster.

There are going to be multiple hiccups for you in the early stages here while you write tests.  The presentation will be linked to later - I'll leave you with the configuration file linked in the footer.  That file that serves as the basis of what we're currently using, which I hope will be useful as a reference of a known good value when you're monkeying with this stuff.  The section under Drupal\DrupalExtension will be the config settings specific to what we're working with here.  In particular, the api_driver line, the drupal: section, and the selectors section define the drupal api driver as our api driver of choice, and the selectors will be used later as a means of signifying to the subsystem the css wrapper class around the outside of our message elements, which it can use to see if certain messages will appear.
</aside>

---------------------------------------------
#Behat Context files

<aside class="notes">
	At this point, we've installed the drupal extension, pointed it at our drupal installation, and run a feature test based on the step definitions offered by the Behat Drupal Extension.  I'd like to move now into the classfiles that are responsible for performing the translation between the text you've seen and the executable PHP code.</aside>


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

<aside class="notes">
	First, we'll look at the Context file generated for us when we executed "behat init".  This file by itself doesn't actually have any step definitions yet. We'll add one now, but we're going to use Behat to help us with the function signature.</aside>


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

Speaker notes:
Here's our modified feature.  I've removed the login test for clarity, and added the line [click] "Given I have a lipsum article".  We're going to use this to create an article with preformatted content, presuming it's the sort of thing we need a lot in our testing.


Executing provides the method signature:

<pre><code class="bash" data-trim data-noescape>
Arons-MacBook-Air:my_behat_test aronbeal$ ./vendor/bin/behat features/my_second_feature.feature
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
<mark class="fragment highlight-green">
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

<aside class="notes">
	Running this new scenario, I get the output shown here.  We see that Behat has noticed we have an undefined step, and has [click] given us a basic method signature that throws a Behat-specific exception by default, indicating we've got unfinished business in the method body.</aside>


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
  <mark class="fragment">
  /**
   * @Given I have a lipsum article
   */
  public function iHaveALipsumArticle() {

    throw new PendingException();
  }</mark>
}
</code></pre>

<aside class="notes">
We copy that signature over to our FeatureContext class.  If we run the test again now, we'll see output indicating that the step was found, and is in a "TODO" status.
	</aside>


<pre><code class="bash full" data-trim data-noescape>
Feature: this test is to simply to get a single working feature.
  In order to create a basis to work from
  As a system developer
  I want to have a standard by which I can draw from for future reference.

  @api
  Scenario: Create a new feature  # features/my_second_feature.feature:7
    Given I have a lipsum article # FeatureContext::iHaveALipsumArticle()
    <mark class="fragment highlight-green">TODO: write pending definition</mark>

1 scenario (1 pending)
1 step (1 pending)
0m0.40s (26.40Mb)
	</code></pre>

---------------------------------------------
#What's next
##Shortcomings, customizations, and future directions


<aside class="notes">
	That's the bare bones of how all this works.  I'm going to need to step it up a notch in terms of complexity in order to talk about the next section.  It's unfortunately unavoidable - I need to talk about the file heirarchy, some optimizations we've done at our shop to make testing easier and more powerful, and some of the pain points with this tool, especially in terms of Behat 3.  Remember, I'll have this presentation online for revisiting later.</aside>

---------------------------------------------
#Pain point #1

- Lack of inter-context communication

<aside class="notes">
When you start digging into this, you're probably going to hit the same issue we did fairly quickly - that of Context files not being able to communicate with each other, and lack of shared state.

Let's look at an example of that first point.
	</aside>

<pre><code class="bash" data-trim>
Arons-MacBook-Air:my_behat_test aronbeal$ tree vendor/drupal/drupal-extension/src/Drupal/DrupalExtension/Context/
vendor/drupal/drupal-extension/src/Drupal/DrupalExtension/Context/
├── Annotation
│   └── Reader.php
├── BatchContext.php
├── ContextClass
│   └── ClassGenerator.php
├── DrupalAwareInterface.php
├── DrupalContext.php
├── DrupalSubContextBase.php
├── DrupalSubContextInterface.php
├── DrushContext.php
├── Environment
│   └── Reader
│       └── Reader.php
├── Initializer
│   └── DrupalAwareInitializer.php
├── MarkupContext.php
├── MessageContext.php
├── MinkContext.php
└── RawDrupalContext.php
</code></pre>
<aside class="notes">
The directory shown above is probably the first place you're going to end up when you start trying to create your own context files.  RawDrupalContext is what you extend when a context file is generated directly for you, and this is where it's found.

You'll find another class in this directory, DrupalContext.  It is responsible for defining many of the steps you see when you execute behat with the 'di' flag.
	</aside>


Found in DrupalContext:
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
<footer>
Further reading: [SubContexts](https://behat-drupal-extension.readthedocs.io/en/3.1/subcontexts.html)
	</footer>

<aside class="notes">
If we look inside DrupalContext, we see that it has the following step definition (among many others).  The step leverages its parent class method to enact a a logout during the current scenario.

The crux of the first problem is this: If I want my custom context to be able to log out as part of its operation, and I wish to call this function, I need to have a way to tell DrupalContext to invoke that function.  Out of the box, however, my context has no way to communicate with any other contexts during runtime.

In the past, you would just extend DrupalContext as a Subcontext.  Unfortunately, subcontexts, which were part of Behat 2 and which this extension was built around, have been removed.  Now everything is a Context, but inter-context communication is not (directly) supported.  You also cannot extend a step-defining context class - the classes with the phpdoc english snippets in them?  The minute you try to, you'll get errors from php complaining that a function is already defined.  This occurs because Behat internals add any step definition tests to the environment, and then tries to add them again with any class that extends the step-defining one.  Step-defining context classes are therefore de-facto final.  Even as much as one step in the class is enough to make it so.

The author of DrupalExtension purports to get around this issue by continuing to support Subcontexts within his codebase.  This might be enough, were it not for pain point #2
	</aside>

---------------------------------------------
#Pain point #2

- Lack of shared state

<aside class="notes">
The second pain point is lack of shared state between running contexts.  Why is this a problem?  As it currently stands, when a class that extends RawDrupalExtension does something like create a user, it stores reference to that user internally on the context instance, so that it can be removed from the database at the completion of a scenario.  The implementation, however, was done in a Behat 2 world with subcontexts in mind.  Under the new paradigm, if a context creates a node for testing purposes, other contexts have no idea that node exists - it's in the database, but they have no means of referencing it.  Similarly, if another context logs in a user, you can't determine anything about that logged in user from your custom context.
	</aside>

---------------------------------------------
#Our custom work at Eelzee

- [https://github.com/aronbeal/drupalextension](https://github.com/aronbeal/drupalextension)

<aside class="notes">
We've been doing a lot of work trying to solve these issues at Eelzee.  On this page, you can see the url of a fork of the DrupalExtension we've been working from.  We very much hope to get the changes we're making folded back into the original project at some point, but it's still early days, and the original author is very busy.  Here's some of the 'eureka' moments we've had so far, the lessons of which will hopefully be helpful to you even if you avoid trying our fork.
	</aside>


##Optimization 1: Storing context references during runtime

- Capture reference to external context in the `@BeforeScenario` hook
- store internally in a custom class (NOT a step-defining class)
- Invoke directly from custom code

<footer>
Further reading: 
- [How to deal with "Step is already defined"](https://www.drupal.org/node/2685951#comment-10961257)
- [Add a cookbook about accessing contexts from each other](https://github.com/Behat/docs/pull/65)
</footer>

<aside class="notes">
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

<aside class="notes">
Here is a couple code snippets showing this in action.  In the interests of clarity, I've trimmed these functions down to the bare bones.

The first uses the @BeforeScenario hook, one of many hooks into the testing process, to capture references to the other contexts during scenario spool-up.  The second retrieves the stored context, and invokes one of its functions on this context's behalf.

DrupalExtension still does not do this out of the box.  You'll need some sort of context capture like this if you want this level of reuse.
	</aside>


##Optimization 2: Refactoring of means of internal storage to allow shared state

<aside class="notes">
The second optimization we performed deals with shared state.  We wanted to be able to have ANY context class running during a scenario be able to know what other nodes had been created by any other context class.  In order to do this, we rewrote the internals of the RawDrupalContext class to be a more advanced static version - more advanced, because we wanted more flexibility with retrieving nodes we had created later during the scenario, and static, so that the containing data structures would no longer live on the instances of the contexts, with no knowledge of each other.  Our revamping has all contexts adding their creations to a common pool.
	</aside>


##Optimization 3: Divestment of functionality from step definitions
<figure class="col polaroid">
	<img class="rot180" src="img/therapy_ball.png" alt="Martin Fowler's testing pyramid."/>
	<figcaption>moving the functionality away from the steps</figcaption>
</figure>

<aside class="notes">
The third optimization came upon the heels of a realization.  You cannot extend step-defining context classes - if you try, you'll get a 'function is already defined' error, as Behat tries to bring the same method call in twice - once for the parent, and once for the inheriting child.

We therefore began removing all significant functionality from the steps themselves, and moving it into the parent classes.  Unlike step-defining context classes, these parent classes could be extended, doing so would maximize function inheritance and reuse.  I liken it to the therapy ball shown above - the functionality lives centrally, with the steps themselves being mere nubs on the surface.  

In practice, this was a little bit overkill - it turned out that only the functionality that deals with stored state needs to live in non-step defining classes.  Still, this approach has proven very effective so far - we've taken this to the point where any context class that doesn't define steps, we declare as abstract, so we know immediately there won't be any steps in it, and any step defining class we declare as final, so as to take the inherit restrictions in the structure and make them formal.
	</aside>


