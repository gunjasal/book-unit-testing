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

* ref
  * [manning](https://freecontent.manning.com/what-is-a-unit-test-part-2-classical-vs-london-schools/)
  * [detroit vs london](https://medium.com/@adrianbooth/test-driven-development-wars-detroit-vs-london-classicist-vs-mockist-9956c78ae95f)
  * [mocksArentStubs](https://martinfowler.com/articles/mocksArentStubs.html)
  * [@SpyBean @MockBean 의도적으로 사용하지 않기](https://jojoldu.tistory.com/320)
  * [TDD에 대한 몇 가지 질문](https://brunch.co.kr/@cleancode/44)

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
