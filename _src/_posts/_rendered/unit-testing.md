---
date: 01/30/2020
title: Android - Unit Testing for Dummies
description: Read about how you can start your unit test journey and become a better developer.
tags: ['android', 'unittesting','testing', 'mockito']
---

### What is Unit Testing?
- In its simplest term, unit testing is testing a small piece of code.
- A unit test usually targets a single method that perform a single task.
- Test ensures that the code is behaving as expected and that code has a deterministic outcome.

### When the Unit Tests are written/run?
There are generally 2 approaches when writing unit tests,

- One is test-driven-development, in which unit tests are written first, these tests specify requirements and at first they should fail because there is no code to produce required output, the code is then written around them to meet requirements and make tests pass.
- Other approach is retrospective testing, that is, after writing code, you write unit tests. This is more commonly used approach in practice (if there are unit tests written at all). You simply write code and then write unit tests after the code and validate code behaviours.

With both approaches, it is a good practice to run unit tests as often as you can, after every code change, this will result in early identification of flaws in the system.

### Who writes Unit Tests?
- Developers. Period.

### Why should you write Unit Tests?
- Unit tests validate small pieces code for expected outcomes and report early issues.
- Unit tests also make sure that the code is giving errors where expected, this way a lot of corner cases are covered.
- Allows identification of logical bugs in the code when they are covering edge cases.
- If unit tests are written regularly, they make code resist to changes that breaks the code. If a change in the code fails the unit test, either the code has issue or test is not updated, both ways, its a warning sign for developer to put an extra effort to investigate the issue before closing the development cycle.
- It seems waste of time and extra effort at the beginning, and it is, but few days effort of maintaining the code with meaningful unit tests saves months of refactoring later.
- It makes code cleaner – since the tests require certain level of abstraction and loose coupling which make code more cleaner.

#### Intention
- The purpose of this writing is to promote the unit tests in software projects, specially those which are shaping the way we live, these projects can be related to finance, medicine, transport or online shopping.
- The bugs or problems in these software causes both customers and developers to lose their time and money.
- Promoting unit tests can save a lot of time in covering basic business logic validations and resulting a reliable software which can be fruitful for all the entities in the business.
- Writing unit test for the sake of unit tests or say achieving high code coverage is not a good idea. If the tests are not playing their part in verifying business flows, they are of no value.
- In programming, every line of code generates a business value, no one likes to lose business, unit tests verify that the business value that code aims to provide is on the spot.

#### Writing Testable Code – A prerequisite (IMPORTANT)
- Unit tests are written in controlled environment, this helps you provide certain conditions under which you expect certain outcomes. If you cannot control the agents or entities that are driving these conditions you cannot be sure about the outputs. This test environment is largely controlled using dependencies you provide to the test case.
- First and foremost thing is separating the dependencies from the code. For this you should be able to recognize what are the dependencies and what is the actual code that needs to be tested.
- In terms of Java and Android, if you are working in Android, your business layer should be testable without any Android dependencies.
- Continuing on Android as example, for the business layer (and unit tests themselves), most of the Android framework stuff will be dependencies for code. Like network library, Retrofit.
- Considering framework elements as dependencies is helpful but you still need a layer of abstraction so you can drive these framework objects the way you want to, consider following:

Instead of:
```
public UserService(User user, Retrofit retrofit)
```

Use some abstraction:
```
public UserService(User user, Request userRequest) 
```

Using an abstraction layer of Request class to wrap retrofit in it so the code doesn’t depend on Android Retrofit.

- Creation of objects should be delegated to upper level, like dependency manager. If a class that needs to be tested is creating its own objects inside, you cannot control data in those objects to test.

Instead of using “new” keywords (constructors/initializers) in code:
```
public UserService(User user){
      request = new Request();
      ...
    }
```

Use dependency injection:
```
public UserService(User user, Request userRequest) 
```

You have now control over creation. You can also use dependency managers like Dagger2 and “inject” this dependency here.

- Another important thing to mention here is composition over inheritance will always be in favour of testable code. Since composition gives you more control over objects when you are using dependency injection, you can easily control these objects in test environment. In above example UserService, you can provide a UserValidator instead of implementing it. Just like UserRepo and Request, it can follow composition. But the abstraction is still maintained since no concrete references are used here.

Instead of:
```
public class UserService implements UserValidator{
...
}
```

You can do:
```
// constructor injection
public UserService(User user, Request userRequest, UserValidator validator) 
```

- Separate creation logic from business logic, that’s what dependency managers do, if this is not the case, you might miss some business logic to test which was ran during creation of objects. Since tests and business classes should not be responsible for creating objects involved in a business logic flow.

Avoid writing business logic in constructor or anywhere in creation classes like dependency managers:
```
  public UserService(User user, Request userRequest){
      if(userRequest !=null){
        this.userRequest= userRequest;   // This block is never tested since it is just creation block
      }
    }
```

- Unit tests usually targets business layer, so regardless of any framework you are, your code that needs to be tested must be using the core language libraries. A good way to realize this in Java/Kotlin is to look at imports section. If there is an import that is not coming from core Java/Kotlin it should raise alarm.
```
import android.os.Bundle; // --- This is not good in business layer
import android.view.View; // --- This is not good in business layer
import com.selfsol.app.models.user; // --- This is okay as it is a POJO model
```
First priority should be to avoid such dependencies, if not possible their creation should be delegated to dependency managers with proper abstractions. So when this class is tested, these dependencies can easily be mocked with dummy values or behavior and unit tests are not blocked.

- Its nice to have to follow an architectural pattern to manage code, whether it be MVVP, MVP, VIPER or any other, primary goal is to make code clean, maintainable and testable, which makes it helpful to work with such codebase in the longer run.
- The implementation of architecture doesn’t really provide all the solutions, so there are still few things that needs to be decoupled, like networking and UI stuff.

Following example illustrates how this abstraction can be achieved and how it is created in dependency manager layer:
```
// class for abstraction of network library
public interface Request   {
    void execute( T data);
}

// class having real network classes and implementation
public class UserRequest implements Request{
    ...
}

// creation of this class
public class SomeDependencyProvider(){
    // notice interface reference in return type
    public Request createRequestObject(){
        return new UserRequest();
    }
}
```

Generally, you don’t need network or UI when writing unit tests, so if there is a good abstraction layer between the business logic layers and UI or Network etc, it will help you mock these entities and validate the actual business logic in tests.

- Static methods makes it hard to test unless they are pure methods and do not use or create any dependency objects in their execution, moreover these static methods or objects cannot be mocked easily which are blocker in unit testing.

Consider following class in which one method is acceptable but the other one creates trouble:
```
public class Utils{
    // this wont be any trouble
    public static boolean isEmptyString( String string){
        return string !=null && !string.isEmpty();
    }
    // this will cause issue since this is using an object that is in its static scope and cannot be mocked
    public static Date formatDate(String pattern, long timeInMillis){
        DateFormatter formatter = new DateFormatter( pattern );
        return formatter.format( timeInMillis );
    }
    
}
```

In other words, static classes cant be used with dependency injections so they lack this flexibility of being separating creation logic from actual business logic. This brings you to the point number 1, where you are unable to control the test environment.

- Tests are written for public methods. Because other classes or layers call public methods of classes that provide business logic. These methods in turn, call private methods which itself covers these private methods. DON’T make private methods public for just writing TESTS.
```
public class UserService{

    public boolean validateUser(User user){
        validateAge(age);
    }
    
    private boolean validateAge(int age){
    }
}
```

In above example, you will write test for validateUser(), which will also cover validateAge(), if you do want to test such methods individually you can introduce an interface that contains these signatures, in this way these methods will hide their implementation details and yet they will be accessible for testing from outside or mocking if required.

- When writing a test for a class, the target class should be instantiated as a concrete implementation, so the real code is ran, and all other participating objects are mocked. If the participating objects have abstraction (they are interface objects), mocking becomes easy.

Consider writing test for UserService:
```
@RunWith(MockitoJUnitRunner.class)
public class UserServiceTest {
    // Class reference
    private UserService userService;
    
    // interface reference mocked
    @Mock
    Request request;

    // interface reference mocked
    @Mock
    UserRepo userRepo;
...
}
```

You can later define how “request” behaves when its methods are called, same for “userRepo”. Point is, making them abstract, decoupled and using dependency injection pattern, it is easier to control these participating objects and you can design your test by providing values and behaviours the way you like to.

### Writing Unit Tests
In the following discussion I will be using Java as the example language and some references from Android Framework. Regardless of this fact, the general guidelines and practices will apply to any OOP based framework.

- You will use Java and its testing framework JUnit in these examples, moreover, you will refer Mockito for mocking objects.
- You will focus on UserService (or UserManager) example, in which, you will validate creation of valid users in a system.
- You will not test classes that serves as dependencies in the UserService, these are UI classes referred as “View” in most of the architectures and Network classes like Request or Response. You will mock these dependencies.
- It is not your responsibility to test 3rd Party library classes like Retrofit, Gson, Picasso, OkHttp etc.
- Meaningful names of test methods should be used, these names can be longer as long as they are self explanatory.
- AAA Testing Pattern – Arrange, Act, Assert is the pattern you will use and it goes as follows:
    - Arrange: Setup objects, parameters and environment for the test
    - Act: Perform action, the actual code is ran on which test is to be done
    - Assert: Assertion or validation is made on the action that was performed if it is giving required results or not

See [UserServiceTest](https://github.com/talhahasanzia/unit-testing-cheatsheet/blob/master/UserServiceTest.java) for details.

#### Arrange
Set dummy inputs for data before executing actual test call. If there are any methods to be mocked, they are also done in this phase.

Lets declare objects first:
```
   private UserService userService;

   @Mock
   Request request;

   @Mock
   UserRepo userRepo;

```
How mocks work?
```
   // MockitoJunitRunner initializes mocks itself when tests starts
   @RunWith(MockitoJUnitRunner.class)
   public class UserServiceTest { ...
```
Setting up the Test Class:
```
userService = new UserService(request, userRepo); // in @Before method
```

The setup seems easy and simple because you already followed the practices in the code that are test friendly, here using injection pattern instead of creating dependencies in the object itself. You were able to provide UserRepo and Request from outside based on your requirement here.

Setting up requirement that are specific to the test case:
```
    @Test
    public void testValidAge_inputPositiveNumber_shouldValidateSuccess() {
        // Arrange
        val testAge = 18
        ...
    }
```

#### Act
Now you perform your test:

```
    @Test
    public void testValidAge_inputPositiveNumber_shouldValidateSuccess() {
        ...
        // Act
        boolean result = userService.validateAge(testAge);
        ...
    }
```

This is how you right test in JUnit. Notice that the function name is like a sentence that tells the whole story. In source code, this convention may not be liked in general, but in tests, this is preferred. It explains a lot about the test. Since tests dont have that amount of documentation that code has (if any), this is important. The convention goes like this:

```
testMethod_conditions_outcomes()
```

Then the "act" happens. The method is called and the value is captured.

#### Assert
```
    @Test
    public void testValidAge_inputPositiveNumber_shouldValidateSuccess() {
        // Arrange
        ...
        // Act
        ...
        // Assert
        assertTrue("Expected to validate positive number as valid age but failed", result);
    } 
```

The value captured in the act is asserted to be true or false. The assertTrue is a JUnit assertion method which checks the given condition along with option to provide a message in case of failure. Let’s not go into implementation at this point just the approach. All the implementations are left blank.

There is a message Expected to validate positive number as valid age but failed passed as 1st parameter. This is failure message. It is important to flag failures with meaningful messages. So when someone come across a failed test case in the log, s/he can identify what is the problem. This is also a good practice.

#### But wait, your code is not that simple!
In most of the cases, this assert flow works in tests. But sometimes the methods don’t return values. Like in `VIPER` which uses callback pattern. In this case view is holding reference to presenter and a view reference is inside a presenter. `View` calls `presenter.fetchSomething()` this method does not return anything but after it fetches, it calls view.`onSuccessfulFetch()`. How can you test presenter since its methods don’t return anything?


Similarly in case of Request when the `UserService` calls `createUser(User user)`, it doesn’t do anything. It calls `request.execute(user, this)` and return nothing.

There are two things you can do to test the flows or method that dont return anything:

- If the method calls another method on any other object, this method call can be verified using Mockito framework. This is called testing interactions where you cannot test states (asserting values).
- Check if the code runs without any exception. This is not the only thing you should be doing, but in rare cases like these, this is important factor.

Following code piece can illustrate this as:
```
    @Test
    public void testCreateUser_validUserProvided_shouldCallRequestExecuteWithUser(){
        // act
        userService.createUser(user);
        // assert
        Mockito.verify(request).execute(user, userService);

    }
```

One of the coolest thing that this `verify` does is that it matches the parameter with actual call. For example:
```
Mockito.verify(request).execute(user, userService);
```
will pass the test, but
```
Mockito.verify(request).execute(null, userService);
```
will fail, why? Because the actual call used same object as you mentioned. This was your specific case where you knew both parameters, but cases where you don’t know the exact parameters, you can specify any() in parameter to allow any object with any value. You can also narrow down this to specific type calling any(String.class) this will allow objects with only string type but with any value in it.
```
Mockito.verify(request).execute(any(), userService);
```
This is really useful in callback patterns. In simple method call a value is returned. In callbacks, the value is returned in a callback’s success method as parameters. Consider following asynchronous call made for validation earlier:

```
// simple call
boolean result = userService.validateAge(18);
```
And its callback version would look like:
```
userService.validateAge(18, validationCallback);
....
```
and where the validation callback is implemented:
```
...
onValidationComplete(boolean success){
...
}
```
You can see that boolean value that was returned in simple call is now a parameter for callback’s method. Considering same example you can write assertion as:
```
Mockito.verify(validationCallback).onValidationComplete(true);
```

This should work right? No. Since the object is mocked you have to specify that when this asynchronous call is made, you need to invoke onValidationComplete(). If this object was not a mock object, the above assertion along with following mocking would not be possible. Why asynchronous is mentioned before? because going on callback pattern has one main purpose of being non-blocking operations.

So how are you going to mock the behavior of these objects? Mocking a method of an object that is marked as Mock is as easy as:
```
// this should be done in "Arrange" phase, in your case in setup() method
Mockito.when(userValidator.validateId(l363)).thenReturn(true);
```

This will setup userValidator to return true when its validateId() is called with id 1363. Similarly you can design many flows using same approach including multiple happy cases and also failure cases to test if code gives expected output on wrong inputs. All great but this is not applicable for callback/asynchronous methods.

Luckily, in Java, you have Mockito‘s Answer to solve the problem. But you have to make another change. Since you want to mock response, you will have to provide it as dependency. See this is how unit tests can help refactor your code to be more decoupled and resilient. Now coming back to point.

Making Response testable in `UserService`: (Refer to [UserService2](https://github.com/talhahasanzia/unit-testing-guide/blob/master/UserService2.java))

```
public class UserService2 implements UserValidator { // notice it is not implementing Response Callback now

    private Request request;
    private UserRepo repo;
    private Response responseCallback;

    public UserService2(Request request, UserRepo repo, Response response) {
        this.request = request;
        this.repo = repo;
        responseCallback= response;
    }

 ...

}
```
Mocking Callback object so its calls can be tested using `Mockito`: (See [UserService2Test](https://github.com/talhahasanzia/unit-testing-guide/blob/master/UserService2Test.java))
```
@RunWith(MockitoJUnitRunner.class)
public class UserService2Test {

    private UserService2 userService;

    @Mock
    Request request;

    @Mock
    UserRepo userRepo;

    @Mock
    User user;

    @Mock
    private Response response;

    @Before
    public void setup() {
        userService = new UserService2(request, userRepo, response);
    }

    @Test
    public void testCreateUser_validUserProvided_shouldCallRequestExecuteWithUser() {
        // arrange
        mockRequestCallbackResponse();
        // act
        userService.createUser(user);
        // assert
        Mockito.verify(request).execute(user, response); // verify request.execute was called
        Mockito.verify(response).onSuccess(user); // verify callback method was called with parameter specified

    }
    
    // call onSuccess on 2nd argument object as in invocation.getArguments()[1] (0 - first, 1 - second)
    private void mockRequestCallbackResponse() {
        Mockito.doAnswer((Answer) invocation -> {
            Response callback = (Response) invocation.getArguments()[1];
            callback.onSuccess(user);
            return null;
            // when request.execute is called
        }).when(request).execute(user, response);
       
    }

}
```

This is like: When `request.execute(user, response)` is called, call `onSuccess(user)` on response object that was passed. And the parameter checks will work like same as mentioned before.

#### Mockito in Kotlin
Kotlin reflections work little bit different than java, in above examples, You have used java, but it should be same for Kotlin, except one thing, mockito you used above cannot mock Kotlin classes. There is a very useful library available on github as [nhaarman/mockito-kotlin](https://github.com/nhaarman/mockito-kotlin) (Now officially available in `Mockito` as of January 2024). This has pretty much everything you need in Kotlin, so you are good to go!

#### Architecture Components – Can You write unit tests?
Now, you have seen drastic improvements in terms of architecture favoured official libraries from Android Team at Google, most prominently, you see MVVM is pushed as standard way of writing Android apps. It’s very easy to loose the sense of writing SOLID code if you don’t have a good understanding of layered architectures, when you have such libraries as LiveData, Room, ViewModel and the awesome Flow API. The point is, no matter how good or efficient these libraries are, your code must have testing module. This argument is supported with the fact that team at Google has always mentioned libraries supporting tests and showing test samples in conferences and talks throughout releases of AndroidX Libraries. There are a lot of resources to get you started testing your MVVM app. One of my personal favorite is [Codelab](https://codelabs.developers.google.com/codelabs/advanced-android-kotlin-training-testing-basics/index.html?index=..%2F..index#0).

The point is, no matter what architecture or libraries are used, if code is well written there will always be a way to write meaningful unit tests. So don’t get lost in fantasies of modern APIs, in fact do get lost in them, explore, try and break them, but remember the fundamental goal : clean; working; bug free code.

#### Cook your own recipe
There is a fair chance that none of above mentioned architectures or libraries are being used in your project. If you remember at the start, its mentioned that most of the guidelines are independent and would like to point that out here again. Cook your own recipe, take ingredients from clean code and official stable libraries. Make sure whatever architecture or libraries you are using is, in its simplest form, loosely coupled and testable.

#### Key Takeaways
Now you have powerful tools at your disposal to write unit tests for most of the code in your project. To put things in perspective, testing on methods that return anything is done with simple `asserts` that test state of returned values. Testing on methods that don’t return anything is done with `verify`. Although it takes much effort to test methods that return no value, but you can actually test the whole interaction along with testing state of the parameters which are same as returned values most of the time.

Some developers take approach of mocking servers that use Retrofit callbacks and those clients. The goal in unit tests is just to mock this layer and if you can do it in a simpler way with just using `Mockito` in Java why not go for it, although this way needs an abstraction layer but that’s what makes code loosely coupled. This is also applicable to testing network calls, I/O asynchronous calls or any long running operation that business logic involves. As long as dependencies are well managed and code is loosely coupled, you can almost write test in any case!

Feel free to give feedback on what I might have missed or what can be improved in this small piece of writing. Find the source code of example classes [here](https://github.com/talhahasanzia/unit-testing-guide). Thanks for reading!

And remember to follow https://testing.googleblog.com/ for updates on new testing libraries and approaches. And https://android-developers.googleblog.com/ for the latest Android news and blogs.

