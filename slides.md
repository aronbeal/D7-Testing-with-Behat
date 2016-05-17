## Eelzee case study: D7 Testing with Behat

Functional testing in Drupal 7 using Behat 3, Mink, Selenium, DrupalExtension, and some other homegrown tools


Speaker Notes:
This presentation is on using Behat 3 as the basis for testing a Drupal 7 installation, using Mink, Selenium, the Behat Drupal Extension, and some other home grown tools to get the testing done. This presentation does not cover installation of said tools, but I’ve included links at the end of the presentation to resources for that - please let me know afterwords if you need help and I’ll assist if I can. This presentation will give a rough overview of the listed tools and the roles they serve, and then delve immediately into the customizations and lessons we at Eelzee found most useful for our particular setup. Not everything in this presentation is going to be generally useful to folks, I’m hoping a discussion of the journey might help those of you who are considering the same.

---------------------------------------------

## Why not Simpletest?

- [meta] Replace the testing framework with PHPUnit and possibly rewrite Simpletest1
- "Since http://drupal.org/node/1567500 Drupal 8 started to use PHPUnit as it's unit test framework. One advantage of PHPUnit is that there are tools around which support it already." - Daniel Wehner2


Speaker Notes:
The tools named on the prior page might be foreign to many here, but SimpleTest shouldn’t be.  Before we start talking about what those listed tools are and how they’ll help us, i should mention why we leaned away from SimpleTest.

On this slide I’ve got a thread title and a quote in this page, with sources footnoted below.  The first is the meta thread titled “Replace the testing framework with PHPUnit and possibly rewrite Simpletest”, whose url is in the footnotes below.  The long term suggestions for this thread include using PHPUnit for unit tests, and Mink+Guzzle+Goutte for web/integration tests, and possibly Behat for  BDD tests.  This thread is the primary rationale for our choice, although it was backed up by other sources like the quote that follows it.  We adopted these tools primarily so that we can write testing that is more or less forward compatible with the future plans of Drupal. 

We’ve been focusing entirely on the functional side of things, so I’ll only be addressing that in the talk today.  I’m a long way from being able to give a talk on PHPUnit with any feeling like I’m passing on a best practice, so I’ll defer that talk for others.