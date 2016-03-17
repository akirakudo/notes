### @searls ruby conf talk

https://www.youtube.com/watch?v=VD51AkG8EZw


From the perspective of prevention he discusses

####  test structure

* too big to fail: tests make big objects harder to manager
    * big objects => big tests
    * many dependencies => hard to do test setup
    * multiple side-effects and return values => lots of verifications
    * many logical branches => have to write many test cases
    * rule of product: the total no. of test cases you have to write is the no. of possible values of each arg *multiplied* together
        ```
        foo(a, b, c)

        Na = no. of possible values of a
        Nb = ...

        num test cases = Na * Nb * Nc
        ```
    * if you are trying to get into testing you first have to get into small objects
    * he limits new object to one public method and 3 dependencies
* every test does the same three things
    1. set stuff up (given)
    2. invoke (when)
    3. verify behaviour and/or state (then)
    * every test is a program that does the same stuff
* rspec
    * let are in the "given" section
    * before could be either given or when
* he puts an empty line between each given/when/then section in every xUnit style test he writes
* he recommends writing tests in a given-when-then conscious way
* smells
    * lots of given steps => object may is hard to setup
    * more than one when step => maybe API is confusing or hard to invoke/understand
    * many then steps => code is doing too much or returning too complex a type
* tests are untested code so test code logic means you spend time trying to figure out the logic not see what it is testing
* he has a squint test
    * can I easily see what is under test
    * are all the methods tested?
    * are they in order?
    * arrange - act - assert should "pop"
* he uses rspec context to point out each logical branch of the method
* tests that are too magic
    * testing libs exist along a spectrum
    * small API, not many features, quicker to learn <--------> large API, lots of features, takes longer to learn
    * aka mintest vs rspec
    * bigger API
        * ++ terse test
        * -- takes longer to learn, looks like magic to new people
    * smaller API
        * -- more "one-off" test helpers
        * ++ quicker (easier?) to learn
* he always calls the thing under test `subject` and always call the thing he will do assertions on `result` or `results`
* consistency is good
    * if you are consistent in your test suite, any inconstency can convey meaning to the reader
    * if your test suite is inconsistent then readers have to read *everything* carefully
* make unimportant test code look obviously silly to the reader
    * test data should be minimal but also minimally meaningful

#### test isolation

* unfocused test suites
    * most teams define a sccuessful test suite as "is it tested"
    * he defines a successful test suite by
        1. is the purpose of each test readily apparent
        1. does the test suite promote consistency?

* he creates a separate suite for each "kind" of test in the suite
    * each suite has its own conventions and consistency
    * creates custome spec helpers for the suite to enforce and encourage that consistency
* TODO http://blog.testdouble.com/posts/2014-05-25-breaking-up-with-your-test-suite.html
* thinking about the agile testing pryamid
    * top = tests are more integrated
    * bottom = tests are less integrated
* a test suite should stay at a consistent height on the pyramid
* be consistent in
    * does this test suite fake 3rd party APIs or not?
    * does this test suite fake out all collaborators or not?
    * does this test suite use the UI or not?
* he recommends starting with two suites
    1. as integrated as we can manage
        * fake as little as we can
        * use UI if it is available
    2. as isolated as possible
    * it sets up two extremes for when he is deciding which suite to put a test in rather than oscillating in-between
* as need arsies you might need to make more suites in the middle but you need to agree on the conventions of the suite before you do it

* tests can be too realistic
    * some people have a goal of their tests being maximally realisistic
    * any "realistic" test has implicit/hidden boundaries on how realisitic it can be e.g. does the test use your production DNS, CDN etc.
    * in reality "real" is impossible to achieve
    * then when something fails, folks check against their goal of "maximum realism" and their answer is to increase the "realism" aka "integratedness" of the tests.
    * realisitc tests have real costs:
        * slower
        * take more time to write, change, debug
        * can fail for reasons other than the code failed
        * more cognitive load to understand them
    * if the test suite has really clear boundaries (saying what it does and does not test) then you can focus on what is tested and what is controlled
    * then when stuff breaks in production
        * team can have a conversation about the trade-offs of why a test didnt' cover that case
        * *maybe* make a separate test (maybe a new suite) to cover that case)
        * don't have to do things that push even harder for the unreachable "fully realistic" goal at an unacceptable cost
    * realism in tests is not an ideal or virtue
        * isolated tests give us much better design feedback

* redundant code coverage
    * redundant test coverage can really kill a team morale
    * you can see the reduendant coverage from the "avg hits/line" column in the coverage
    * outside-in TDD can help with this (use lots of mocks)
        * TODO http://blog.testdouble.com/posts/2015-09-10-how-i-use-test-doubles.html
* careless mocking
    * he uses mocks for API discovery (fake all dependencies that don't exist yet)
    * he strong advises
        * not using a mix of real and fake objects
        * don't use test doubles just for the few objects that are hard to setup
    * haphazard mocking
        * confuses the reader
        * treat the symptom of test pain not the cause
            * if some of your objects are hard to setup that is a symptom not the cause - try to treat the cause by fixing those objects
* app frameworks
    * app frameworks provide a lot of "integration level concerns" - they help you plug you domain models into the rest of the world
    * frameworks focus on integration problems
    * framewroks test helpers are very integrated
    * if you look at the framework as the "giver of all things I need" then we end up writing all integration tests
    * he suggests having a suite that couples to the framework and one that does not
        * rspec encourages this with rails these days with `spec_helper.rb` vs `rails_helper.rb`

#### test feedback

* bad error messages waste time when tests fail
    * error output is a very important feature of assertion libraries
* slow loops
    * recommends monitoring my own feedback loop
* painful test data
    * there are different levels of "control" of test data
        * inline
            * good for models
        * fixtures
            * good for integration tests
        * data dump
            * good for smoke tests
            * can be faster than factories
        * "self priming tests"
            * you have to use the app UI to get the app into a state that you can run the test
            * you have to do this if you need to test something in production
* superlinear build slowdown
    * the time a single test takes to run is
        * app time + test time + setup and teardown time
    * as the app gets bigger, app time and setup/teardown time will too so the time taken to run the same test will increase even though that test has not changed at all!
        * he recommends monitoring build time
        * also can cap build time and when the suite approaches it delete or refactor tests to stay within it
    * avoid the urge to create a new integration test for each feature
        * try to have integration tests that zig-zag through features - this is more like what users do anyway
* false negatives
    * what does it mean when a build fails?
        * waht file needs to change to fix the build?
            * a true negative: red build means the code was broken
                * depressingly rare
                * increase confidence in our tests
            * a false negative: red build means we forgot to update a test
                * depressingly common
                * errode confidence in our tests
    * causes of false-negative failures:
        1. slow tests
            * tests were too slow to run before I pushed
        2. redundant tests
            * i changed some code but lots of tests depended on it
    * track whether each build failure is a true or false negative and how long it took you to fix it

#### Summary

Test suites without a plan are problematic - Team should agree on some choices
for their suites:

* discuss the defnintion of a successful test suite:
    1. is the purpose of each test readily apparent
    1. does the test suite promote consistency?
* discuss and agree on the boundaries of "realisim" in our tests and what we are willing to accept
* agree on how we use mocking
    * don't have a mix of mocks and real objects
    * don't use mocks as a way to avoid difficult to setup objects - that is fixing the symptom, not the cause of test pain
    * mock or real all 3rd party APIs?
    * mock or real all collaborator objects?
* don't be haphazard with mocking - have a plan
* decide how many suites we should have and what level of isolation they have
* set a maximum build time and agree we will refactor tests/delete them to keep within it
* only use `rails_helper` when you really need it
* agree not to create an integration test for each feature
    * agree to try and have integration tests that cover a lot of ground - maybe like a real user would

things about beemo test suite
* we use rails_helper everywhere
* we don't have an agreement on mocking/levels of isolation
* we have some controller tests - should they be more integrated?
* our suites are organised by type not by level of isolation

## http://blog.testdouble.com/posts/2014-05-25-breaking-up-with-your-test-suite.html

> Kent beck"
> I get paid for code that works. I test as little as possible to reach a given level of confidence.

* Testing is not an end in itself
* Neither is coding, coding is not an end in itself
    * the "carrying cost" of code is high - if you can solve a clients problem without code you probably should

Questions to ask yourself

* Is TDD right in this context?
* Is Rails right for this job?
* Is ruby?

There are many things tests can *possibly* help with and they break down into two categories

1. understanding - value comes from writing test
2. confidence - value comes from running test

Going deeper into these categories

* types of understandings (the value is in the *writing* and maintaining the test as it informs the design of the system)
    1. validate code is useful to others
    1. grow a maintainable private API
    1. make the public API easy to use
    1. narrowly specify our dependencies (???)
    1. keep the product simple
    1. design a good API for each object
* types of confidences (the value is in *running* tests to tell you whether things are good or not)
    1. verify that production is working
    1. safeguard our use of 3rd party code
    1. know that my code works
    1. ensure behaviour of others services
    1. safely change and refactor

BUT no one test can do all of these things at the same time - single responsibility testing!

=> not all your tests have the same goals!

Some examples

1. Bugfixing tests that are more integrated
1. exploratory tests that were used during design to decide on the API by mocking out all collaborators

@searls thinks that it should be clear what kind of test we are looking at before we open the file

Unclear tests:

* cost money because none of the things below have value but they do take time
* have unclear rules (about mocks etc.)
* constantly rediscivering the "purpose" of the test
* rules are debatable
* structure is ad-hoc

Every test suite should promote at most exactly one type of confidence and one type of understanding

=> it should be obvious from the file path and name what the 1) confidence and 2) understanding is for the test file I am about to open

### Suite 1: safety test suite

All of these mean roughly the same thing:

* Smoke test
* Acceptance tests
* Feature test
* End-to-end test

@searls pulled them out into the pithy SAFE acronym
The SAFE test suite is the suite which provides safety!

* the user = real world user as possible
    * HTTP API endpoint => api client
    * HTML endpoint => some sort of web automation e.g. selenium, capybara etc.
* confidence
    * all the layers of the application work
* understanding
    * how simple is our product?
        * if you can't run all the possible things a user might want to do in the app in under 30 mins the app is *complex*
        * if you can't add a new safety test to the app then it might be *complicated*
* guidelines
    * they shouldn't know about internal APIs
        * ideally bind to user visible stuff not markup or DB changes
    * enforce a time budget on the safety test suite because it will get slow
* warning signs
    * failures due to refactors are false negatives
    * human intuition tends to value overly realistic tests
    * superlinear slowdown
        * more tests => slower build
        * as system gets bigger each individual test will run slower

* safe tests are accurate but also expensive

### Suite 2: consumption test suite

* user:
    * whoever is using our thing where thing can be a service, an API endpoint, a page on the site etc.
* confidence:
    * verify the behaviour we are directly responsible for
* understanding:
    * is our API easy to use
    * if it is hard to write a test it is probably hard to use
* guidelines
    * module boundaries should be meaningful beyond testing
    * fake all external depencencies
    * exercise public APIs not private APIs
    * organise by consumer's desired outcomes not class names
* warning signs:
    * if you are using faked services these should not be slow!
        * if they are slow then you should fix it
    * this suite is you refactoring protection suite
        * if it is not catching refactoring errors you should fix it


Inter-service tests (tests between two services) are tempting but they are

* hard to setup
* slow to run
* they represent redundant test coverage

The root cause is a lack of trust that the other services will continue to work

@searls says to default to trusting the other services (whether they are
managed by your team or another team) on which you depend. When you can't
trust the other service, write contract tests (see below).

## Contract test suite

Write a test that represents your interests in somebody else's repo and have
your contact details on the test so they can tell you if that test breaks. This allows you to discuss the problem before it hits production

Useful for

* if you are using an API in an unusual or unsupported way i.e. you think the
  maintainers might not be testing your use-case properly

* user:
    * you!
* confidence:
    * dependencies will continue to work the way we need them to
* understanding:
    * learn whether that service continues to be a good fit for our needs
    * if it keeps breaking then we might have a problem and we get to make that decision *before* it breaks in production
* guidelines
    * written just like consumption tests
    * commited into the repo they are testing
    * maintainers of that repo can provide some setup and teardown convenience helpers
* warning signs:
    * it is not a replacement for human interaction! Your first reaction to a service not working as you need is to go talk to the people who maintain it.

None of the tests above provide good feedback about the design of our code

@searls thinks TDD is about discovery

* discover of API
* discovery of tiny, boring, consistent units of code that break a big problems into small manageable ones

## Adapter test suite

* Mocking what you don't own leads to useless pain!
* instead wrap 3rd party APIs with tiny objects that you do own
* adapters should be thin!
    * often thin enough not to need tests

* user
    * your application trying to udnerstand its relationship to a 3rd party API
    * exercise the 3rd party API under realistic circumstances
* confidence
    * tests of libraries warn when an upgrade might be unsafe
    * for network services it can alert us to outages and breaking changes
* understanding
    * serve as specs for how we use the 3rd party thing
    * drastically reduces the cost of replacement later
    * easy to replace one gem with another if our usage of the gem is down to one little adapter object
* guidelines
    * only test adapters when you have good cause
    * by default you shouldn't be testing the framework
    * default to not writing adapter tests
* warning signs
    * can be tricky to run in CI because they depend on 3rd party gem/API
        * might make more sense to run from the safety suite
    * are likely to be slow for reasons you can't control

### Discovery test suite

* user
    * the first person to call a new method
    * they are concerned with the inputs that the new method needs and the outputs AND side-effects it will have
        * they are concerned with inputs and outcomes
    * the discovery test suite is code by wishful thinking
* confidence
    * is limited. The discovery test suite does not provide much confidence that things work
        * it can provide confidence that pure first order functions work
* understanding
    * is rich
    * I'm getting small things or "free SRP"
    * separation of roles
* guidelines
    * command and query objects discover dependencies using test doubles
    * @searls really does make a LOT of small objects, each of which falls into one category
        1. command object
        2. query object
        3. logic object
            * logical tests discover implementation by usage
                * @searls recommends NOT using test doubles for logical tests
    * He tries to limit objects to one public method and 3 private methods
    * command and query objects contain very little logic and do use test doubled to discover collaborator interfaces
    * test pain is good in this stage as it indicates problems with your code design
* warning signs
    * yield small and *disposable* units
        * don't be afraid to throw away those small units
    * re-use is overrated - this model of work creates a lot of specialised things that don't lend themselves particularly to re-use
    * extract refactors in this model of work are a smell - instead destroy the old thing and make a new thing
        * it should not be necessary if we are preemptively pushing out small things
    * frameworks will fight you
        * frameworks and TDD try to solve the same problem
            * frameworks provide a structure and types for you to use
            * TDD wants you to discover that set of types for yourself

Only the command and query objects in the discovery test suite should use mock objects!

## Summary

All of the above is just one way to break up into multiple test suites

* safety suite
* contract suite
* consumption suite
* discovery suite
* adapter suite

## What we have by default in rails apps

`spec/features`
    * seems to map to a mix of the safety and consumption suites here
    * -- a mix of slow safety tests and fast consumption tests
    * -- no clear mocking policies (3rd party api, collaborator)
`spec/models` suite
    * seems to map to a mix of discovery + adapter + consumption
    * -- no clear mocking policies (3rd party api, collaborator)

## Criteria for categorising a test suite

* XYZ suite
    * user:
    * confidence:
    * understanding:
    * organised by: class name|customer behaviour
    * 3rd party API policy: all real|all mock
    * collaborator object policy: all real|all mock
    * guidelines:
        add guidelines to help us get this right
    * warning signs:
        add warning signs that we might not be sticking to the policy
    * limits of its realism
        agree on what this suite does not try to address


Examples of test suites from standard rails apps

* models suite
    * user:
        * client code of each model
    * confidence:
    * understanding:
        * how easy is each model to setup
    * guidelines:
    * warning signs:
        * no clear mocking policy
    * organised by: class name
    * 3rd party API policy: sometimes real, sometimes mocks
    * collaborator object policy: sometimes real, sometimes mocks
* features suite
    * user:
    * confidence:
    * understanding:
    * guidelines:
        * organised by:
    * warning signs:
* requests suite
    * user:
    * confidence:
    * understanding:
    * guidelines:
    * warning signs:
* acceptance suite
    * user:
    * confidence:
    * understanding:
    * guidelines:
    * warning signs:
* background jobs suite
    * user:
    * confidence:
    * understanding:
    * guidelines:
    * warning signs:
* rake tasks suite
    * user:
    * confidence:
    * understanding:
    * guidelines:
    * warning signs:

The following suites are usually ignored by rails devs these days

* controllers suite
* routes suite
* helpers suite
* views suite

# http://blog.testdouble.com/posts/2015-09-10-how-i-use-test-doubles.html

* Bottom up TDD requires you to come up with your first design on your own
    * ++ it does help you implement that design
    * -- it kinda requires you to have a good design in your head before you start
        * this is possibly why it works so well for experienced devs
* Top down TDD
    * tries to help you come up with the design as well as implement it

```
write the first top-level test
? should that be a controller or acceptance test?
? is it a safety test or a discovery test? is it quite integrated?
when you do find an object you need a sense of whether it is collaborator or logic object?
```

@searls objects tend to fall into

1. collaborator objects
    * contain very little logic
    * they gather input for the logic objects
    * they pipeline output from the logic objects
    * use test doubles to discover their dependencies
    * in functional lang they are
    * they don't hold any state about the transaction in progress - the value objects do that
2. logic objects
    * tend to be pure functions
    * "logical leaf node"
    * because they are pure functions we don't need test doubles
2. value objects
    * wrap data in the application
    * they are the data that flows through the pipes we created (pipes of collaborator and logic objects)
    * provide methods that expose that data
    * these don't use test doubles
4. wrapper objects
    * it wraps a 3rd party API
    * unit tests not that useful - @searls doesn't write unit tests of these
    * he uses the safety net integration tests to catch issues

When he has to make big changes he favours rewrites i.e. don't try to make an
existing small object od something new, instead created a new small object and
change the plumbing

Pros/cons of "Discovery testing"

* intense design pressure yields small focused units
* ++ every method signature and type is validated before it exists - very little waste
* refactoring becomes an exceptional case
* -- much complex to learn that than red-green-refactor
* -- radical implementation changes favour resrites over refactor
    * tests are tightly coupled to the implementation
* -- collaborator tests value is front-loaded
    * they tend not to be valued by folks who don't practice this workflow

Most test double libs tend to be better at one style of TDD than another.
Older larger libs tesnd to be unopinionated which makes them confusing for new folks

### VerbNoun.verb naming for objects

@searls creats objects in verbNoun form not NounVerb e.g.

GeneratesSeedWorld vs SeedWorldGenerator
BuildsTransaction vs TransactionBuilder

his stated reasons are

1. the verb is what matters
1. over time if the verb stays the same but the noun generalizes you end up with slightly cleaner design

he tends to have the main (only?) public method of each of the VerbNoun objects match the verb

BuildsTransaction.build
GeneratesSeedWorld.generate

## How I think about objects

I think I need to get away from objects being nouns/people - they can also be
processes! VerbNoun naming might be a good way to break that habit

## Things to try in my own process

1. try to think about objects in the collaborator, logic, value categories

2. have a box drawing desgin step where I try to break the problem into an object tree
    * collaborator (input and output) and logical objects go in the tree
    * have a separarate area for value objects (they get passed around)

    reasons I don't do this at the moment:
        * i feel like i own't be able to see what objects I need until I have some code
            * this might not be true - experiment!

