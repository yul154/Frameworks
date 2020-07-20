

# TDD

Traditional approach:
In the early, developers would often write their functional code and then figure out how to write the unit test to validate the assumptions made in how the code should function. 

TDD benefit
* balance between automated testing and functiona coding durin development
* Consistent repeatability of test execution
* up-front design with testing in mind

Red/Green/Refactor
> Common pattern in TDD
1. Design a test: What is the expected output or result?
2. run the test(red): it may reasonably be expected that the test would fail were it to be run
3. write  the code and test again: The missing feature should be implemented so that the test will now pass (green).
4. refactor the code: Now that the functionality is in place, it can be optimized for improved readability and maintainability

Terminology
* xUnit Testing: generic reference to automated testing using a unit test framework
* Class-under-test: one or more test class correpsonding to single functional class
* Method-under-test:a single JUNIT test will be executing to verify functionalit and validate assumptiongs
* Test Fixture: a fixed state in code which is tested used as input for a test.


# JUNIT
* Junit class is standrad java class
## Constructs
* `@Test`:

Verfication methods
* `org.junit.Assert.*`
* HamcrestMatcherAssert: declaring assertions that som consider more readable

@Before/@BeforeClass
* Method level
* step up and teardown steps



# Mockito

Mock
* a way to test the functionality of a class in isolation
* Mocking does not require a database connection or properties file read or file server read to test a functionality.

Mockito
* Mock objects are nothing but proxy for actual implementations.

Pipeline

1. Creating mock objects with Mockito
  * Using the static mock() method.
  * Using the @Mock annotation.





# Summary

1. Automation Testing tools you used?
* Mockito
* Junit

2. How do you do pipeline testing? Do you deploy automatically or manually?

3. Did you write test cases?

6.Exception handling

6. ClassCastExcetpion , what kinds of exception? How handle it ? scenario ? how to catch an exception?

* How do you do pipeline testing? 
Red/Green/Refactor
Common pattern in TDD
1. Design a test: What is the expected output or result?
2. run the test(red): it may reasonably be expected that the test would fail were it to be run
3. write the code and test again: The missing feature should be implemented so that the test will now pass (green).
4. refactor the code: Now that the functionality is in place, it can be optimized for improved readability and maintainability

