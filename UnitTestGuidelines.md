# Unit Testing Guidelines #
The guidelines in this document are based on recommendations from [Art of Unit Testing](http://osmy.in/1Dpu9Nc "Art of Unit Testing") by Roy Osherove (2013), and the [Testing on the Toilet](http://osmy.in/1AjT0k0 "Testing on the Toilet") blog by Google. 

I have also created a [2-hour course on unit testing at Pluralsight](http://osmy.in/EffectiveCSharpUnitTesting) that covers these topics in depth.

## Definitions ##
It's important to have a common language when talking about testing strategies so that we can 
understand each other's preferences without confusion over what each other think of as a unit test.

### Unit of Work ###
Everything that happens from invoking a public method to it returning the results after it's finished; it’s the work done along the path you see the debugger take through your code.

### Unit Test ###
Code that invokes a unit of work within the confines of a project layer while faking external dependencies and validates an assumption about one specific scenario. 

*Naming convention*

**UnitOfWork\_InitialCondition\_ExpectedResult**

*	Unit of Work - name of method or the description of the unit of work, such as "Login"
*	Initial Condition - the conditions being tested, such as "InvalidUser" or a description of the parameters 
being passed into the unit of work
*	Expected Result - your expected outcome, such as "UserNotFoundMessage"


Use readability as your guide to the name; the test name should read like a sentence with no ands and 
ors in it.

*Examples*

Login\_InvalidUser\_UserNotFoundMessage

UpdateDisplayOrder\_MoveFirst\_MovesToFirstPosition

ValidateWorkPhaseSEctionEstimateDetails\_WithErrors\_ReturnsSuccessIsFalse

To make this very easy, you can install this [unit test snippet](https://gist.github.com/osmyn/906c917653a30864cb52dee02c36c14e) in Visual Studio to stub out your unit tests for you.

### Integration Test ###
While unit tests fake dependencies to test scenarios in the unit of work, an integration test uses real 
dependencies, covers many scenarios, or crosses layers in the test. Examples of integration tests include 
changing data in a database, accessing the file system, working with system time, checking that all 
controllers have a specific attribute, or using the actual service layer from a controller instead of faking 
it.

Integration tests are important, but should be put into their own project so that they can run only on 
check in or manually because they generally will take longer to run and may require special setup.

### Stub ###
A substitute for a dependency in the system that is used only so that the dependency void is filled and 
the test runs without a compile error. A stub is never asserted against - it can never make a test fail. A 
stub can return a test-specified value from its operations or throw an exception.

### Mock ###
A substitute for a dependency in the system that knows whether or not it was called and is asserted 
against - it can make a test fail. It is used to make sure the unit of work actually called the expected 
dependency.

### Fake ###
A generic term that can be used as a verb to describe what a stub or a mock does; they fake the 
behavior of the dependency so that the real dependency does not need to be used.

### Refactoring ###
Restructuring the design of your code should be done frequently - after writing several tests or 
completing a unit of work - and should be possible most times without breaking any of your tests even 
when you refactor some logic into private methods.

### Over Specifying ###
Unit tests that too often are describing what should happen and how through over use of mocks instead 
of testing a scenario returns the expected outcome.

### End Result Types of Unit Tests ###
Unit tests come in the following three varieties:
*	Value-based - check the value returned from a unit of work. 
*	State-based - check for noticeable behavior changes after changing state
*	Interaction - check how a unit of work makes calls to another object


**Value-based**: The easiest test to complete, just make sure the return result is what you expect and 
ignore the rest.

**State-based**: if a method changes the state of a class' public property, you can test that pretty easily. If 
you need to test the state of a method-scope variable it gets a little trickier. You can arrange a 
.DoInstead() on a service call to then control what happens instead of that service call. Using this 
technique, it's possible to get a copy of a parameter into the scope of your unit test like:

       myService.Arrange(x => x.Save).DoInstead((int arg1) => { unitTestScopeVar = arg1;})

**Interaction-based**: interaction based testing uses mocks that know how many times a method has been 
called (.OccursOnce(), .MustBeCalled()) or the order it is called in. This is the last thing you should turn 
to for testing and avoid it if possible because it makes your code very hard to refactor since changing 
how anything works breaks tests just by not doing something in the same number of steps instead of 
breaking tests because the end result is wrong.

### Mocking ###
The JustMock framework calls all fakes mocks, even if they are stubs.

*	Mock.Create<IService>() is a stub of that service. 
*	myService.Arrange(x => x.Save).Returns("Success") (or the similar Mock.Arrange) is a stub of 
x.Save
*	myService.Arrange(x => x.Save).DoInstead((int arg1) => { toTest = arg1;}) is a way to use a stub 
to check the state of an argument passed into the stubbed service
*	myService.Save().MustBeCalled() is a mock because it now knows it was called and can break 
the test if it is not called

Each test should only test one scenario, so avoid more than one mock per test. Any time a mock is being 
used you are doing an interaction test, and always choose to do interaction testing as the last option 
when the interacting between objects are the end result (such as testing a service method that just 
passes the call to the repository).

Tests will always be more maintainable when you don't assert that an object was called. If more than 5% 
of your tests have mock objects, then you might be over specifying things. If tests specify too many 
expectations, then they become very fragile and break even when the overall functionality isn't broken; 
the extra specifications can make the test fail for the wrong reasons.

*When to mock*

*	Testing an event
*	Testing a scenario where there is not a state change or value returned, such as a pass-through 
method on the service layer, or a catch block that just logs or emails the error using an external 
dependency

### Tips ###
*	Specify only one of the three end result types
*	Use non-strict fakes when you can so that tests will break less often for unexpected method 
calls
*	No private method exists without a reason; somewhere a public API calls into the private 
method and your test should cover the scenarios possible, which should also lead to 100% 
coverage of the private method without ever explicitly testing that method. If you test only the 
private method and it works, it does not mean the public API uses it correctly.

### Google's "Testing on the Toilet" Blog ###
Google's testing team started posting articles on the doors of bathroom stalls as a way to get people 
thinking about good testing habits, hence the name of the blog. Here are some example posts:

*	[Don't overuse mocks](http://osmy.in/1F79VtI)
*	[Testing state vs. interactions](http://osmy.in/1vrIAd9)
*	[Test behaviors, not methods](http://osmy.in/1F7a5kT)
*	[Effective Testing](http://osmy.in/1zSEr7j)
*	[Measuring Coverage](http://osmy.in/1FIkBMM)

--------------
# Unit Testing Checklist #
- [ ] The test does not cross project layers or use real dependencies 
(file, database, system time)
- [ ] The test does not invoke private methods
- [ ] The test name is easy to read 
(UnitOfWork\_InitialCondition\_ExpectedResult)
- [ ] The test does not check interactions where value or state change 
checks could be used for full coverage
- [ ] The test only checks one of: value result, state change, or 
interaction
- [ ] The test does not use strict fakes where it would pass with loose 
fakes
- [ ] The test does not fake more than it has to
- [ ] The test tests for behavior over one-test-to-one-method design
- [ ] The test does not use a mock where it is not testing interaction
- [ ] The test does not have more than one mock (used for interaction)
- [ ] The test does not have flow control (switch/if/while)
- [ ] The test does not test a third party library

## Overall ##

- [ ] The tests cover the common scenarios and some corner cases if practical
- [ ] The tests don't repeat code that could be refactored into builder or 
factory methods


