---
layout: post
title: Testing Microservices
tags: programming testing
excerpt: Learn the concepts you need to build a reliable, fast and effective test suite for your microservices
comments: true
---

<div class="highlight">
  <strong>Update:</strong> Prefer to read this on your e-reader? This article is now available as an <a href="/blog/ebook-testing-microservices/">e-book version</a>
</div>

Microservices have been all the rage for a while. If you attended any tech conference or read software engineering blogs lately, you'll either be amazed or fed up with all the stories that companies love to share about their microservices journey.

Behind all the hype are some true advantages to adopting a microservice architecture. And of course -- as with every architecture decision -- there will be trade-offs. I won't give you a lecture about the benefits and drawbacks of microservices or whether you should use them. Others have done [a way better job](https://www.martinfowler.com/microservices) at breaking this down than I could. Chance is, if you're reading this you somehow ended up with the decision to take a look into microservices and what it means to test them.

This post is part one of a two parts series. The post you're reading right now explains high-level concepts and what type of tests you should have for your microservices. It also introduces you to a number of resources that you can use to learn more about the field.

You want to get more hands on (and you're not afraid of Java)? Take a look at the [second part](/blog/testing-java-microservices/) where we look at a sample microservice codebase and see how the concepts we learned in this post can be implemented.

This is a long post, go grab a cup of coffee and take your time. Maybe add it to your bookmarks and come back later. This post tries to be exhaustive, still it's just a starting point. Go ahead and dive deeper into the world of test automation using the linked resources. And most importantly: Start, experiment and learn what it means to test microservices on your own. You won't get it all correct from the beginning. That's ok. Start with best intentions, be diligent and explore!

Ready? Let's rock!

## <abbr title="too long; didn't read">tl;dr</abbr>
Here's what you'll take away from this post:

<div class="highlight">
    <ol>
	<li>Automate your tests (whoa, surprise!)</li>
	<li>Continuous delivery makes your life easier</li>
	<li>Remember the <a href="https://martinfowler.com/bliki/TestPyramid.html">test pyramid</a> <em>(don't be too confused by the original names of the layers, though)</em></li>
	<li>Write tests with different granularity</li>
	<li>Use unit test to test the insides of your application</li>
	<li>Use integration tests to test data serialization/deserialization</li>
	<li>Test collaboration between services with contract tests (CDC)</li>
	<li>Use end-to-end tests sparingly, limit to high-value user journeys</li>
	<li>Don't just test from a developer's perspective, make sure to test features from a user's perspective as well</li>
	<li>Exploratory testing will spot issues your build pipeline didn't catch</li>
    </ol>
</div>

{% include table_of_contents.html %}

## Microservices Need (Test) Automation
Microservices go hand in hand with **continuous delivery**, a practice where you automatically ensure that your software can be released into production any time. You use a **build pipeline** to automatically test and deploy your application to your testing and production environments.

Once you advance on your microservices quest you'll be juggling dozens, maybe even hundreds of microservices. At this point building, testing and deploying these services manually becomes impossible -- unless you want to spend all your time with manual, repetitive work instead of delivering working software. Automating everything -- from build to tests, deployment and infrastructure -- is your only way forward.

![build pipeline](/assets/img/uploads/buildPipeline.png)
*Use build pipelines to automatically and reliably get your software into production*

Most microservices success stories are told by teams who use continuous delivery or **continuous deployment** (every software change that's proven to be releasable will be deployed to production). These teams make sure that changes get into the hands of their customers quickly.

How do you proof that your latest change still results in releasable software? You test your software including the latest change thoroughly. Traditionally you'd do this manually by deploying your application to a test environment and then performing some black-box style testing e.g. by clicking through your user interface to see if anything's broken.

It's obvious that testing all changes manually is time-consuming, repetitive and tedious. Repetitive is boring, boring leads to mistakes and makes you look for a different job by the end of the week.

Luckily there's a remedy for repetitive tasks: **automation**.

Automating your tests can be a big game changer in your life as a software developer. Automate your tests and you no longer have to mindlessly follow click protocols in order to check if your software still works correctly. Automate your tests and you can change your codebase without batting an eye. If you've ever tried doing a large-scale refactoring without a proper test suite I bet you know what a terrifying experience this can be. How would you know if you accidentally broke stuff along the way? Well, you click through all your manual test cases, that's how. But let's be honest: do you really enjoy that? How about making even large-scale changes and knowing whether you broke stuff within seconds while taking a nice sip of coffee? Sounds more enjoyable if you ask me.

Automation in general and test automation specifically are essential to building a successful microservices architecture. Do yourself a favor and take a look at the concepts behind continuous delivery ([the Continuous Delivery book](https://www.amazon.com/gp/product/0321601912) is my go to resource). You will see that diligent automation allows you to deliver software faster and more reliable. Continuous delivery paves the way into a new world full of fast feedback and experimentation. At the very least it makes your life as a developer more peaceful.

## The Test Pyramid
If you want to get serious about automated tests for your software there is one key concept that you should know about: the **test pyramid**. Mike Cohn came up with this concept in his book [Succeeding with Agile](https://www.amazon.com/dp/0321579364/ref=cm_sw_r_cp_dp_T2_bbyqzbMSHAG05). It's a great visual metaphor telling you to think about different layers of testing. It also tells you how much testing to do on each layer.

![Test Pyramid](/assets/img/uploads/testPyramid.png)
_The test pyramid_

Mike Cohn's original test pyramid consists of three layers that your test suite should consist of (bottom to top):

  1. Unit Tests
  2. Service Tests
  3. User Interface Tests

Unfortunately the concept of the test pyramid falls a little short if you take a closer look. [Some argue](https://watirmelon.blog/2011/06/10/yet-another-software-testing-pyramid/) that either the naming or some conceptual aspects of Mike Cohn's test pyramid are not optimal, and I have to agree. From a modern point of view the test pyramid seems overly simplistic and can therefore be a bit misleading.

Still, due to it's simplicity the essence of the test pyramid serves as a good rule of thumb when it comes to establishing your own test suite. Your best bet is to remember two things from Cohn's original test pyramid:

  1. Write tests with different granularity
  2. The more high-level you get the fewer tests you should have

Stick to the pyramid shape to come up with a healthy, fast and maintainable test suite: Write _lots_ of small and fast _unit tests_. Write _some_ more coarse-grained tests and _very few_ high-level tests that test your application from end to end. Watch out that you don't end up with a [test ice-cream cone](https://watirmelon.blog/2012/01/31/introducing-the-software-testing-ice-cream-cone/) that will be a nightmare to maintain and takes way too long to run.

Don't become too attached to the names of the individual layers in Cohn's test pyramid. In fact they can be quite misleading: _service test_ is a term that is hard to grasp (Cohn himself talks about the observation that [a lot of developers completely ignore this layer](https://www.mountaingoatsoftware.com/blog/the-forgotten-layer-of-the-test-automation-pyramid)). In the days of modern single page application frameworks like react, angular, ember.js and others it becomes apparent that _UI tests_ don't have to be on the highest level of your pyramid -- you're perfectly able to unit test your UI in all of these frameworks.

Given the shortcomings of the original names it's totally okay to come up with other names for your test layers, as long as you keep it consistent within your codebase and your team's discussions.

## Types of Tests
While the test pyramid suggests that you'll have three different types of tests (_unit tests_, _service tests_ and _UI tests_) I need to disappoint you. Your reality will look a little more diverse. Lets keep Cohn's test pyramid in mind for its good things (use test layers with different granularity, make sure they're differently sized) and find out what types of tests we need for an effective test suite.

### Unit tests
The foundation of your test suite will be made up of unit tests. Your unit tests make sure that a certain unit (your _subject under test_) of your codebase works as intended. The number of unit tests in your test suite will largely outnumber any other type of test.

![unit tests](/assets/img/uploads/unitTest.png)
*A unit test typically replaces external collaborators with mocks or stubs*

#### What's a Unit?
If you ask three different people what _"unit"_ means in the context of unit tests, you'll probably receive four different, slightly nuanced answers. To a certain extend it's a matter of your own definition and it's okay to have no canonical answer.

If you're working in a functional language a _unit_ will most likely be a single function. Your unit tests will call a function with different parameters and ensure that it returns the expected values. In an object-oriented language a unit can range from a single method to an entire class.

#### Sociable and Solitary
Some argue that all collaborators (e.g. other classes that are called by your class under test) of your subject under test should be substituted with _mocks_ or _stubs_ to come up with perfect isolation and to avoid side-effects and complicated test setup. Others argue that only collaborators that are slow or have bigger side effects (e.g. classes that access databases or make network calls) should be stubbed or mocked.

[Occasionally](https://www.martinfowler.com/bliki/UnitTest.html) people label these two sorts of tests as **solitary unit tests** for tests that stub all collaborators and **sociable unit tests** for tests that allow talking to real collaborators (Jay Fields' [Working Effectively with Unit Tests](https://leanpub.com/wewut) coined these terms). If you have some spare time you can go down the rabbit hole and [read more about the pros and cons](https://martinfowler.com/articles/mocksArentStubs.html) of the different schools of thought.

At the end of the day it's not important to decide if you go for solitary or sociable unit tests. Writing automated tests is what's important. Personally, I find myself using both approaches all the time. If it becomes awkward to use real collaborators I will use mocks and stubs generously. If I feel like involving the real collaborator gives me more confidence in a test I'll only stub the outermost parts of my service.

#### Mocking and Stubbing
**Mocking** and **stubbing** ([there's a difference](https://martinfowler.com/articles/mocksArentStubs.html) if you want to be precise) should be heavily used instruments in your unit tests.

In plain words it means that you replace a real thing (e.g. a class, module or function) with a fake version of that thing. The fake version looks and acts like the real thing (answers to the same method calls) but answers with canned responses that you define yourself at the beginning of your unit test.

Regardless of your technology choice, there's a good chance that either your language's standard library or some popular third-party library will provide you with elegant ways to set up mocks. And even writing your own mocks from scratch is only a matter of writing a fake class/module/function with the same signature as the real one and setting up the fake in your test.

Your unit tests will run very fast. On a decent machine you can expect to run thousands of unit tests within a few minutes. Test small pieces of your codebase in isolation and avoid hitting databases, the filesystem or firing HTTP queries (by using mocks and stubs for these parts) to keep your tests fast.

Once you got a hang of writing unit tests you will become more and more fluent in writing them. Stub out external collaborators, set up some input data, call your subject under test and check that the returned value is what you expected. Look into [Test-Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) and let your unit tests guide your development; if applied correctly it can help you get into a great flow and come up with a good and maintainable design while automatically producing a comprehensive and fully automated test suite. Still, it's no silver bullet. Go ahead, give it a real chance and see if it feels right for you.


#### Unit Testing is Not Enough
A good unit test suite will be immensely helpful during development: You know that all the small units you tested are working correctly in isolation. Your small-scoped unit tests help you narrowing down and reproducing errors in your code. On top they give you fast feedback while working with the codebase and will tell you whether you broke something unintendedly. Consider them as a tool _for developers_ as they are written from the developer's point of view and make their job easier.

Unfortunately writing unit alone won't get you very far. With unit tests you don't know whether your application as a whole works as intended. You don't know whether the features your customers love actually work. You don't know if you did a proper job plumbing and wiring all those components, classes and modules together.

Maybe there's something funky happening once all your small units join forces and work together as a bigger system. Maybe your code works perfectly fine when running against a mocked database but fails when it's supposed to write data to a real database. And maybe you wrote perfectly elegant and well-crafted code that totally fails to solve your users problem. Seems like we need more in order to spot these problems.

### Integration Tests
All non-trivial applications will integrate with some other parts (databases, filesystems, network, and other services in your microservices landscape). When writing unit tests these are usually the parts you leave out in order to come up with better isolation and fast tests. Still, your application will interact with other parts and this needs to be tested. _Integration tests_ are there to help. They test the integration of your application with all the parts that live outside of your application.

Integration tests live at the boundary of your service. Conceptually they're always about triggering an action that leads to integrating with the outside part (filesystem, database, etc). A database integration test would probably look like this:

![a database integration test](/assets/img/uploads/dbIntegrationTest.png)
*A database integration test integrates your code with a real database*

    1. start a database
    2. connect your application to the database
    3. trigger a function within your code that writes data to the database
    4. check that the expected data has been written to the database by reading the data from the database


Another example, an integration test for your REST API could look like this:

![an HTTP integration test](/assets/img/uploads/httpIntegrationTest.png)
*An HTTP integration test checks that real HTTP calls hit your code correctly*

    1. start your application
    2. fire an HTTP request against one of your REST endpoints
    3. check that the desired interaction has been triggered within your application


Your integration tests -- like unit tests -- can be fairly whitebox. Some frameworks allow you to start your application while still being able to mock some other parts of your application so that you can check that the correct interactions have happened.

Write integration tests for all pieces of code where you either _serialize_ or _deserialize_ data. In a microservices architecture this happens more often than you might think. Think about:

  * Calls to your services' REST API
  * Reading from and writing to databases
  * Calling other microservices
  * Reading from and writing to queues
  * Writing to the filesystem

Writing integration tests around these boundaries ensures that writing data to and reading data from these external collaborators works fine.

If possible you should prefer to run your external dependencies locally: spin up a local MySQL database, test against a local ext4 filesystem. In some cases this won't be easy. If you're integrating with third-party systems from another vendor you might not have the option to run an instance of that service locally (though you should try; talk to your vendor and try to find a way).

If there's no way to run a third-party service locally you should opt for running a dedicated test instance somewhere and point at this test instance when running your integration tests. Avoid integrating with the real production system in your automated tests. Blasting thousands of test requests against a production system is a surefire way to get people angry because you're cluttering their logs (in the best case) or even <abbr title="Denial of Service">DoS</abbr>'ing their service (in the worst case).

With regards to the test pyramid, integration tests are on a higher level than your unit tests. Integrating slow parts like filesystems and databases tends to be much slower than running unit tests with these parts stubbed out. They can also be harder to write than small and isolated unit tests, after all you have to take care of spinning up an external part as part of your tests. Still, they have the advantage of giving you the confidence that your application can correctly work with all the external parts it needs to talk to. Unit tests can't help you with that.

### UI Tests
Most applications have some sort of user interface. Typically we're talking about a web interface in the context of web applications. People often forget that a REST API or a command line interface is as much of a user interface as a fancy web user interface.

_UI tests_ test that the user interface of your application works correctly. User input should trigger the right actions, data should be presented to the user, the UI state should change as expected.

![user interface tests](/assets/img/uploads/ui_tests.png)

UI Tests and end-to-end tests are sometimes (as in Mark Cohn's case) said to be the same thing. For me this conflates two things that are not _necessarily_ related.

Yes, testing your application end-to-end often means driving your tests through the user interface. The inverse, however, is not true.

Testing your user interface doesn't have to be done in an end-to-end fashion. Depending on the technology you use, testing your user interface can be as simple as writing some unit tests for your frontend javascript code with your backend stubbed out.

With traditional web applications testing the user interface can be achieved with tools like [Selenium](http://docs.seleniumhq.org/). If you consider a REST API to be your user interface you should have everything you need by writing proper integration tests around your API.

With web interfaces there's multiple aspects that you probably want to test around your UI: behaviour, layout, usability or adherence to your corporate design are only a few.

Fortunally, testing the **behaviour** of your user interface is pretty simple. You click here, enter data there and want the state of the user interface to change accordingly. Modern single page application frameworks ([react](https://facebook.github.io/react/), [vue.js](https://vuejs.org/), [Angular](https://angular.io/) and the like) often come with their own tools and helpers that allow you to thorougly test these interactions in a pretty low-level (unit test) fashion. Even if you roll your own frontend implementation using vanilla javascript you can use your regular testing tools like [Jasmine](https://jasmine.github.io/) or [Mocha](http://mochajs.org/). With a more traditional, server-side rendered application, Selenium-based tests will be your best choice.

Testing that your web application's **layout** remains intact is a little harder. Depending on your application and your users' needs you may want to make sure that code changes don't break the website's layout by accident.

The problem is that computers are notoriously bad at checking if something "looks good" (maybe some clever machine learning algorithm can change that in the future).

There are some tools to try if you want to automatically check your web application's design in your build pipeline. Most of these tools utilize Selenium to open your web application in different browsers and formats, take screenshots and compare these to previously taken screenshots. If the old and new screenshots differ in an unexpected way, the tool will let you know.

[Galen](http://galenframework.com/) is one of these tools. But even rolling your own solution isn't too hard if you have special requirements. Some teams I've worked with built [lineup](https://github.com/otto-de/lineup) and its Java-based cousin [jlineup](https://github.com/otto-de/jlineup) to achieve something similar. Both tools take the same Selenium-based approach I described before.

Once you want to test for **usability** and a "looks good" factor you leave the realms of automated testing. This is the area where you should rely on [exploratory testing](https://en.wikipedia.org/wiki/Exploratory_testing), usability testing (this can even be as simple as [hallway testing](https://en.wikipedia.org/wiki/Usability_testing#Hallway_testing)) and showcases with your users to see if they like using your product and can use all features without getting frustrated or annoyed.

### Contract Tests
One of the big benefits of a microservice architecture is that it allows your organisation to scale their development efforts quite easily. You can spread the development of microservices across different teams and develop a big system consisting of multiple loosely coupled services without stepping on each others toes.

Splitting your system into many small services often means that these services need to communicate with each other via certain (hopefully well-defined, sometimes accidentally grown) interfaces.

Interfaces between microservices can come in different shapes and technologies. Common ones are

  * REST and JSON via HTTPS
  * <abbr title="remote procedure calls">RPC</abbr> using something like [gRPC](https://grpc.io/)
  * building an event-driven architecture using queues

For each interface there are two parties involved: the **provider** and the **consumer**. The provider serves data to consumers. The consumer processes data obtained from a provider. In a REST world a provider builds a REST API with all required endpoints; a consumer makes calls to this REST API to fetch data or trigger changes in the other service. In an asynchronous, event-driven world, a provider (often rather called **publisher**) publishes data to a queue; a consumer (often called **subscriber**) subscribes to these queues and reads and processes data.

![contract tests](/assets/img/uploads/contract_tests.png)
_Each interface has a providing (or publishing) and a consuming (or subscribing) party. The specification of an interface can be considered a contract._

As you often spread the consuming and providing services across different teams you find yourself in the situation where you have to clearly specify the interface between these services (the so called **contract**). Traditionally companies have approached this problem in the following way:

  1. Write a long and detailed interface specification (the _contract_)
  2. Implement the providing service according to the defined contract
  3. Throw the interface specification over the fence to the consuming team
  4. Wait until they implement their part of consuming the interface
  5. Run some large-scale manual system test to see if everything works
  6. Hope that both teams stick to the interface definition forever and don't screw up

If you're not stuck in the dark ages of software development, you hopefully have replaced steps _5._ and _6._ with something more automated. Automated contract tests make sure that the implementations on the consumer and provider side still stick to the defined contract. They serve as a good regression test suite and make sure that deviations from the contract will be noticed early.

In a more agile organisation you should take the more efficient and less wasteful route. All your microservices live within the same organisation. It really shouldn't be too hard to talk to the developers of the other services directly instead of throwing overly detailed documentation over the fence. After all they're your co-workers and not a third-party vendor that you could only talk to via customer support or legally bulletproof contracts.

**Consumer-Driven Contract tests** (**CDC tests**) let the consumers drive the implementation of a contract. Using CDC, consumers of an interface write tests that check the interface for all data they need from that interface. The consuming team then publishes these tests so that the publishing team can fetch and execute these tests easily. The providing team can now develop their API by running the CDC tests. Once all tests pass they know they have implemented everything the consuming team needs.

![CDC tests](/assets/img/uploads/cdc_tests.png)
_Contract tests ensure that the provider and all consumers of an interface stick to the defined interface contract. With CDC tests consumers of an interface publish their requirements in the form of automated tests; the providers fetch and execute these tests continuously_

This approach allows the providing team to implement only what's really necessary (keeping things simple, <abbr title="You ain't gonna need it">YAGNI</abbr> and all that). The team providing the interface should fetch and run these CDC tests continuously (in their build pipeline) to spot any breaking changes immediately. If they break the interface their CDC tests will fail, preventing breaking changes to go live. As long as the tests stay green the team can make any changes they like without having to worry about other teams.

The Consumer-Driven Contract approach would leave you with a process looking like this:

  1. The consuming team writes automated tests with all consumer expectations
  3. They publish the tests for the providing team
  4. The providing team runs the CDC tests continuously and keeps them green
  5. Both teams talk to each other once the CDC tests break

If your organisation adopts microservices, having CDC tests is a big step towards establishing autonomous teams. CDC tests are an automated way to foster team communication. They ensure that interfaces between teams are working at any time. Failing CDC tests are a good indicator that you should walk over to the affected team, have a chat about any upcoming API changes and figure out how you want to move forward.

A naive implementation of CDC tests can be as simple as firing requests against an API and assert that the responses contain everything you need. You then package these tests as an executable (.gem, .jar, .sh) and upload it somewhere the other team can fetch it (e.g. an artifact repository like [Artifactory](https://www.jfrog.com/artifactory/)).

Over the last couple of years the CDC approach has become more and more popular and several tools been build to make writing and exchanging them easier.

[Pact](https://github.com/realestate-com-au/pact) is probably the most prominent one these days. It has a sophisticated approach of writing tests for the consumer and the provider side, gives you stubs for third-party services out of the box and allows you to exchange CDC tests with other teams. Pact has been ported to a lot of platforms and can be used with JVM languages, Ruby, .NET, JavaScript and many more.

If you want to get started with CDCs and don't know how, Pact can be a sane choice. The [documentation](https://docs.pact.io/) can be overwhelming at first. Be patient and work through it. It helps to get a firm understanding for CDCs which in turn makes it easier for you to advocate for the use of CDCs when working with other teams. You can also find a hands-on example in the [second part of this series](/blog/testing-java-microservices/).

Consumer-Driven Contract tests can be a real game changer as you venture further on your microservices journey. Do yourself a favor, read up on that concept and give it a try. A solid suite of CDC tests is invaluable for being able to move fast without breaking other services and cause a lot of frustration with other teams.

### End-to-End Tests
Testing your deployed application via its user interface is the most end-to-end way you could test your application. The previously described, webdriver driven UI tests are a good example of end-to-end tests.

![an end-to-end test](/assets/img/uploads/e2etests.png)
_End-to-end tests test your entire, completely integrated system_

End-to-end tests give you the biggest confidence when you need to decide if your software is working or not. [Selenium](http://docs.seleniumhq.org/) and the [WebDriver Protocol](https://www.w3.org/TR/webdriver/) allow you to automate your tests by automatically driving a (headless) browser against your deployed services, performing clicks, entering data and checking the state of your user interface. You can use Selenium directly or use tools that are build on top of it, [Nightwatch](http://nightwatchjs.org/) being one of them.

End-to-End tests come with their own kind of problems. They are notoriously flaky and often fail for unexpected and unforseeable reasons. Quite often their failure is a false positive. The more sophisticated your user interface, the more flaky the tests tend to become. Browser quirks, timing issues, animations and unexpected popup dialogs are only some of the reasons that got me spending more of my time with debugging than I'd like to admit.

In a microservices world there's also the big question of who's in charge of writing these tests. Since they span multiple services (your entire system) there's no single team responsible for writing end-to-end tests.

If you have a centralised _quality assurance_ team they look like a good fit. Then again having a centralised QA team is a big anti-pattern and shouldn't have a place in a DevOps world where your teams are meant to be truly cross-functional. There's no easy answer who should own end-to-end tests. Maybe your organisation has a community of practice or a _quality guild_ that can take care of these. Finding the correct answer highly depends on your organisation.

Furthermore, end-to-end tests require a lot of maintenance and run pretty slowly. Once you have more than a couple of microservices in place you won't even be able to run your end-to-end tests locally -- as this would require to start all your microservices locally as well. Good luck spinning up hundreds of microservices on your development machine without frying your RAM.

Due to their high maintenance cost you should aim to reduce the number of end-to-end tests to a bare minimum.

Think about the high-value interactions users will have with your application. Try to come up with user journeys that define the core value of your product and translate the most important steps of these user journeys into automated end-to-end tests.

If you're building an e-commerce site your most valuable customer journey could be a user searching for a product, putting it in the shopping basket and doing a checkout. That's it. As long as this journey still works you shouldn't be in too much trouble. Maybe you'll find one or two more crucial user journeys that you can translate into end-to-end tests. Everything more than that will likely be more painful than helpful.

Remember: you have lots of lower levels in your test pyramid where you already tested all sorts of edge cases and integrations with other parts of the system. There's no need to repeat these tests on a higher level. High maintenance effort and lots of false positives will slow you don't and make sure you'll lose trust in your tests rather sooner than later.

### Acceptance Tests -- Do Your Features Work Correctly?
The higher you move up in your test pyramid the more likely you enter the realms of testing whether the features you're building work correctly from a user's perspective. You can treat your application as a black box and shift  the focus in your tests from

> when I enter the values `x` and `y`, the return value should be `z`

towards

> _given_ there's a logged in user  
> _and_ there's an article "bicycle"  
> _when_ the user navigates to the "bicycle" article's detail page  
> _and_ clicks the "add to basket" button  
> _then_ the article "bicycle" should be in their shopping basket

Sometimes you'll hear the terms [**functional test**](https://en.wikipedia.org/wiki/Functional_testing) or [**acceptance test**](https://en.wikipedia.org/wiki/Acceptance_testing#Acceptance_testing_in_extreme_programming) for these kinds of tests. Sometimes people will tell you that functional and acceptance tests are different things. Sometimes the terms are conflated. Sometimes people will argue endlessly about wording and definitions. Often this discussion is a pretty big source of confusion.

Here's the thing: At one point you should make sure to test that your software works correctly from a _user's_ perspective, not just from a technical perspective. What you call these tests is really not that important. Having these tests, however, is. Pick a term, stick to it, and write those tests.

This is also the moment where people talk about <abbr title="Behaviour-Driven Development">BDD</abbr> and tools that allow you to implement tests in a BDD fashion. BDD or a BDD-style way of wrtiting tests can be a nice trick to shift your mindset from implementation details towards the users' needs. Go ahead and give it a try.

You don't even need to adopt full-blown BDD tools like [Cucumber](https://cucumber.io/) (though you can). Some assertion libraries (like [chai.js](http://chaijs.com/guide/styles/#should) allow you to write assertions with `should`-style keywords that can make your tests read more BDD-like. And even if you don't use a library that provides this notation, clever and well-factored code will allow you to write user behaviour focused tests. Some helper methods/functions can get you a very long way:

{% highlight python %}
def test_add_to_basket():
    # given
    user = a_user_with_empty_basket()
    user.login()
    bicycle = article(name="bicycle", price=100)

    # when
    article_page.add_to_.basket(bicycle)

    # then
    assert user.basket.contains(bicycle)
{% endhighlight %}

Acceptance tests can come in different levels of granularity. Most of the time they will be rather high-level and test your service through the user interface. However, it's good to understand that there's technically no need to write acceptance tests at the highest level of your test pyramid. If your application design and your scenario at hand permits that you write an acceptance test at a lower level, go for it. Having a low-level test is better than having a high-level test. The concept of acceptance tests -- proving that your features work correctly for the user -- is completely orthogonal to your test pyramid.

### Exploratory Testing
Even the most diligent test automation efforts are not perfect. Sometimes you miss certain edge cases in your automated tests. Sometimes it's nearly impossible to detect a particular bug by writing a unit test. Certain quality issues don't even become apparent within your automated tests (think about design or usability). Despite your best intentions with regards to test automation, manual testing of some sorts is still a good idea.

![exploratory testing](/assets/img/uploads/exploratoryTesting.png)
_Use exploratory testing to spot all quality issues that your build pipeline didn't spot_

Include [Exploratory Testing](https://en.wikipedia.org/wiki/Exploratory_testing) in your testing portfolio. It is a manual testing approach that emphasizes the tester's freedom and creativity to spot quality issues in a running system. Simply take some time on a regular schedule, roll up your sleeves and try to break your application. Use a destructive mindset and come up with ways to provoke issues and errors in your application. Document everything you find for later. Watch out for bugs, design issues, slow response times, missing or misleading error messages and everything else that would annoy you as a user of your software.

The good news is that you can happily automate most of your findings with automated tests. Writing automated tests for the bugs you spot makes sure there won't be any regressions of that bug in the future. Plus it helps you narrowing down the root cause of that issue during bugfixing.

During exploratory testing you will spot problems that slipped through your build pipeline unnoticed. Don't be frustrated. This is great feedback on the maturity of your build pipeline. As with any feedback, make sure to act on it: Think about what you can do to avoid these kinds of problems in the future. Maybe you're missing out on a certain set of automated tests. Maybe you have just been sloppy with your automated tests in this iteration and need to test more thoroughly in the future. Maybe there's a shiny new tool or approach that you could use in your pipeline to avoid these issues in the future. Make sure to act on it so your pipeline and your entire software delivery will grow more mature the longer you go.

## Avoid Test Duplication
Now that you know that you should write different types of tests there's one more pitfall to avoid: test duplication. While your gut feeling might say that there's no such thing as too many tests let me assure you, there is. Every single test in your test suite is additional baggage and doesn't come for free. Writing and maintaining tests takes time. Reading and understanding other people's test takes time. And of course, running tests takes time.

As with production code you should strive for simplicity and avoid duplication. If you managed to test all of your code's edge cases on a unit level there's no need to test these edge cases again on a higher-level. Keep this as a rule of thumb.

If your high-level test adds additional value (e.g. testing the integration with a real database) than it's a good idea to have this higher level test even though you might have tested the same database access function in a unit test. Just make sure to focus on the integration part in that test and avoid going through all possible edge-cases again.

Duplicating tests can be quite tempting, especially when you're new to test automation. Be aware of the additional cost and don't be afraid to delete tests if you were able to replace them with lower level tests or if they no longer provide any value.


## Summary
You did it! You made it through a huge wall of text explaining why and how you should test your microservices. Remember the test pyramid, write unit, integration and high-level tests and get your hands dirty with this CDC thingy!

I know, this is a lot to digest. Hopefully this wasn't all new to you so you can focus on the things that are new to you. If you're craving for code now and want to see all these concepts in action, my [second post](/blog/testing-java-microservices) should be your next stop.

If you feel that you need more in-depth information on a certain topic, take a look at my list of further resources. They should keep you busy for a while.

Happy testing!

## Further reading

  * ***Building Microservices*** **by Sam Newman**  
  This book contains so much more there is to know about building microservices. A lot of the ideas in this article can be found in this book as well. The chapter about testing is available as a free sample [over at O'Reilly](https://opds.oreilly.com/learning/building-microservices-testing).
  * ***Continuous Delivery*** **by Jez Humble and Dave Farley**  
  The canonical book on continuous delivery. Contains a lot of useful information about build pipelines, test and deployment automation and the cultural mindset around CD. This book has been a real eye opener in my career.
  * ***[Working Effectively with Unit Tests](https://leanpub.com/wewut)*** **by Jay Fields**  
  If you level up your unit testing skills or read more about mocking, stubbing, sociable and solitary unit tests, this is your resource.
  * ***[Testing Microservices](https://martinfowler.com/articles/microservice-testing)*** **by Toby Clemson**  
  A fantastic slide deck with a lot of useful information about the different considerations when testing a microservice. Has lots of nice diagrams to show what boundaries you should be looking at.
  * ***Growing Object-Oriented Software Guided by Tests*** by **Steve Freeman and Nat Pryce**  
  If you're still trying to get your head around this whole testing thing (and ideally are working with Java) this is the single book you should be reading right now.
  * ***Test-Driven Development: By example*** by **Kent Beck**  
  The classic TDD book by Kent Beck. Demonstrates on a hands-on walkthrough how you TDD your way to working software.
