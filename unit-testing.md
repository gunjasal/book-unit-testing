# Chapter 1 The goal of unit testing

## 1.2 The goal of unit testing
* The goal is to enable sustainable growth of the software project. 
* (side effect) unit testing practices lead to a better design.

### Test Costs
* Refactoring the test when you refactor the underlying code
* Running the test on each code change
* Dealing with false alarms raised by the test
* Spending time reading the test when you’re trying to understand how the underlying code behaves


## 1.3 Code coverage metric
* Code coverage (test coverage) = Lines of code executed / Total number of lines
* Branch coverage = Branches traversed / Total number of branches

## 1.4 What makes a successful test suite?
* It’s integrated into the development cycle
* It targets only the most important parts of your code base
  * In most applications, the most important part is the part that contains business logic—the domain model.
  * All other parts
    * Infrastructure code
    * External services and dependencies, such as the database and third-party systems 
    * Code that glues everything together
  * Some of your tests, such as integration tests, can go beyond the domain model and verify how the system works as a whole, including the noncritical parts of the code base
* It provides maximum value with minimum maintenance costs
  * Recognizing a valuable test (and, by extension, a test of low value)
    * To recognize a test of high value, you need a frame of reference
  * Writing a valuable test
    * On the other hand, writing a valuable test requires you to also know code design techniques.

### The only way to achieve the goal of unit testing (that is, enabling sustainable project growth) is to
* Learn how to differentiate between a good and a bad test.
* Be able to refactor a test to make it more valuable.


# Chapter 2 What is a unit test?

## 2.1 The definition of “unit test”
* Verifies a small piece of code (also known as a unit)
* Does it quickly,
* And does it in an isolated manner.
  * Classical(Detroit)
  * London (mockist) : Test Double

### 2.1.1 The isolation issue: The London take
* During the arrange phase, the tests put together two kinds of objects: the system under test (SUT) and one collaborator. In this case, 
  * Customer is the SUT
  * Store is the collaborator. 
* Listing 2.1 Tests written using the classical style of unit testing
    ```
    public void Purchase_succeeds_when_enough_inventory() {
        // Arrange
        var store = new Store();
        store.AddInventory(Product.Shampoo, 10);
        var customer = new Customer();

        // Act
        bool success = customer.Purchase(store, Product.Shampoo, 5);

        // Assert
        Assert.True(success);
        Assert.Equal(5, store.GetInventory(Product.Shampoo));
    }

    public void Purchase_fails_when_not_enough_inventory() {
        // Arrange
        var store = new Store();
        store.AddInventory(Product.Shampoo, 10);
        var customer = new Customer();
    
        // Act
        bool success = customer.Purchase(store, Product.Shampoo, 15);
    
        // Assert
        Assert.False(success);
        Assert.Equal(10, store.GetInventory(Product.Shampoo));
    }

    public enum Product {
        Shampoo,
        Book
    }
    ```

* Listing 2.2 Tests written using the London style of unit testing
    ```
    public void Purchase_succeeds_when_enough_inventory() {
        // Arrange
        var storeMock = new Mock<IStore>();
        storeMock
            .Setup(x => x.HasEnoughInventory(Product.Shampoo, 5))
            .Returns(true);
        var customer = new Customer();

        // Act
        bool success = customer.Purchase(storeMock.Object, Product.Shampoo, 5);

        // Assert
        Assert.True(success);
        storeMock.Verify(
            x => x.RemoveInventory(Product.Shampoo, 5),
            Times.Once
        );
    }

    public void Purchase_fails_when_not_enough_inventory() {
        // Arrange
        var storeMock = new Mock<IStore>();
        storeMock
            .Setup(x => x.HasEnoughInventory(Product.Shampoo, 5))
            .Returns(false);
        var customer = new Customer();

        // Act
        bool success = customer.Purchase(storeMock.Object, Product.Shampoo, 5);

        // Assert
        Assert.False(success);
        storeMock.Verify(
            x => x.RemoveInventory(Product.Shampoo, 5),
            Times.Never
        );
    }
    ```
## 2.2 The classical and London schools of unit testing
* the root of the differences between the London and classical schools is the isolation attribute. 
  * The London school views it as isolation of the system under test from its collaborators, whereas the classical school views it as isolation of unit tests themselves from each other.

### 2.2.1 How the classical and London schools handle dependencies
* It’s fine not to substitute objects that don’t ever change— immutable objects.
  * Of the two dependencies of Customer, only Store contains an internal state that can change over time.

## 2.3 Contrasting the classical and London schools of unit testing
* The London school’s approach provides the following benefits:
  * Better granularity: The tests are fine-grained and check only one class at a time.
  * It’s easier to unit test a larger graph of interconnected classes.
  * If a test fails, you know for sure which functionality has failed.

### 2.3.1 Unit testing one class at a time
* The point about better granularity relates to the discussion about what constitutes a unit in unit testing. The London school considers a class as such a unit. Coming from an object-oriented programming background, developers usually regard classes as the atomic building blocks that lie at the foundation of every code base. This naturally leads to treating classes as the atomic units to be verified in tests, too. This tendency is understandable but misleading.
  * Tests shouldn’t verify units of code. Rather, they should verify units of behavior: something that is meaningful for the problem domain and, ideally, something that a business person can recognize as useful.

### 2.3.2 Unit testing a large graph of interconnected classes
* Instead of finding ways to test a large, complicated graph of interconnected classes, you should focus on not having such a graph of classes in the first place. More often than not, a large class graph is a result of a code design problem.
* If you see that to unit test a class, you need to extend the test’s arrange phase beyond all reasonable limits, it’s a certain sign of trouble. The use of mocks only hides this problem


## 2.4 Integration tests in the two schools
* The London school considers any test that uses a real collaborator object an integration test. 
  * Most of the tests written in the classical style would be deemed integration tests by the London school proponents. 
### 2.4.1 End-to-end tests are a subset of integration tests
* In short, an integration test is a test that verifies that your code works in integration with shared dependencies, out-of-process dependencies, or code developed by other teams in the organization.
* Since end-to-end tests are the most expensive in terms of maintenance, it’s better to run them late in the build process, after all the unit and integration tests have passed. You may possibly even run them only on the build server, not on individual developers’ machines.


# Chapter 3 The anatomy of a unit test
## 3.1 How to structure a unit test
### 3.1.1. Using the AAA pattern
* The AAA pattern advocates for splitting each test into three parts: arrange, act, and assert.
  ```
  public class Calculator
  {
      public double Sum(double first, double second)
      {
          return first + second;
      }
  }

  # Listing 3.1 A test covering the Sum method in calculator
  public void Sum_of_two_numbers() {
      // Arrange
      double first = 10;
      double second = 20;
      var calculator = new Calculator();

      // Act
      double result = calculator.Sum(first, second);

      // Assert
      Assert.Equal(30, result);
  }
  ```

  * In the arrange section, you bring the system under test (SUT) and its dependen- cies to a desired state.
  * In the act section, you call methods on the SUT, pass the prepared dependen- cies, and capture the output value (if any).
  * In the assert section, you verify the outcome. The outcome may be represented by the return value, the final state of the SUT and its collaborators, or the meth- ods the SUT called on those collaborators

* Given-When-Then pattern
  * You might have heard of the Given-When-Then pattern, which is similar to AAA. This pattern also advocates for breaking the test down into three parts:
    * Given—Corresponds to the arrange section
    * When—Corresponds to the act section
    * Then—Corresponds to the assert section

* The natural inclination is to start writing a test with the arrange section. After all, it comes before the other two. This approach works well in the vast majority of cases, but starting with the assert section is a viable option too. When you practice Test-Driven Development (TDD)—that is, when you create a failing test before developing a feature—you don’t know enough about the feature’s behavior yet. So, it becomes advantageous to first outline what you expect from the behavior and then figure out how to develop the system to meet this expectation.
* If you write the production code before the test, by the time you move on to the test, you already know what to expect from the behavior, so starting with the arrange section is a better option.

### 3.1.2 Avoid multiple arrange, act, and assert sections
* `arrange -> act -> assert -> act more -> assert more`
* When you see multiple act sections separated by assert and, possibly, arrange sections, 
  * it means the test verifies multiple units of behavior. And, as we discussed in chapter 2, such a test is no longer a unit test but rather is an integration test.
  * It’s sometimes fine to have multiple act sections in integration tests.
* As you may remember from the previous chapter, integration tests can be slow. 
  * One way to speed them up is to group several integration tests together into a single test with multiple acts and assertions. 
  * It’s especially helpful when system states naturally flow from one another
  * this optimization technique is only applicable to integration tests—and not all of them

### 3.1.3 Avoid if statements in tests
* a unit test with an if statement is also an anti-pattern
  * A test— whether a unit test or an integration test—should be a simple sequence of steps with no branching.
* An if statement indicates that the test verifies too many things at once. Such a test, therefore, should be split into several tests.

### 3.1.4 How large should each section be?
* THE ARRANGE SECTION IS THE LARGEST
  * It can be as large as the `act` and `assert` sections combined. But if it becomes significantly larger than that, it’s better to extract the arrangements either into private methods within the same test class or to a separate factory class. Two popular patterns can help you reuse the code in the arrange sections: 
    * `Object Mother and Test Data Builder.`
* WATCH OUT FOR ACT SECTIONS THAT ARE LARGER THAN A SINGLE LINE
  * If the `act` consists of two or more lines, it could indicate a problem with the SUT’s public API.
    ```
    # Listing 3.3 A two-line act section
    public void Purchase_succeeds_when_enough_inventory() {
      // Arrange
      var store = new Store();
      store.AddInventory(Product.Shampoo, 10);
      var customer = new Customer();

      // Act
      bool success = customer.Purchase(store, Product.Shampoo, 5);
      store.RemoveInventory(success, Product.Shampoo, 5);

      // Assert
      Assert.True(success);
      Assert.Equal(5, store.GetInventory(Product.Shampoo));
    }
    ```
  * Note that this is not an issue with the test itself. The test still verifies the same unit of behavior: the process of making a purchase. The issue lies in the API sur- face of the Customer class. It shouldn’t require the client to make an additional method call.
    * Such an inconsistency is called an invariant violation. 
    * The act of protecting your code against potential inconsistencies is called encapsulation.
    * When an inconsistency penetrates into the database, it becomes a big problem: now it’s impossible to reset the state of your application by simply restarting it. You’ll have to deal with the cor- rupted data in the database and, potentially, contact customers and handle the situation on a case-by-case basis.

### 3.1.5 How many assertions should the assert section hold?
* You may have heard about the guideline of having one assertion per test. 
  * It takes root in the premise discussed in the previous chapter: the premise of targeting the smallest piece of code possible.
  * As you already know, this premise is incorrect. 
    * A unit in unit testing is a unit of behavior, not a unit of code. 
    * A single unit of behavior can exhibit multiple outcomes, and it’s fine to evaluate them all in one test.
  * Having that said, you need to watch out for assertion sections that grow too large: 
    * it could be a sign of a missing abstraction in the production code. 
    * For example, instead of asserting all properties inside an object returned by the SUT, it may be better to define proper equality members in the object’s class.
    * You can then compare the object to an expected value using a single assertion.

### 3.1.6 What about the teardown phase?
* Note that most unit tests don’t need teardown. 
* Unit tests don’t talk to out-of-process dependencies and thus don’t leave side effects that need to be disposed of. That’s a realm of integration testing.

### 3.1.7 Differentiating the system under test
* The SUT plays a significant role in tests. It provides an entry point for the behavior you want to invoke in the application.
* Thus it’s important to differentiate the SUT from its dependencies, especially when there are quite a few of them, so that you don’t need to spend too much time figuring out who is who in the test. To do that, always name the SUT in tests sut.
  ```
  public void Sum_of_two_numbers() {
    // Arrange
    double first = 10;
    double second = 20;
    var sut = new Calculator();

    // Act
    double result = sut.Sum(first, second);

    // Assert
    Assert.Equal(30, result);
  }
  ```

### 3.1.8 Dropping the arrange, act, and assert comments from tests
* Just as it’s important to set the SUT apart from its dependencies, it’s also important to differentiate the three sections from each other, so that you don’t spend too much time figuring out what section a particular line in the test belongs to.
  * One way to do that is to put // Arrange, // Act, and // Assert comments before the beginning of each section. 
  * Another way is to separate the sections with empty lines, as shown next.
    * It doesn’t work as well in large tests, though, where you may want to put additional empty lines inside the arrange section to differentiate between configuration stages. 
    ```
    public void Sum_of_two_numbers() {
      double first = 10;
      double second = 20;
      var sut = new Calculator();

      double result = sut.Sum(first, second);

      Assert.Equal(30, result);
    }
    ```

## 3.2 Exploring the xUnit testing framework
## 3.3 Reusing test fixtures between tests
* Reusing code between arrange sections is a good way to shorten and simplify your tests
  * The first—incorrect—way to reuse test fixtures is to initialize them in the test’s constructor
  ```
  // Listing 3.7 Extracting the initialization code into the test constructor
  public class CustomerTests {
      private readonly Store _store;
      private readonly Customer _sut;

      [BeforeEach]
      public CustomerTests() {
          _store = new Store();
          _store.AddInventory(Product.Shampoo, 10);
          _sut = new Customer();
      }

      [Fact]
      public void Purchase_succeeds_when_enough_inventory() {
          bool success = _sut.Purchase(_store, Product.Shampoo, 5);

          Assert.True(success);
          Assert.Equal(5, _store.GetInventory(Product.Shampoo));
      }

      [Fact]
      public void Purchase_fails_when_not_enough_inventory() {
          bool success = _sut.Purchase(_store, Product.Shampoo, 15);

          Assert.False(success);
          Assert.Equal(10, _store.GetInventory(Product.Shampoo));
      }
  }
  ```
  * The two tests in listing 3.7 have common configuration logic. 
    * In fact, their arrange sections are the same and thus can be fully extracted into CustomerTests’s constructor
    * With this approach, you can significantly reduce the amount of test code
  * But this technique has two significant drawbacks:
    * It introduces high coupling between tests. 
    * It diminishes test readability.

### 3.3.1 High coupling between tests is an anti-pattern
### 3.3.2 The use of constructors in tests diminishes test readability
### 3.3.3 A better way to reuse test fixtures
* the beneficial one—is to introduce private factory methods in the test class
* By extracting the common initialization code into private factory methods, 
  * you can also shorten the test code, 
  * but at the same time keep the full context of what’s going on in the tests. 
  * Moreover, the private methods don’t couple tests to each other as long as you make them generic enough.
* This is both highly readable and reusable. 
  * It’s readable because you don’t need to examine the internals of the factory method to understand the attributes of the created store. 
  * It’s reusable because you can use this method in other tests, too.
  ```
  // Listing 3.7 Extracting the initialization code into the test constructor
  public class CustomerTests {
    [Fact]
    public void Purchase_succeeds_when_enough_inventory() {
      Store store = CreateStoreWithInventory(Product.Shampoo, 10);
      Customer sut = CreateCustomer();

      bool success = _sut.Purchase(_store, Product.Shampoo, 5);

      Assert.True(success);
      Assert.Equal(5, _store.GetInventory(Product.Shampoo));
    }

    [Fact]
    public void Purchase_fails_when_not_enough_inventory() {
      Store store = CreateStoreWithInventory(Product.Shampoo, 10);
      Customer sut = CreateCustomer();
      
      bool success = _sut.Purchase(_store, Product.Shampoo, 15);
      
      Assert.False(success);
      Assert.Equal(10, _store.GetInventory(Product.Shampoo));
    }

    private Store CreateStoreWithInventory(
      Product product, int quantity) {
      Store store = new Store();
      store.AddInventory(product, quantity);
      return store;
    }
  
    private static Customer CreateCustomer() {
        return new Customer();
    }
  }
  ```
## 3.4 Naming a unit test
* One of the most prominent, and probably least helpful, is the following convention:
  * `[MethodUnderTest]_[Scenario]_[ExpectedResult]`
    * MethodUnderTest is the name of the method you are testing.
    * Scenario is the condition under which you test the method.
    * ExpectedResult is what you expect the method under test to do in the current scenario.
  * It’s unhelpful specifically 
    * because it encourages you to focus on implementation details instead of the behavior.
    * Don’t include the name of the SUT’s method in the test’s name.
* Simple phrases in plain English do a much better job
  * With simple phrases, you can describe the system behavior in a way that’s meaningful to a customer or a domain expert.

### 3.4.1 Unit test naming guidelines
* Don’t follow a rigid naming policy.
* Name the test as if you were describing the scenario to a non-programmer who is familiar with the problem domain.
* Separate words with underscores.
### 3.4.2 Example: Renaming a test toward the guidelines
```
// Listing 3.10 A test named using the rigid naming policy
public void IsDeliveryValid_InvalidDate_ReturnsFalse() {
    DeliveryService sut = new DeliveryService();
    DateTime pastDate = DateTime.Now.AddDays(-1);
    Delivery delivery = new Delivery {
        Date = pastDate
    };

    bool isValid = sut.IsDeliveryValid(delivery);
 
    Assert.False(isValid);
}
```
* The following would be a good first try:
  * `public void Delivery_with_invalid_date_should_be_considered_invalid()`
    * The name now makes sense to a non-programmer, which means programmers will have an easier time understanding it, too.
    * The name of the SUT’s method—IsDeliveryValid—is no longer part of the test’s name.
  * `public void Delivery_with_past_date_should_be_invalid()`
    * The wording should be is another common anti-pattern. 
    * I mentioned that a test is a single, atomic fact about a unit of behavior. There’s no place for a wish or a desire when stating a fact.
  * `public void Delivery_with_past_date_is_invalid()`
    * Name the test accordingly—replace `should be` with `is`
  * `public void Delivery_with_a_past_date_is_invalid()`
     * And finally, there’s no need to avoid basic English grammar.

## 3.5 Refactoring to parameterized tests
* most unit testing frameworks provide functionality that allows you to group similar tests using parameterized tests
* The existing test is called `Delivery_with_a_past_date_is_invalid`. We could add three more:
  * public void Delivery_for_today_is_invalid()
  * public void Delivery_for_tomorrow_is_invalid()
  * public void The_soonest_delivery_date_is_two_days_from_now()
* Using parameterized tests, you can significantly reduce the amount of test code, but this benefit comes at a cost. 
  ```
  // Listing 3.11 A test that encompasses several facts
  public class DeliveryServiceTests {
      [InlineData(-1, false)]
      [InlineData(0, false)]
      [InlineData(1, false)]
      [InlineData(2, true)]
      [Theory]
      public void Can_detect_an_invalid_delivery_date(int daysFromNow, bool expected) {
          DeliveryService sut = new DeliveryService();
          DateTime deliveryDate = DateTime.Now
              .AddDays(daysFromNow);
          Delivery delivery = new Delivery {
              Date = deliveryDate
          };

          bool isValid = sut.IsDeliveryValid(delivery);
          
          Assert.Equal(expected, isValid);
      }
  }
  ```
  * It’s now hard to figure out what facts the test method represents. And the more parameters there are, the harder it becomes. 
  * As a compromise, you can extract the positive test case into its own test and benefit from the descriptive naming where it matters the most
    ```
    // Listing 3.12 Two tests verifying the positive and negative scenarios
    public class DeliveryServiceTests
    {
        [InlineData(-1)]
        [InlineData(0)]
        [InlineData(1)]
        [Theory]
        public void Detects_an_invalid_delivery_date(int daysFromNow)
        { /* ... */ }
        
        [Fact]
        public void The_soonest_delivery_date_is_two_days_from_now()
        { /* ... */ }
    }
    ```
### 3.5.1 Generating data for parameterized tests

## 3.6 Using an assertion library to further improve test readability
* One more thing you can do to improve test readability is to use an assertion library
  ```
  [Fact]
  public void Sum_of_two_numbers()
  {
      var sut = new Calculator();

      double result = sut.Sum(10, 20);

      Assert.Equal(30, result);
  }

  // Now compare it to the following, which uses a fluent assertion:
  [Fact]
  public void Sum_of_two_numbers()
  {
      var sut = new Calculator();

      double result = sut.Sum(10, 20);

      result.Should().Be(30);
  }
  ```
* NOTE: The paradigm of object-oriented programming (OOP) has become a success partly because of this readability benefit. With OOP, you, too, can structure the code in a way that reads like a story.

# Chapter 4 The four pillars of a good unit test
## 4.1 Diving into the four pillars of a good unit test
* Protection against regressions
* Resistance to refactoring
* Fast feedback
* Maintainability

### 4.1.1 The first pillar: Protection against regressions
* A regression is when a feature stops working as intended after a certain event (usually, a code modification). 
  * The terms regression and software bug are synonyms and can be used interchangeably.
* The worst part is that the more features you develop, the more chances there are that you’ll break one of those features with a new release. 
  * An unfortunate fact of programming life is that code is not an asset, it’s a liability.
  * The larger the code base, the more exposure it has to potential bugs.
* To evaluate how well a test scores on the metric of protecting against regressions, you need to take into account the following:
  * The amount of code that is executed during the test
    * Generally, the larger the amount of code that gets executed, the higher the chance that the test will reveal a regression
    * While it helps to know that this code runs without throwing exceptions, you also need to validate the out- come it produces.
  * The complexity of that code
    * Code that represents complex business logic is more important than boilerplate code—bugs in business-critical functionality are the most damaging.
  * The code’s domain significance
    * On the other hand, it’s rarely worthwhile to test trivial code. Such code is short and doesn’t contain a substantial amount of business logic.
    * An example of trivial code is a single-line property like this:
      ```
      public class User
      {
          public string Name { get; set; }
      }
      ```
    * Furthermore, in addition to your code, the code you didn’t write also counts: for example, libraries, frameworks, and any external systems used in the project.
      * For the best protection, the test must include those libraries, frameworks, and external sys- tems in the testing scope, in order to check that the assumptions your software makes about these dependencies are correct.
* To maximize the metric of protection against regressions, the test needs to aim at exercising as much code as possible.

### 4.1.2 The second pillar: Resistance to refactoring
* The second attribute of a good unit test is resistance to refactoring—the degree to which a test can sustain a refactoring of the underlying application code without turning red (failing).
* To evaluate how well a test scores on the metric of resisting to refactoring, you need to look at how many `false positives` the test generates. The fewer, the better.
* As you may recall from chapter 1, the goal of unit testing is to enable sustainable project growth. 
  * The mechanism by which the tests enable sustainable growth is that they allow you to add new features and conduct regular refactorings without introducing regressions.
  * There are two specific benefits here:
    * Tests provide an early warning when you break existing functionality.
    * You become confident that your code changes won’t lead to regressions.
  * False positives interfere with both of these benefits:
    * If tests fail with no good reason, they dilute your ability and willingness to react to problems in code.
    * On the other hand, when false positives are frequent, you slowly lose trust in the test suite.
    * This lack of trust leads to fewer refactorings, because you try to reduce code changes to a minimum in order to avoid regressions.

### 4.1.3 What causes false positives?
* The number of false positives a test produces is directly related to the way the test is structured. 
* The more the test is coupled to the implementation details of the system under test (SUT), the more false alarms it generates.
  * You need to make sure the test verifies the end result the SUT delivers: its observable behavior, not the steps it takes to do that.
  * Tests should approach SUT verification from the end user’s point of view and check only the outcome meaningful to that end user.
* The best way to structure a test is to make it tell a story about the problem domain.
  * Should such a test fail, that failure would mean there’s a disconnect between the story and the actual application behavior.

### 4.1.4 Aim at the end result instead of implementation details
* This test treats MessageRenderer as a black box and is only interested in its observable behavior. (!what about out-of-process dependencies?)
  * As a result, the test is much more resistant to refactoring—it doesn’t care what changes you make to the SUT as long as the HTML output remains the same

## 4.2 The intrinsic connection between the first two attributes
* As I mentioned earlier, there’s an intrinsic connection between the first two pillars of a good unit test—`protection against regressions` and `resistance to refactoring`

### 4.2.1 Maximizing test accuracy
 * functionality broken + test passes (false negative)
   * protection against regressions needed
   * How good the test is at indicating the presence of bugs (lack of false negatives, the sphere of protection against regressions)
 * functionality correct + test fails (false positive)
   * resistance to refactoring needed
   * How good the test is at indicating the absence of bugs (lack of false positives, the sphere of resistance to refactoring)

### 4.2.2 The importance of false positives and false negatives: The dynamics

## 4.3 The third and fourth pillars: Fast feedback and maintainability
* Fast feedback
  * The faster the tests, the more of them you can have in the suite and the more often you can run them.
  * That’s because slow tests discourage you from running them often, and therefore lead to wasting more time moving in a wrong direction.
*  Maintainability
  * How hard it is to understand the test
    * This component is related to the size of the test. The fewer lines of code in the test, the more readable the test is.
  * How hard it is to run the test
    * If the test works with out-of-process dependencies, you have to spend time keeping those dependencies operational

## 4.4 In search of an ideal test
* Here are the four attributes of a good unit test once again:
  * Protection against regressions
  * Resistance to refactoring
  * Fast feedback
  * Maintainability
* These four attributes, when multiplied together, determine the value of a test.
  * `Value estimate = [0..1] * [0..1] * [0..1] * [0..1]`
* Remember, all code, including test code, is a liability.
  * Set a fairly high threshold for the minimum required value, and only allow tests in the suite if they meet this threshold.

### 4.4.1 Is it possible to create an ideal test?
* Unfortunately, it’s impossible to create such an ideal test. 
  * The reason is that the first three attributes—`protection against regressions, resistance to refactoring, and fast feedback`— are mutually exclusive. 
  * Moreover, because of the multiplication principle (see the calculation of the value estimate in the previous section), it’s even trickier to keep the balance.

### 4.4.2 Extreme case #1: End-to-end tests
* As you may remember from chapter 2, end-to-end tests look at the system from the end user’s perspective.
  * Since end-to-end tests exercise a lot of code, they provide the best protection against regressions.
  * In fact, of all types of tests, end-to-end tests exercise the most code—both your code and the code you didn’t write but use in the project, 
    * such as external libraries, frameworks, and third-party applications.
  * End-to-end tests are also immune to false positives and thus have a good resistance to refactoring. 
    * A refactoring, if done correctly, doesn’t change the system’s observable behavior and therefore doesn’t affect the end-to-end tests.
  * However, despite these benefits, end-to-end tests have a major drawback: they are slow. 
    * Any system that relies solely on such tests would have a hard time getting rapid feedback.
  * + branch condition testing

### 4.4.3 Extreme case #2: Trivial tests
* Unlike end-to-end tests, trivial tests do provide fast feedback—they run very quickly.
  * They also have a fairly low chance of producing a false positive, so they have good resistance to refactoring. 
* Trivial tests taken to an extreme result in tautology tests.
  * They don’t test anything because they are set up in such a way that they always pass or contain semantically meaningless assertions.

### 4.4.4 Extreme case #3: Brittle tests
* Similarly, it’s pretty easy to write a test that runs fast and has a good chance of catching a regression but does so with a lot of false positives.
  * it can’t withstand a refactoring and will turn red regardless of whether the underlying functionality is broken.
* The test is focusing on hows instead of whats and thus ingrains the SUT’s implementation details, preventing any further refactoring.

### 4.4.5 In search of an ideal test: The results
* Unfortunately, it’s impossible to create an ideal test that has a perfect score in all three attributes
  * Therefore, you have to make trade-offs.
* The fourth attribute, maintainability, is not correlated to the first three, with the exception of end-to-end tests.
  * End-to-end tests are normally larger in size because of the necessity to set up all the dependencies such tests reach out to. 
* In reality, though, resistance to refactoring is non-negotiable. 
  * You should aim at gaining as much of it as you can, provided that your tests remain reasonably quick and you don’t resort to the exclusive use of end-to-end tests.
  * The reason resistance to refactoring is non-negotiable is that whether a test possesses this attribute is mostly a binary choice
      * the test either has resistance to refactoring or it doesn’t. There are almost no intermediate stages in between.
* Figure 4.10 
  * The best tests exhibit maximum `maintainability` and `resistance to refactoring`; always try to max out these two attributes. 
  * The trade-off comes down to the choice between `protection against regressions` and `fast feedback`.

## 4.5 Exploring well-known test automation concepts
### 4.5.1 Breaking down the Test Pyramid
* The Test Pyramid is a concept that advocates for a certain ratio of different types of tests in the test suite (figure 4.11):
  * Unit tests
  * Integration tests
  * End-to-end tests
* Tests in higher pyramid layers favor protection against regressions, while lower layers emphasize execution speed.
* Notice that neither layer gives up resistance to refactoring. 
  * Naturally, end-to-end and inte- gration tests score higher on this metric than unit tests, 
  * but only as a side effect of being more detached from the production code.
* The exact mix between types of tests will be different for each team and project. 
  * But in general, it should retain the pyramid shape: 
    * end-to-end tests should be the minority; 
    * unit tests, the majority; 
    * and integration tests somewhere in the middle.
  * The reason end-to-end tests are the minority is, again, the multiplication rule described in section 4.4. 
    * End-to-end tests score extremely low on the metric of fast feed- back. 
    * They also lack maintainability: they tend to be larger in size and require additional effort to maintain the involved out-of-process dependencies.
    * Thus, end-to-end tests only make sense when applied to the most critical functionality—features in which you don’t ever want to see any bugs
  * There are exceptions to the Test Pyramid. 
    * For example, if all your application does is basic create, read, update, and delete (CRUD) operations with very few business rules or any other complexity, your test “pyramid” will most likely look like a rectangle with an equal number of unit and integration tests and no end-to-end tests.
      * Unit tests are less useful in a setting without algorithmic or business complexity— they quickly descend into trivial tests. 
      * At the same time, integration tests retain their value—it’s still important to verify how code, however simple it is, works in integration with other subsystems, such as the database.

### 4.5.2 Choosing between black-box and white-box testing
* The other well-known test automation concept is black-box versus white-box testing.
  * Black-box testing is a method of software testing that examines the functionality of a system without knowing its internal structure.
    * Such testing is normally built around specifications and requirements: what the application is supposed to do, rather than how it does it.
  * White-box testing is the opposite of that. It’s a method of testing that verifies the application’s inner workings.
    * The tests are derived from the source code, not requirements or specifications.
* There are pros and cons to both of these methods. 
  * White-box testing tends to be more thorough. 
    *By analyzing the source code, you can uncover a lot of errors that you may miss when relying solely on external specifications. 
    * On the other hand, tests resulting from white-box testing are often brittle, as they tend to tightly couple to the specific implementation of the code under test.
    * Such tests produce many false positives and thus fall short on the metric of resistance to refactoring. 
  * Black-box testing provides the oppo- site set of pros and cons (table 4.1).
* As you may remember from section 4.4.5, you can’t compromise on resistance to refactoring: a test either possesses resistance to refactoring or it doesn’t. 
  * Therefore, choose blackbox testing over white-box testing by default. 
  * Make all tests—be they unit, integration, or end-to-end—view the system as a black box and verify behavior meaningful to the problem domain.
  * If you can’t trace a test back to a business requirement, it’s an indi- cation of the test’s brittleness. 
    * Either restructure or delete this test; don’t let it into the suite as-is.
* Note that even though black-box testing is preferable when writing tests, you can still use the white-box method when analyzing the tests. 
  * Use code coverage tools to see which code branches are not exercised, but then turn around and test them as if you know nothing about the code’s internal structure. 
  * Such a combination of the white-box and black-box meth- ods works best.

# Chapter 5 Mocks and test fragility
* The use of mocks in tests is a controversial subject. Some people argue that mocks are a great tool and apply them in most of their tests. Others claim that mocks lead to test fragility and try not to use them at all. 
* There’s a deep and almost inevitable connection between mocks and test fragility.

## 5.1 Differentiating mocks from stubs
###  5.1.1 The types of test doubles
* A `test double` is an overarching term that describes all kinds of non-production-ready, fake dependencies in tests.
  * mock (mock, spy)
    * Mocks help to emulate and examine `outcoming` interactions.
    * These interactions are calls the SUT makes to its dependencies to change their state.
  * stub (stub, dummy, fake)
    * Stubs help to emulate `incoming` interactions. 
    * These interactions are calls the SUT makes to its dependencies to get input data (figure 5.2).
* All other differences between the five variations are insignificant implementation details. 
  * For example, spies serve the same role as mocks. 
    * The distinction is that spies are written manually, whereas mocks are created with the help of a mocking framework. 
    * Sometimes people refer to spies as handwritten mocks.
  * On the other hand, the difference between a stub, a dummy, and a fake is in how intelligent they are. 
    * A dummy is a simple, hardcoded value such as a null value or a made-up string. 
      * It’s used to satisfy the SUT’s method signature and doesn’t participate in producing the final outcome. 
      * A stub is more sophisticated. It’s a fully fledged dependency that you configure to return different values for different scenarios. 
      * Finally, a fake is the same as a stub for most purposes. The difference is in the rationale for its creation: a fake is usually implemented to replace a dependency that doesn’t yet exist.
* Mocks help to `emulate and examine` interactions between the SUT and its dependencies, 
  * while stubs only help to `emulate` those interactions. This is an important distinction. You will see why shortly.

### 5.1.2 Mock (the tool) vs. mock (the test double)
* The term mock is overloaded and can mean different things in different circumstances
  * people often use this term to mean any test double, whereas mocks are only a subset of test doubles.
  * But there’s another meaning for the term mock. You can refer to the classes from mocking libraries as mocks, too.
    ```
    // Listing 5.1 Using the Mock class from a mocking library to create a mock
    public void Sending_a_greetings_email() {
        var mock = new Mock < IEmailGateway > (); // Uses a mock (the tool) to create a mock (the test double)
        var sut = new Controller(mock.Object);

        sut.GreetUser("user@email.com");

        mock.Verify(  // Examines the call from the SUT to the test double
            x => x.SendGreetingsEmail("user@email.com"),
            Times.Once
        );
    }
    ```
  * The test in the following listing also uses the Mock class, but the instance of that class is not a mock, it’s a stub.
    ```
    // Listing 5.2 Using the Mock class to create a stub
    public void Creating_a_report() {
        var stub = new Mock < IDatabase > (); // Uses a mock (the tool) to create a stub
        stub.Setup(x => x.GetNumberOfUsers()).Returns(10); // Sets up a canned answer
        var sut = new Controller(stub.Object);

        Report report = sut.CreateReport();
        Assert.Equal(10, report.NumberOfUsers);
    }
    ```
  * This test double emulates an `incoming` interaction—a call that provides the SUT with input data.
  * On the other hand, in the previous example (listing 5.1), the call to `SendGreetingsEmail()` is an `outcoming` interaction. Its sole purpose is to incur a side effect—send an email.p

### 5.1.3 Don’t assert interactions with stubs
* Asserting interactions with stubs is a common anti-pattern that leads to fragile tests.
* As you might remember from chapter 4, the only way to avoid false positives and thus improve resistance to refactoring in tests is to make those tests verify the end result (which, ideally, should be meaningful to a non-programmer), not implementation details.
* In listing 5.1, the check `mock.Verify(x => x.SendGreetingsEmail("user@email.com"))` corresponds to an actual outcome, and that outcome is meaningful to a domain expert: sending a greetings email is something business people would want the system to do.
* The following listing shows an example of such a brittle test.
  ```
  // Listing 5.3 Asserting an interaction with a stub
  public void Creating_a_report() {
      var stub = new Mock < IDatabase > ();
      stub.Setup(x => x.GetNumberOfUsers()).Returns(10);
      var sut = new Controller(stub.Object);

      Report report = sut.CreateReport();
      
      Assert.Equal(10, report.NumberOfUsers);
      stub.Verify( // Asserts the interaction with the stub
          x => x.GetNumberOfUsers(),
          Times.Once
      );
  }
  ```
  * This practice of verifying things that aren’t part of the end result is also called overspecification.
  
### 5.1.4 Using mocks and stubs together
* Sometimes you need to create a test double that exhibits the properties of both a mock and a stub.
  ```
  // Listing 5.4 storeMock: both a mock and a stub
  public void Purchase_fails_when_not_enough_inventory() {
      var storeMock = new Mock < IStore > ();
      storeMock  
          .Setup(x => x.HasEnoughInventory( // Sets up a canned answer
              Product.Shampoo, 5))
          .Returns(false);
      var sut = new Customer();

      bool success = sut.Purchase(storeMock.Object, Product.Shampoo, 5);

      Assert.False(success);
      storeMock.Verify(  // Examines a call from the SUT
          x => x.RemoveInventory(Product.Shampoo, 5),
              Times.Never
      );
  }
  ```
  * This test uses storeMock for two purposes: it returns a canned answer and verifies a method call made by the SUT. 
    * Notice, though, that these are two different methods: the test sets up the answer from `HasEnoughInventory()` 
    * but then verifies the call to `RemoveInventory()`.
  * Thus, the rule of not asserting interactions with stubs is not violated here.
* When a test double is both a mock and a stub, it’s still called a mock, not a stub.

### 5.1.5 How mocks and stubs relate to commands and queries
* The notions of mocks and stubs tie to the command query separation (CQS) principle.
  * The CQS principle states that every method should be either a command or a query, but not both.
    * As shown in figure 5.3, commands are methods that produce side effects and don’t return any value (return void). 
    * Examples of side effects include mutating an object’s state, changing a file in the file system, and so on. 
    * Queries are the opposite of that—they are side-effect free and return a value.
  * Of course, it’s not always possible to follow the CQS principle. 
    * There are always methods for which it makes sense to both incur a side effect and return a value. 
    * A classical example is stack.Pop(). This method both removes a top element from the stack and returns it to the caller.
  * Test doubles that substitute commands become mocks. Similarly, test doubles that substitute queries are stubs.
    * `SendGreetingsEmail()` is a command whose side effect is sending an email.
    * `GetNumberOfUsers()` is a query that returns a value and doesn’t mutate the database state.
      ```
      var mock = new Mock<IEmailGateway>();
      mock.Verify(x => x.SendGreetingsEmail("user@email.com"));

      var stub = new Mock<IDatabase>();
      stub.Setup(x => x.GetNumberOfUsers()).Returns(10);
      ```
## 5.2 Observable behavior vs. implementation details
* Shit in, shit out.
* In chapter 4, you also saw that the main reason tests deliver false positives (and thus fail at resistance to refactoring) is because they couple to the code’s implementation details. 
* The only way to avoid such coupling is to verify the end result the code produces (its observable behavior) and distance tests from implementation details as much as possible. 
* In other words, tests must focus on the whats, not the hows.

### 5.2.1 Observable behavior is not the same as a public API
* All production code can be categorized along two dimensions:
  * Public API vs. private API (where API means application programming interface)
  * Observable behavior vs. implementation details
* For a piece of code to be part of the system’s `observable` behavior, it has to do one of the following things:
  * Expose an operation that helps the client achieve one of its goals. 
    * An operation is a method that performs a calculation or incurs a side effect or both.
  * Expose a state that helps the client achieve one of its goals. 
    * State is the current condition of the system.
  * Any code that does neither of these two things is an `implementation detail`.
* Ideally, the system’s public API surface should coincide with its observable behavior, and all its implementation details should be hidden from the eyes of the clients.
  * Often, though, the system’s public API extends beyond its observable behavior and starts exposing implementation details. Such a system’s implementation details `leak` to its public API surface (figure 5.5).

### 5.2.2 Leaking implementation details: An example with an operation
* Let’s take a look at examples of code whose implementation details leak to the public API.
  ```
  // Listing 5.5 User class with leaking implementation details
  public class User {
      public string Name {
          get;
          set;
      }

      public string NormalizeName(string name) {
          string result = (name ? ? "").Trim();

          if (result.Length > 50)
              return result.Substring(0, 50);

          return result;
      }
  }

  public class UserController {
      public void RenameUser(int userId, string newName) {
          User user = GetUserFromDatabase(userId);

          string normalizedName = user.NormalizeName(newName);
          user.Name = normalizedName;

          SaveUserToDatabase(user);
      }
  }
  ```
  * So, why isn’t User’s API well-designed? Look at its members once again: 
    * the Name property and the NormalizeName method. Both of them are public. 
    * Therefore, in order for the class’s API to be well-designed, these members should be part of the observable behavior.
    * The only reason `UserController` calls this method is to satisfy the invariant of User. `NormalizeName` is therefore an implementation detail that leaks to the class’s public API (figure 5.6).
    * To fix the situation and make the class’s API well-designed, User needs to hide NormalizeName() and call it internally
      ```
      // Listing 5.6 A version of User with a well-designed API
      public class User {
          private string _name;
          public string Name {
              get => _name;
              set => _name = NormalizeName(value);
          }

          private string NormalizeName(string name) {
              string result = (name ? ? "").Trim();
          
              if (result.Length > 50)
                  return result.Substring(0, 50);
          
              return result;
          }
      }
      public class UserController {
          public void RenameUser(int userId, string newName) {
              User user = GetUserFromDatabase(userId);
              user.Name = newName;
              SaveUserToDatabase(user);
          }
      }
      ```
  * If the number of operations the client has to invoke on the class to achieve a single goal is greater than one, then that class is likely leaking implementation details.
    *  Ideally, any individual goal should be achieved with a single operation.

### 5.2.3 Well-designed API and encapsulation
* Maintaining a well-designed API relates to the notion of encapsulation.
* Without encapsulation, you have no practical way to cope with ever-increasing code complexity.
  * When the code’s API doesn’t guide you through what is and what isn’t allowed to be done with that code, you have to keep a lot of information in mind to make sure you don’t introduce inconsistencies with new code changes.

### 5.2.4 Leaking implementation details: An example with state
*  Let’s also look at an example with state. 
* Another guideline flows from the definition of a well-designed API: 
  * you should expose the absolute minimum number of operations and state.

## 5.3 The relationship between mocks and test fragility
### 5.3.1 Defining hexagonal architecture
### 5.3.2 Intra-system vs. inter-system communications
* Intra-system communications are implementation details because the collaborations your domain classes go through in order to perform an operation are not part of their observable behavior. 
  * These collaborations don’t have an immediate connection to the client’s goal. Thus, coupling to such collaborations leads to fragile tests.
* Inter-system communications are a different matter. 
  * Unlike collaborations between classes inside your application, the way your system talks to the external world forms the observable behavior of that system as a whole. 
  * It’s part of the contract your application must hold at all times (figure 5.12).
* The use of mocks is beneficial when verifying the communication pattern between your system and external applications.
  * Conversely, using mocks to verify communications between classes inside your system results in tests that couple to implementation details and therefore fall short of the resistance-to-refactoring metric.
### 5.3.3 Intra-system vs. inter-system communications: An example
* In the following listing, the CustomerController class is an application service that orchestrates the work between domain classes `(Customer, Product, Store)` and the external application `(EmailGateway, which is a proxy to an SMTP service)`.
  ```
  // Listing 5.9 Connecting the domain model with external applications
  public class CustomerController {
      public bool Purchase(int customerId, int productId, int quantity) {
          Customer customer = _customerRepository.GetById(customerId);
          Product product = _productRepository.GetById(productId);
  
          bool isSuccess = customer.Purchase(_mainStore, product, quantity);
  
          if (isSuccess) {
              _emailGateway.SendReceipt(
                  customer.Email, product.Name, quantity);
          }
  
          return isSuccess;
      }
  }
  ```
  * The client of the application is the third-party system. 
  * This system’s goal is to make a purchase, and it expects the cus- tomer to receive a confirmation email as part of the successful outcome.
  * The call to the SMTP service is a legitimate reason to do mocking. 
    * It doesn’t lead to test fragility because you want to make sure this type of communication stays in place even after refactoring.

  ```
  // Listing 5.10 Mocking that doesn’t lead to fragile tests
  public void Successful_purchase() {
      var mock = new Mock < IEmailGateway > ();
      var sut = new CustomerController(mock.Object);
      
      bool isSuccess = sut.Purchase(customerId: 1, productId: 2, quantity: 5);
      
      Assert.True(isSuccess);
      mock.Verify( // Verifies that the system sent a receipt about the purchase
          x => x.SendReceipt("customer@email.com", "Shampoo", 5),
          Times.Once
      );
  }

  // Listing 5.11 Mocking that leads to fragile tests
  public void Purchase_succeeds_when_enough_inventory() {
      var storeMock = new Mock < IStore > ();
      storeMock
          .Setup(x => x.HasEnoughInventory(Product.Shampoo, 5))
          .Returns(true);
      var customer = new Customer();
      
      bool success = customer.Purchase(storeMock.Object, Product.Shampoo, 5);
      
      Assert.True(success);
      storeMock.Verify(
          x => x.RemoveInventory(Product.Shampoo, 5),
          Times.Once
      );
  }
  ```
    * Unlike the communication between `CustomerController` and the SMTP service, the `RemoveInventory()` method call from Customer to Store doesn’t cross the application boundary: both the caller and the recipient reside inside the application. 
    * Also, this method is neither an operation nor a state that helps the client achieve its goals. 
    * The client of these two domain classes is `CustomerController` with the goal of making a purchase. 

## 5.4 The classical vs. London schools of unit testing, revisited
* In chapter 2, I mentioned that I prefer the classical school of unit testing over the London school. I hope now you can see why. 
  * The London school encourages the use of mocks for all but immutable dependencies and doesn’t differentiate between intra-system and inter-system communications. 
  * As a result, tests check communications between classes just as much as they check communications between your application and external systems.
* This indiscriminate use of mocks is why following the London school often results in tests that couple to implementation details and thus lack resistance to refactoring.

### 5.4.1 Not all out-of-process dependencies should be mocked out
* types of dependencies (refer to chapter 2 for more details):
  * `Shared dependency` — A dependency shared by tests (not production code)
  * `Out-of-process dependency` — A dependency hosted by a process other than the pro- gram’s execution process (for example, a database, a message bus, or an SMTP service)
  * `Private dependency` — Any dependency that is not shared
* The classical school recommends avoiding shared dependencies because they provide the means for tests to interfere with each other’s execution context and thus prevent those tests from running in parallel. 
  * The ability for tests to run in parallel, sequentially, and in any order is called `test isolation`.
* If a shared dependency is not out-of-process, then it’s easy to avoid reusing it in tests by providing a new instance of it on each test run. 
  * In cases where the shared dependency is out-of-process, testing becomes more complicated.
  * The usual approach is to replace such dependencies with test doubles—mocks and stubs.
* Not all out-of-process dependencies should be mocked out, though.
  * `If an out-of- process dependency is only accessible through your application, then communications with such a dependency are not part of your system’s observable behavior.`
  * An out-of-process dependency that can’t be observed externally, in effect, acts as part of your application (figure 5.14).
    * Figure 5.14 Communications with an out-of-process dependency that can’t be observed externally are implementation details. They don’t have to stay in place after refactoring and therefore shouldn’t be verified with mocks.
    * A good example here is an application database: a database that is used only by your application. 
    * No external system has access to this database. Therefore, you can modify the communication pattern between your system and the application database in any way you like, as long as it doesn’t break existing functionality. 
    * Because that database is completely hidden from the eyes of the clients, you can even replace it with an entirely different storage mechanism, and no one will notice.
  * The use of mocks for out-of-process dependencies that you have a full control over also leads to brittle tests.

### 5.4.2 Using mocks to verify behavior
* Mocks are often said to verify behavior. 
  * In the vast majority of cases, they don’t. 
  * The way each individual class interacts with neighboring classes in order to achieve some goal has nothing to do with observable behavior; it’s an implementation detail. 
  * Such a level of detail is too granular. What matters is the behavior that can be traced back to the client goals.
* Mocks have something to do with behavior only when they verify interactions that cross the application boundary and only when the side effects of those interactions are visible to the external world.

# Chapter 6 Styles of unit testing
* There are three such styles: output-based, state-based, and communication- based testing. 
  * Among the three, the output-based style produces tests of the highest quality, state-based testing is the second-best choice, and communication-based testing should be used only occasionally.

## 6.1 The three styles of unit testing
* As I mentioned in the chapter introduction, there are three styles of unit testing:
  * Output-based testing
  * State-based testing
  * Communication-based testing
* You can employ one, two, or even all three styles together in a single test.

### 6.1.1 Defining the output-based style
* The first style of unit testing is the `output-based` style, where you feed an input to the system under test (SUT) and check the output it produces (figure 6.1). 
  * This style of unit testing is only applicable to code that doesn’t change a global or internal state, so the only component to verify is its return value.
  * The output-based style of unit testing is also known as functional. This name takes root in functional programming, a method of programming that emphasizes a preference for side-effect-free code.
  ```
  // Listing 6.1 Output-based testing
  public class PriceEngine {
      public decimal CalculateDiscount(params Product[] products) {
          decimal discount = products.Length * 0.01;
          return Math.Min(discount, 0.2);
      }
  }

  public void Discount_of_two_products() {
      var product1 = new Product("Hand wash");
      var product2 = new Product("Shampoo");
      var sut = new PriceEngine();

      decimal discount = sut.CalculateDiscount(product1, product2);

      Assert.Equal(0.02, discount);
  }
  ```
### 6.1.2 Defining the state-based style
* The `state-based` style is about verifying the state of the system after an operation is com plete (figure 6.3). 
  * The term state in this style of testing can refer to the state 
    * of the SUT itself, 
    * of one of its collaborators, 
    * or of an out-of-process dependency, such as the database or the filesystem.
  ```
  // Listing 6.2 State-based testing
  public class Order {
      private readonly List < Product > _products = new List < Product > ();
      public IReadOnlyList < Product > Products => _products.ToList();

      public void AddProduct(Product product) {
          _products.Add(product);
      }
  }

  public void Adding_a_product_to_an_order() {
      var product = new Product("Hand wash");
      var sut = new Order();

      sut.AddProduct(product);

      Assert.Equal(1, sut.Products.Count);
      Assert.Equal(product, sut.Products[0]);
  }
  ```
### 6.1.3 Defining the communication-based style
* Finally, the third style of unit testing is `communication-based` testing. 
  * This style uses mocks to verify communications between the system under test and its collaborators (figure 6.4).
  ```
  // Listing 6.3 Communication-based testing
  public void Sending_a_greetings_email() {
      var emailGatewayMock = new Mock <IEmailGateway> ();
      var sut = new Controller(emailGatewayMock.Object);
      
      sut.GreetUser("user@email.com");
      
      emailGatewayMock.Verify(
          x => x.SendGreetingsEmail("user@email.com"),
          Times.Once
      );
  }
  ```

## 6.2 Comparing the three styles of unit testing
* What’s interesting is comparing them to each other using the four attributes of a good unit test. Here are those attributes again (refer to chapter 4 for more details):
  * Protection against regressions 
  * Resistance to refactoring
  * Fast feedback
  * Maintainability

### 6.2.1 Comparing the styles using the metrics of protection against regressions and feedback speed
* This metric is a product of the following three characteristics:
  * The amount of code that is executed during the test 
  * The complexity of that code
  * Its domain significance
* Generally, you can write a test that exercises as much or as little code as you like; no particular style provides a benefit in this area.
* The same is true for the code’s complexity and domain significance.
  * The only exception is the communication-based style: overusing it can result in shallow tests that verify only a thin slice of code and mock out everything else. 
  * Such shallowness is not a definitive feature of communication-based testing, though, but rather is an extreme case of abusing this technique.

### 6.2.2 Comparing the styles using the metric of resistance to refactoring
* When it comes to the metric of resistance to refactoring, the situation is different. Resistance to refactoring is the measure of how many false positives (false alarms) tests gen- erate during refactorings. 
  * False positives, in turn, are a result of tests coupling to code’s implementation details as opposed to observable behavior.
* Output-based testing provides the best protection against false positives because the resulting tests couple only to the method under test.
* State-based testing is usually more prone to false positives. In addition to the method under test, such tests also work with the class’s state. 
  * Probabilistically speaking, the greater the coupling between the test and the production code, the greater the chance for this test to tie to a leaking implementation detail. 
* Communication-based testing is the most vulnerable to false alarms. As you may remember from chapter 5, the vast majority of tests that check interactions with test doubles end up being brittle. 
  * This is always the case for interactions with stubs—you should never check such interactions. 
  * `Mocks are fine only when they verify interactions that cross the application boundary and only when the side effects of those interactions are visible to the external world.`
  * You can reduce the number of false positives to a minimum by maintaining proper encapsulation and coupling tests to observable behavior only.

### 6.2.3 Comparing the styles using the metric of maintainability
* Maintainability evaluates the unit tests’ maintenance costs and is defined by the following two characteristics:
  * How hard it is to understand the test, which is a function of the test’s size
  * How hard it is to run the test, which is a function of how many out-of-process dependencies the test works with directly
* Larger tests are less maintainable because they are harder to grasp or change when needed.
* Similarly, a test that directly works with one or several out-of-process dependencies (such as the database) is less maintainable because you need to spend time keeping those out-of-process dependencies operational: rebooting the database server, resolving network connectivity issues, and so on.
* MAINTAINABILITY OF OUTPUT-BASED TESTS
  * Compared with the other two types of testing, output-based testing is the most main- tainable. 
* MAINTAINABILITY OF STATE-BASED TESTS
  * State-based tests are normally less maintainable than output-based ones. This is because state verification often takes up more space than output verification.
    ```
    // Listing 6.4 State verification that takes up a lot of space
    public void Adding_a_comment_to_an_article() {
        var sut = new Article();
        var text = "Comment text";
        var author = "John Doe";
        var now = new DateTime(2019, 4, 1);

        sut.AddComment(text, author, now);

        Assert.Equal(1, sut.Comments.Count);
        Assert.Equal(text, sut.Comments[0].Text);
        Assert.Equal(author, sut.Comments[0].Author);
        Assert.Equal(now, sut.Comments[0].DateCreated);
    }
    ```
  * Although this test is simplified and con- tains just a single comment, its assertion part already spans four lines. 
    * State-based tests often need to verify much more data than that and, therefore, can grow in size significantly.
    ```
    // Listing 6.5 Using helper methods in assertions
    public void Adding_a_comment_to_an_article() {
        var sut = new Article();
        var text = "Comment text";
        var author = "John Doe";
        var now = new DateTime(2019, 4, 1);

        sut.AddComment(text, author, now);

        sut.ShouldContainNumberOfComments(1).WithComment(text, author, now); // helper method
    }
    ```
  * Another way to shorten a state-based test is to define equality members in the class that is being asserted.
    ```
    // Listing 6.6 Comment compared by value
    public void Adding_a_comment_to_an_article() {
        var sut = new Article();
        var comment = new Comment(
            "Comment text",
            "John Doe",
            new DateTime(2019, 4, 1));

        sut.AddComment(comment.Text, comment.Author, comment.DateCreated);
        
        sut.Comments.Should().BeEquivalentTo(comment);
    }
    ```
  * This is a powerful technique, but it works only when the class is inherently a `value` and can be converted into a `value object`.
    * Otherwise, it leads to code pollution (pollut- ing production code base with code whose sole purpose is to enable or, as in this case, simplify unit testing).
  * As you can see, these two techniques—using helper methods and converting classes into value objects—are applicable only occasionally. 
* MAINTAINABILITY OF COMMUNICATION-BASED TESTS
  * Communication-based tests score worse than output-based and state-based tests on the maintainability metric. 
  * Communication-based testing requires setting up test doubles and interaction assertions, and that takes up a lot of space.
    * Tests become even larger and less maintainable when you have mock chains (mocks or stubs returning other mocks, which also return mocks, and so on, several layers deep).

### 6.2.4 Comparing the styles: The results
* Output-based testing shows the best results. This style produces tests that rarely couple to implementation details and thus don’t require much due diligence to main- tain proper resistance to refactoring. Such tests are also the most maintainable due to their conciseness and lack of out-of-process dependencies.
* Always prefer output-based testing over everything else. Unfortunately, it’s easier said than done. 
  * This style of unit testing is only applicable to code that is written in a functional way, which is rarely the case for most object-oriented programming languages.

## 6.3 Understanding functional architecture
* For a deeper look at functional programming, see Scott Wlaschin’s website and books at https://fsharpforfunandprofit.com/books.
### 6.3.1 What is functional programming?
* Functional programming is programming with mathematical functions.
* Let’s take the `CalculateDiscount()` method from listing 6.1 as an example (I’m copying it here for convenience):
  ```
  public decimal CalculateDiscount(Product[] products)
  {
      decimal discount = products.Length * 0.01m;
      return Math.Min(discount, 0.2m);
  }
  ```
  * This method has 
    * one input (a Product array) 
    * and one output (the decimal discount), both of which are explicitly expressed in the method’s signature. 
    * There are no hidden inputs or outputs. 
  * On the other hand, hidden inputs and outputs make the code less testable (and less readable, too). Types of such hidden inputs and outputs include the following:
    * `Side effects` — A side effect is an output that isn’t expressed in the method signature and, therefore, is hidden. An operation creates a side effect when it mutates the state of a class instance, updates a file on the disk, and so on.
    * `Exceptions` — When a method throws an exception, it creates a path in the program flow that bypasses the contract established by the method’s signature. The thrown exception can be caught anywhere in the call stack, thus introducing an additional output that the method signature doesn’t convey.
    * `A reference to an internal or external state` — For example, a method can get the current date and time using a static property such as `DateTime.Now`. It can query data from the database, or it can refer to a private mutable field. These are all inputs to the execution flow that aren’t present in the method signature and, therefore, are hidden.
* A good rule of thumb when determining whether a method is a mathematical function is to see if you can replace a call to that method with its return value without changing the program’s behavior. The ability to replace a method call with the corresponding value is known as `referential transparency`. 
  ```
  // This method is a mathematical function.
  public int Increment(int x) {
    return x + 1; 
  }
  ```
  * On the other hand, the following method is not a mathematical function. In this example, the hidden output is the change to field x (a side effect):
    ```
    int x = 0;
    
    public int Increment() {
      x++;
      return x; 
    }
    ```
  ```
  // Listing 6.7 Modification of an internal state
  public Comment AddComment(string text) {
      var comment = new Comment(text);
      _comments.Add(comment); // Side effect
      return comment;
  }
  ```
### 6.3.2 What is functional architecture?
* You can’t create an application that doesn’t incur any side effects whatsoever, of course. 
  * Such an application would be impractical. 
  * After all, side effects are what you create all applications for: updating the user’s information, adding a new order line to the shopping cart, and so on.
* The goal of functional programming is 
  * not to eliminate side effects altogether 
  * but rather to introduce a separation between code that handles business logic and code that incurs side effects.
* These two responsibilities are complex enough on their own; mixing them together multiplies the complexity and hinders code maintainability in the long run. 
  * This is where functional architecture comes into play. It separates business logic from side effects by `pushing those side effects to the edges of a business operation.`
  * Functional architecture 
    * maximizes the amount of code written in a purely functional (immutable) way, while minimizing code that deals with side effects. 
    * Immutable means unchangeable: once an object is created, its state can’t be modified. This is in contrast to a mutable object (changeable object), which can be modified after it is created.
* The separation between business logic and side effects is done by segregating two types of code:
  * Code that makes a decision — This code doesn’t require side effects and thus can be written using mathematical functions.
  * Code that acts upon that decision — This code converts all the decisions made by the mathematical functions into visible bits, such as changes in the database or messages sent to a bus.
* The code that makes decisions is often referred to as a functional core (also known as an immutable core). The code that acts upon those decisions is a mutable shell (figure 6.9).
  * To maintain a proper separation between these two layers, you need to make sure the classes representing the decisions contain enough information for the mutable shell to act upon them without additional decision-making. 
  * In other words, the mutable shell should be as dumb as possible. 
  * The goal is to cover the functional core extensively with output-based tests and leave the mutable shell to a much smaller number of integration tests.

* Encapsulation and immutability
  * Like encapsulation, functional architecture (in general) and immutability (in particular) serve the same goal as unit testing: enabling sustainable growth of your software project.
    * In fact, there’s a deep connection between the concepts of encapsulation and immutability.
  * Encapsulation safeguards the class’s internals from corruption by
    * Reducing the API surface area that allows for data modification 
    * Putting the remaining APIs under scrutiny
  * Immutability tackles this issue of preserving invariants from another angle. With immutable classes, you don’t need to worry about state corruption because it’s impossible to corrupt something that cannot be changed in the first place. 
    * As a consequence, there’s no need for encapsulation in functional programming.

### 6.3.3 Comparing functional and hexagonal architectures
* There are a lot of similarities between functional and hexagonal architectures. 
  * Both of them are built around the idea of separation of concerns.
  * The details of that sepa- ration vary, though.
* As you may remember from chapter 5, the hexagonal architecture differentiates the domain layer and the application services layer (figure 6.10). 
  * The domain layer is accountable for business logic while the application services layer, for communication with external applications such as a database or an SMTP service. 
    * This is very similar to functional architecture, where you introduce the separation of decisions and actions.
  * Another similarity is the one-way flow of dependencies. 
    * In the hexagonal architecture, classes inside the domain layer should only depend on each other; they should not depend on classes from the application services layer. 
    * Likewise, the immutable core in functional architecture doesn’t depend on the mutable shell. 
    * It’s self-sufficient and can work in isolation from the outer layers. 
    * This is what makes functional architecture so testable: you can strip the immutable core from the mutable shell entirely and simulate the inputs that the shell provides using simple values.
* The difference between the two is in their treatment of side effects. 
  * Functional architecture pushes all side effects out of the immutable core to the edges of a business operation. 
    * These edges are handled by the mutable shell. 
  * On the other hand, the hexagonal architecture is fine with side effects made by the domain layer, as long as they are limited to that domain layer only.
    * All modifications in hexagonal architecture should be contained within the domain layer and not cross that layer’s boundary. 
    * For example, a domain class instance can’t persist something to the database directly, but it can change its own state. 
    * An application service will then pick up this change and apply it to the database.
* Functional architecture is a subset of the hexagonal architecture. You can view functional architecture as the hexagonal architecture taken to an extreme.

## 6.4 Transitioning to functional architecture and output- based testing
* In this section, we’ll take a sample application and refactor it toward functional architecture. You’ll see two refactoring stages:
  * Moving from using an out-of-process dependency to using mocks 
  * Moving from using mocks to using functional architecture

### 6.4.1 Introducing an audit system
* The AuditManager class is hard to test as-is, because it’s tightly coupled to the file-system. Before the test, you’d need to put files in the right place, and after the test finishes, you’d read those files, check their contents, and clear them out (figure 6.12).
  ```
  // Listing 6.8 Initial implementation of the audit system
  public class AuditManager {
      private readonly int _maxEntriesPerFile;
      private readonly string _directoryName;

      public AuditManager(int maxEntriesPerFile, string directoryName) {
          _maxEntriesPerFile = maxEntriesPerFile;
          _directoryName = directoryName;
      }

      public void AddRecord(string visitorName, DateTime timeOfVisit) {
          string[] filePaths = Directory.GetFiles(_directoryName);
          (int index, string path)[] sorted = SortByIndex(filePaths);
          
          string newRecord = visitorName + ';' + timeOfVisit;
          
          if (sorted.Length == 0) {
              string newFile = Path.Combine(_directoryName, "audit_1.txt");
              File.WriteAllText(newFile, newRecord);
              return;
          }
          
          (int currentFileIndex, string currentFilePath) = sorted.Last();
          List < string > lines = File.ReadAllLines(currentFilePath).ToList();
          
          if (lines.Count < _maxEntriesPerFile) {
              lines.Add(newRecord);
              string newContent = string.Join("\r\n", lines);
              File.WriteAllText(currentFilePath, newContent);
          } else {
              int newIndex = currentFileIndex + 1;
              string newName = $ "audit_{newIndex}.txt";
              string newFile = Path.Combine(_directoryName, newName);
              File.WriteAllText(newFile, newRecord);
          }
      }
  }
  ```
  * You won’t be able to parallelize such tests—at least, not without additional effort that would significantly increase maintenance costs. The bottleneck is the filesys- tem: it’s a shared dependency through which tests can interfere with each other’s execution flow.
  * The filesystem also makes the tests slow. Maintainability suffers, too, because you have to make sure the working directory exists and is accessible to tests—both on your local machine and on the build server. Table 6.2 sums up the scoring.
  * By the way, tests working directly with the filesystem don’t fit the definition of a unit test. They don’t comply with the second and the third attributes of a unit test, thereby falling into the category of integration tests (see chapter 2 for more details):
    * A unit test verifies a single unit of behavior, 
    * Does it quickly,
    * And does it in isolation from other tests.

### 6.4.2 Using mocks to decouple tests from the filesystem
```
// Listing 6.9 Injecting the filesystem explicitly via the constructor
public class AuditManager {
    private readonly int _maxEntriesPerFile;
    private readonly string _directoryName;
    private readonly IFileSystem _fileSystem; // The new interface represents the filesystem.

    public AuditManager(
        int maxEntriesPerFile,
        string directoryName,
        IFileSystem fileSystem) {
        _maxEntriesPerFile = maxEntriesPerFile;
        _directoryName = directoryName;
        _fileSystem = fileSystem;
    }
}

// Listing 6.10 Using the new IFileSystem interface 
public void AddRecord(string visitorName, DateTime timeOfVisit) {
    string[] filePaths = _fileSystem.GetFiles(_directoryName); // The new interface in action
    (int index, string path)[] sorted = SortByIndex(filePaths);

    string newRecord = visitorName + ';' + timeOfVisit;

    if (sorted.Length == 0) {
        string newFile = Path.Combine(_directoryName, "audit_1.txt");
        _fileSystem.WriteAllText(newFile, newRecord); // The new interface in action
        return;
    }

    (int currentFileIndex, string currentFilePath) = sorted.Last();
    List < string > lines = _fileSystem.ReadAllLines(currentFilePath);

    if (lines.Count < _maxEntriesPerFile) {
        lines.Add(newRecord);
        string newContent = string.Join("\r\n", lines);
        _fileSystem.WriteAllText(currentFilePath, newContent); // The new interface in action
    } else {
        int newIndex = currentFileIndex + 1;
        string newName = $ "audit_{newIndex}.txt";
        string newFile = Path.Combine(_directoryName, newName);
        _fileSystem.WriteAllText(newFile, newRecord); // The new interface in action
    }
}

public interface IFileSystem
{
    string[] GetFiles(string directoryName);
    void WriteAllText(string filePath, string content);
    List<string> ReadAllLines(string filePath);
}
```
* Now that AuditManager is decoupled from the filesystem, the shared dependency is gone, and tests can execute independently from each other. Here’s one such test.
  ```
  // Listing 6.11 Checking the audit system’s behavior using a mock
  public void A_new_file_is_created_when_the_current_file_overflows() {
      var fileSystemMock = new Mock<IFileSystem>();
      fileSystemMock
          .Setup(x => x.GetFiles("audits"))
          .Returns(new string[] {
              @"audits\audit_1.txt",
              @"audits\audit_2.txt"
          });
      fileSystemMock
          .Setup(x => x.ReadAllLines(@"audits\audit_2.txt"))
          .Returns(new List<string> {
              "Peter; 2019-04-06T16:30:00",
              "Jane; 2019-04-06T16:40:00",
              "Jack; 2019-04-06T17:00:00"
          });
      var sut = new AuditManager(3, "audits", fileSystemMock.Object);

      sut.AddRecord("Alice", DateTime.Parse("2019-04-06T18:00:00"));

      fileSystemMock.Verify(x => x.WriteAllText(
          @"audits\audit_3.txt",
          "Alice;2019-04-06T18:00:00"));
  }
  ```
  * This test verifies that when the number of entries in the current file reaches the limit (3, in this example), a new file with a single audit entry is created. Note that this is a legitimate use of mocks.
  * The application creates files that are visible to end users (assuming that those users use another program to read the files, be it specialized soft- ware or a simple notepad.exe). 
  * Therefore, communications with the filesystem and the side effects of these communications (that is, the changes in files) are part of the application’s observable behavior.

### 6.4.3 Refactoring toward functional architecture
* Instead of hiding side effects behind an interface and injecting that interface into AuditManager, you can move those side effects out of the class entirely. Audit- Manager is then only responsible for making a decision about what to do with the files.
* A new class, Persister, acts on that decision and applies updates to the filesys- tem (figure 6.14).
  ```
  // Listing 6.12 The AuditManager class after refactoring
  public class AuditManager {
      private readonly int _maxEntriesPerFile;

      public AuditManager(int maxEntriesPerFile) {
          _maxEntriesPerFile = maxEntriesPerFile;
      }

      public FileUpdate AddRecord(
          FileContent[] files,
          string visitorName,
          DateTime timeOfVisit) 
      {
          (int index, FileContent file)[] sorted = SortByIndex(files);
          string newRecord = visitorName + ';' + timeOfVisit;
          
          if (sorted.Length == 0) {
              return new FileUpdate("audit_1.txt", newRecord);
          }

          (int currentFileIndex, FileContent currentFile) = sorted.Last();
          List<string> lines = currentFile.Lines.ToList();

          if (lines.Count < _maxEntriesPerFile) {
              lines.Add(newRecord);
              string newContent = string.Join("\r\n", lines);
              return new FileUpdate(currentFile.FileName, newContent);
          } else {
              int newIndex = currentFileIndex + 1;
              string newName = $ "audit_{newIndex}.txt";
              return new FileUpdate(newName, newRecord);
          }
      }
  }

  public class FileContent {
      public readonly string FileName;
      public readonly string[] Lines;
      public FileContent(string fileName, string[] lines)
      {
          FileName = fileName;
          Lines = lines;
      }
  }

  public class FileUpdate
  {
      public readonly string FileName;
      public readonly string NewContent;
      public FileUpdate(string fileName, string newContent)
      {
          FileName = fileName;
          NewContent = newContent;
      }
  }

  // Listing 6.13 The mutable shell acting on AuditManager’s decision
  public class Persister
  {
      public FileContent[] ReadDirectory(string directoryName)
      {
          return Directory
              .GetFiles(directoryName)
              .Select(x => new FileContent(
                  Path.GetFileName(x),
                  File.ReadAllLines(x))
              ).ToArray();
      }
      public void ApplyUpdate(string directoryName, FileUpdate update)
      {
          string filePath = Path.Combine(directoryName, update.FileName);
          File.WriteAllText(filePath, update.NewContent);
      }
  }
  ```
  * Notice how trivial this class is. All it does is read content from the working directory and apply updates it receives from AuditManager back to that working directory. 
  * It has no branching (no if statements); all the complexity resides in the AuditManager class. `This is the separation between business logic and side effects in action.`
  * To maintain such a separation, you need to keep the interface of FileContent and FileUpdate as close as possible to that of the framework’s built-in file-interaction commands. 
  ```
  // Listing 6.14 Gluing together the functional core and mutable shell
  public class ApplicationService {
      private readonly string _directoryName;
      private readonly AuditManager _auditManager;
      private readonly Persister _persister;

      public ApplicationService(string directoryName, int maxEntriesPerFile) {
          _directoryName = directoryName;
          _auditManager = new AuditManager(maxEntriesPerFile);
          _persister = new Persister();
      }

      public void AddRecord(string visitorName, DateTime timeOfVisit) {
          FileContent[] files = _persister.ReadDirectory(_directoryName);
          FileUpdate update = _auditManager.AddRecord(files, visitorName, timeOfVisit);
          _persister.ApplyUpdate(_directoryName, update);
      }
  }
  ```
  * Along with gluing the functional core together with the mutable shell, the application service also provides an entry point to the system for external clients (figure 6.15).
  ```
  // Listing 6.15 The test without mocks
  public void A_new_file_is_created_when_the_current_file_overflows() {
      var sut = new AuditManager(3);
      var files = new FileContent[] {
          new FileContent("audit_1.txt", new string[0]),
          new FileContent("audit_2.txt", new string[] {
              "Peter; 2019-04-06T16:30:00",
              "Jane; 2019-04-06T16:40:00",
              "Jack; 2019-04-06T17:00:00"
          })
      };
      
      FileUpdate update = sut.AddRecord(files, "Alice", DateTime.Parse("2019-04-06T18:00:00"));

      Assert.Equal("audit_3.txt", update.FileName);
      Assert.Equal("Alice;2019-04-06T18:00:00", update.NewContent);
  }
  ```

### 6.4.4 Looking forward to further developments

## 6.5 Understanding the drawbacks of functional architecture
* Unfortunately, functional architecture isn’t always attainable. 
  * And even when it is, the maintainability benefits are often offset by a performance impact and increase in the size of the code base.

### 6.5.1 Applicability of functional architecture
* Functional architecture worked for our audit system because this system could gather all the inputs up front, before making a decision. 
  * Often, though, the execution flow is less straightforward. 
  * You might need to query additional data from an out-of-process dependency, based on an intermediate result of the decision-making process.
* Collaborators vs. values
  * You may have noticed that AuditManager’s AddRecord() method has a dependency that’s not present in its signature: the `_maxEntriesPerFile `field. The audit manager refers to this field to make a decision to either append an existing audit file or create a new one.
  * Although this dependency isn’t present among the method’s arguments, it’s not hid den. It can be derived from the class’s constructor signature. 
  * And because the `_maxEntriesPerFile` field is `immutable`, it stays the same between the class instantiation and the call to `AddRecord()`. 
    * In other words, that field is a value.
  * The situation with the IDatabase dependency is different because it’s a collaborator, not a value like _maxEntriesPerFile. As you may remember from chapter 2, a `collaborator` is a dependency that is one or the other of the following:
    * Mutable (allows for modification of its state)
    * A proxy to data that is not yet in memory (a shared dependency)
  * The `IDatabase` instance falls into the second category and, therefore, is a collabo- rator. It requires an additional call to an out-of-process dependency and thus pre- cludes the use of output-based testing.

### 6.5.2 Performance drawbacks
* The performance impact on the system as a whole is a common argument against functional architecture. Note that it’s not the performance of tests that suffers. 
  * The output-based tests we ended up with work as fast as the tests with mocks. 
  * It’s that the system itself now has to do more calls to out-of-process dependencies and becomes less performant. 
  * The initial version of the audit system didn’t read all files from the working directory, and neither did the version with mocks. 
    * But the final version does in order to comply with the read-decide-act approach.
* The choice between a functional architecture and a more traditional one is `a trade-off between performance and code maintainability (both production and test code)`. 
  * In some systems where the performance impact is not as noticeable, it’s better to go with functional architecture for additional gains in maintainability. 
  * In others, you might need to make the opposite choice. There’s no one-size-fits-all solution.

### 6.5.3 Increase in the code base size
* The same is true for the size of the code base. Functional architecture requires a clear separation between the functional (immutable) core and the mutable shell. 
  * This necessitates additional coding initially, 
  * although it ultimately results in reduced code complexity and gains in maintainability.
* Not all projects exhibit a high enough degree of complexity to justify such an initial investment, though. 
  * Some code bases aren’t that significant from a business perspective or are just plain too simple. 
  * It doesn’t make sense to use functional architecture in such projects because the initial investment will never pay off. 
  * Always apply functional architecture strategically, taking into account the complexity and importance of your system.
* Finally, don’t go for purity of the functional approach if that purity comes at too high a cost. 
  * In most projects, you won’t be able to make the domain model fully immutable and thus can’t rely solely on output-based tests, at least not when using an OOP language like C# or Java. 
  * In most cases, you’ll have a combination of output-based and state-based styles, with a small mix of communication-based tests, and that’s fine.
  * The goal of this chapter is not to incite you to transition all your tests toward the output-based style; the goal is to transition as many of them as reasonably possible.

# Chapter 7 Refactoring toward valuable unit tests
* In chapter 1, I defined the properties of a good unit test suite:
  * It is integrated into the development cycle.
  * It targets only the most important parts of your code base.
  * It provides maximum value with minimum maintenance costs. To achieve this last attribute, you need to be able to:
    * Recognize a valuable test (and, by extension, a test of low value).
    * Write a valuable test.
* Chapter 4 covered the topic of recognizing a valuable test using the four attributes: protection against regressions, resistance to refactoring, fast feedback, and main- tainability. 
* And chapter 5 expanded on the most important one of the four: resis- tance to refactoring.
* You saw an example of a code base transformation in chapter 6, where we refac- tored an audit system toward a functional architecture and, as a result, were able to apply output-based testing. 

## 7.1 Identifying the code to refactor
* It’s rarely possible to significantly improve a test suite without refactoring the underlying code. 

### 7.1.1 The four types of code
* In this section, I describe the four types of code that serve as a foundation for the rest of this chapter.
  * All production code can be categorized along two dimensions: 
    * Complexity or domain significance
    * The number of collaborators
  * `Code complexity` is defined by the number of decision-making (branching) points in the code. The greater that number, the higher the complexity.
  * `Domain significance` shows how significant the code is for the problem domain of your project. 
    * Normally, all `code in the domain layer` has a direct connection to the end users’ goals and thus exhibits a high domain significance. 
    * On the other hand, `utility code` doesn’t have such a connection.
* `Complex code and code that has domain significance` benefit from unit testing the most because the corresponding tests have great protection against regressions.
* The second dimension is the `number of collaborators` a class or a method has.
  * As you may remember from chapter 2, a collaborator is a dependency that is either mutable or out-of-process (or both). 
  * Code with a large number of collaborators is expensive to test. That’s due to the maintainability metric, which depends on the size of the test.
  * It takes space to bring collaborators to an expected condition and then check their state or interactions with them afterward. And the more collaborators there are, the larger the test becomes.
* `The type of the collaborators also matters.` 
  * Out-of-process collaborators are a no-go when it comes to the domain model. 
  * They add additional maintenance costs due to the necessity to maintain complicated mock machinery in tests. 
  * You also have to be extra prudent and only use mocks to verify interactions that cross the application boundary in order to maintain proper resistance to refactoring (refer to chapter 5 for more details). 
  * It’s better to delegate all communications with out-of-process dependencies to classes outside the domain layer. The domain classes then will only work with in-process dependencies.
* The combination of code complexity, its domain significance, and the number of collaborators give us the four types of code shown in figure 7.1:
  * `Domain model and algorithms` (figure 7.1, top left)—Complex code is often part of the domain model but not in 100% of all cases. You might have a complex algorithm that’s not directly related to the problem domain.
    * Unit testing the top-left quadrant (domain model and algorithms) gives you the best return for your efforts. The resulting unit tests are highly valuable and cheap. 
  * `Trivial code` (figure 7.1, bottom left)—Examples of such code in C# are parameter- less constructors and one-line properties: they have few (if any) collaborators and exhibit little complexity or domain significance.
    * Trivial code shouldn’t be tested at all; such tests have a close-to-zero value.
  * `Controllers` (figure 7.1, bottom right)—This code doesn’t do complex or business- critical work by itself but coordinates the work of other components like domain classes and external applications.
    * As for controllers, you should test them briefly as part of a much smaller set of the overarching integration tests (I cover this topic in part 3).
  * `Overcomplicated code` (figure 7.1, top right)—Such code scores highly on both metrics: it has a lot of collaborators, and it’s also complex or important. An example here are fat controllers (controllers that don’t delegate complex work anywhere and do everything themselves).
    * The most problematic type of code is the overcomplicated quadrant. It’s hard to unit test but too risky to leave without test coverage.
    * Such code is one of the main rea- sons many people struggle with unit testing.
* `The more important or complex the code, the fewer collaborators it should have.`
* Getting rid of the overcomplicated code and unit testing only the domain model and algorithms is the path to a highly valuable, easily maintainable test suite. With this approach, you won’t have 100% test coverage, but you don’t need to—100% coverage shouldn’t ever be your goal. 
  * Your goal is a test suite where each test adds significant value to the project. Refactor or get rid of all other tests. 
  * Don’t allow them to inflate the size of your test suite.

### 7.1.2 Using the Humble Object pattern to split overcomplicated code
* To split overcomplicated code, you need to use the `Humble Object design pattern`. 
  * This pattern was introduced by Gerard Meszaros in his book xUnit Test Patterns: Refactoring Test Code (Addison-Wesley, 2007) as one of the ways to battle code coupling, but it has a much broader application.
* We often find that code is hard to test because it’s coupled to a framework dependency (see figure 7.3). Examples include asynchronous or multi-threaded execution, user interfaces, communication with out-of-process dependencies, and so on.
  * To bring the logic of this code under test, you need to extract a testable part out of it. As a result, the code becomes a thin, humble wrapper around that testable part: it glues the hard-to-test dependency and the newly extracted component together, but itself contains little or no logic and thus doesn’t need to be tested (figure 7.4).
  * hexagonal architecture advocates for the separation of business logic and communications with out-of-process dependencies. This is what the domain and application services layers are responsible for, respectively.
  * Functional architecture goes even further and separates business logic from com- munications with all collaborators, not just out-of-process ones.
* Another way to view the Humble Object pattern is as a means to adhere to the Single Responsibility principle, which states that each class should have only a single responsibility.
    *  One such responsibility is always business logic; the pattern can be applied to segregate that logic from pretty much anything.
* In our particular situation, we are interested in the separation of business logic and orchestration. You can think of these two responsibilities in terms of code depth versus code width. Your code can be either deep (complex or important) or wide (work with many collaborators), but never both (figure 7.6).
  * You already saw the relationship between this pattern and hexagonal and functional architectures. 
  * Other examples include the Model-View-Presenter (MVP) and the Model-View-Controller (MVC) patterns. 
    * These two patterns help you decouple business logic (the Model part), UI concerns (the View), and the coordination between them (Presenter or Controller). 
    * The Presenter and Controller components are humble objects: they glue the view and the model together.
  * Another example is the Aggregate pattern from Domain-Driven Design.2 One of its goals is to reduce connectivity between classes by grouping them into clusters— aggregates.
    * The classes are highly connected inside those clusters, but the clusters them- selves are loosely coupled.
    * Note that improved testability is not the only reason to maintain the separation between business logic and orchestration. Such a separation also helps tackle code complexity, which is crucial for project growth, too, especially in the long run. 

## 7.2 Refactoring toward valuable unit tests
### 7.2.1 Introducing a customer management system
### 7.2.2 Take 1: Making implicit dependencies explicit
### 7.2.3 Take 2: Introducing an application services layer
### 7.2.4 Take 3: Removing complexity from the application service
### 7.2.5 Take 4: Introducing a new Company class

## 7.3 Analysis of optimal unit test coverage
### 7.3.1 Testing the domain layer and utility code
### 7.3.2 Testing the code from the other three quadrants
### 7.3.3 Should you test preconditions?
* There’s no hard rule here, but the general guideline I recommend is to test all preconditions that have domain significance. 

## 7.4 Handling conditional logic in controllers
* The separation between business logic and orchestration works best when a busi- ness operation has three distinct stages:
  * Retrieving data from storage
  * Executing business logic
  * Persisting data back to the storage (figure 7.10)
* There are a lot of situations where these stages aren’t as clearcut, though. As we discussed in chapter 6, you might need to query additional data from an out-of-process depen- dency based on an intermediate result of the decision-making process (figure 7.11).
  * As also discussed in the previous chapter, you have three options in such a situation:
    * `Push all external reads and writes to the edges anyway.` This approach preserves the read-decide-act structure but concedes performance: the controller will call out-of-process dependencies even when there’s no need for that.
    * `Inject the out-of-process dependencies into the domain model` and allow the business logic to directly decide when to call those dependencies.
    * `Split the decision-making process into more granular steps` and have the controller act on each of those steps separately.
  * The challenge is to balance the following three attributes:
    * `Domain model testability`, which is a function of the number and type of collaborators in domain classes
    * `Controller simplicity`, which depends on the presence of decision-making (branching) points in the controller
    * `Performance`, as defined by the number of calls to out-of-process dependencies
  * Each option only gives you two out of the three attributes (figure 7.12):
    * `Pushing all external reads and writes to the edges of a business operation`—Preserves controller simplicity and keeps the domain model isolated from out-of-process dependencies (thus allowing it to remain testable) but concedes performance.
    * `Injecting out-of-process dependencies into the domain model`—Keeps performance and the controller’s simplicity intact but damages domain model testability.
    * `Splitting the decision-making process into more granular steps`—Helps with both performance and domain model testability but concedes controller simplicity. You’ll need to introduce decision-making points in the controller in order to manage these granular steps.  
  * In most software projects, performance is important, so the first approach (pushing external reads and writes to the edges of a business operation) is out of the question.  
  * The second option (injecting out-of-process dependencies into the domain model) brings most of your code into the overcomplicated quadrant on the types-of-code dia- gram.
  * That leaves you with the third option: splitting the decision-making process into smaller steps. With this approach, you will have to make your controllers more com- plex, which will also push them closer to the overcomplicated quadrant.

### 7.4.1 Using the CanExecute/Execute pattern
* The first way to mitigate the growth of the controllers’ complexity is to use the `Can-Execute/Execute pattern`, which helps avoid leaking of business logic from the domain model to controllers.
  ```
  // Listing 7.10 Changing an email using the CanExecute/Execute pattern
  public string CanChangeEmail() {
      if (IsEmailConfirmed)
          return "Can't change a confirmed email";
      return null;
  }

  public void ChangeEmail(string newEmail, Company company) {
      Precondition.Requires(CanChangeEmail() == null);
      /* the rest of the method */
  }
  ```
  * This approach provides two important benefits:
    * The controller no longer needs to know anything about the process of changing emails. All it needs to do is call the CanChangeEmail() method to see if the operation can be done. Notice that this method can contain multiple valida- tions, all encapsulated away from the controller.
    * The additional precondition in ChangeEmail() guarantees that the email won’t ever be changed without checking for the confirmation first.

### 7.4.2 Using domain events to track changes in the domain model
* you can track important changes in the domain model and then convert those changes into calls to out-of-process dependencies after the business operation is complete. `Domain events` help you implement such tracking.
  * A domain event describes an event in the application that is mean- ingful to domain experts. The meaningfulness for domain experts is what differentiates domain events from regular events (such as button clicks). Domain events are often used to inform external applications about import- ant changes that have happened in your system.

# Chapter 8 Why integration testing?

##
### 
### 
### 

##
### 
### 
### 
### 

## Reference
* https://site.mockito.org/
* [manning - unit testing](https://freecontent.manning.com/what-is-a-unit-test-part-2-classical-vs-london-schools/)
* [detroit vs london school](https://medium.com/@adrianbooth/test-driven-development-wars-detroit-vs-london-classicist-vs-mockist-9956c78ae95f)
* [mocksArentStubs](https://martinfowler.com/articles/mocksArentStubs.html)
* [@SpyBean @MockBean 의도적으로 사용하지 않기](https://jojoldu.tistory.com/320)
* [TDD에 대한 몇 가지 질문](https://brunch.co.kr/@cleancode/44)
* [1. 테스트하기 좋은 코드 - 테스트하기 어려운 코드](https://jojoldu.tistory.com/674)
  * [Testing, Oh my! by Jin-Wook Chung](https://jwchung.github.io/testing-oh-my)
  * [스프링캠프 2019 [Track 2 Session 3] : 무엇을 테스트할 것인가? 어떻게 테스트할 것인가? (권용근)](https://www.youtube.com/watch?v=YdtknE_yPk4)
* [HumbleObject](https://martinfowler.com/bliki/HumbleObject.html)
* [TDD is dead](https://dhh.dk/2014/tdd-is-dead-long-live-testing.html)

* https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices#characteristics-of-a-good-unit-test
* https://github.com/jojoldu/review/blob/master/OKKY_TDD/README.md
https://github.com/msbaek/atdd-example
* [백명석 clean coders](https://www.youtube.com/playlist?list=PLeQ0NTYUDTmMM71Jn1scbEYdLFHz5ZqFA)

---
tdd/bdd
test (unit test vs integration test , system test , endpoint integration test)
unit testing
integration testing
  transaction
  concurrency
end-to-end testing

test fixture
test double

what to test?
  everthing (clean coder) vs domain-heart vs test-reflief
```
// You Must Know It Works, Clean Coder, p.46

How much of the code should be tested with these automated unit tests? Do
I really need to answer that question? All of it! All. Of. It.

Am I suggesting 100% test coverage? No, I’m not suggesting it. I’m demanding it.
Every single line of code that you write should be tested. Period.

Isn’t that unrealistic? Of course not. You only write code because you expect it
to get executed. If you expect it to get executed, you ought to know that it
works. The only way to know this is to test it.
```
does this (test) code hurts?
testing test codes

spring annotations
  * https://reflectoring.io/spring-boot-test/
  * https://dev.to/stealthmusic/why-you-should-start-using-junit-5-5gk2
  @SpringBootTest
  @WebMvcTest
  @MockMvc
  @Nested
  ...

boundaryb
testAny

---
Practical Testing in Spring
