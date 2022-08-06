# Tutorial: Build Modern Cross-Platform Apps with .NET

[1. Introduction](README.md)  
[2. Avalonia UI](2_Avalonia.md)  
[3. Dependency Injection](3_DependencyInjection.md)  
[4. MVVM Design](4_MVVM.md)  
[5. Dialogs and Tools](5_DialogsTools.md)  
[6. Unit Testing](6_UnitTesting.md)  
[7. Reactive](7_Reactive.md)  
[8. Deployment](8_Deployment.md)

## 6. Unit Testing

Just like MVVM ravioli code is longer to write but requires less changes and is more expandable, unit testing can double your programming time but cut down debugging by 10x. If you've ever tried to estimate your time on a project, whatever time you need has to be multiplied by 4-10x because of testing and debugging. While Unit Testing can represent 50% of your coding time, it can drastically reduce debugging time while greatly increasing the code quality. You would be surprised: you think your code is great, but unit tests rarely pass on the first try!

On a personal note, it has been difficult to adapt my mind to the unit test mindset. Some say that it takes about a year to get used to it. It's all about mindset. If you structured your code correctly from the start, then it makes things a lot easier. Whether or not you want to create unit tests now, do read this section to understand how to structure your code to allow it.

Here are situations I faced where unit tests really shine:

- I created a wrapper to a complex online API ([OntraportApi.Net](https://github.com/mysteryx93/OntraportApi.NET)). First, their services can change and introduce bugs or different behaviors at any time, it allows me to detect problems. Morever, I refactored the code. It allows me to know that the public API maintains the same behaviors! Tests rarely work on first try; there were a lot of broken details to fix. Imagine trying to fix the API manually after refactoring? It would leave tons of functions untested and broken.

- I created a recurring payment processing system. There are like 100 different scenarios that can occur! How the heck would you test that manually? There is a single method that gets called by a webhook when a payment comes in; you need to test that single method in a hundred different ways. I can't imagine doing that manually.

Some sitations where unit testing is harder: user controls and extensions methods. I do not recommend starting to learn with those. I recommend starting by testing some back-end services.

### How to Code with Unit Tests

#### 1. Tests First

As counter-intuitive as it sounds, the recommended approach is to write your tests first and the code second, meaning that all your tests will fail at first.

1. Create your public interfaces
2. Write down a list of all scenarios that need to be tested
3. Start writing your tests with the expected behaviors by calling the interfaces
4. Write your code until all tests pass
5. Enjoy a service that just "works" as expected no matter what you feed in!

At this point, you can plug the service to the UI and know that it just works. That's one thing less to worry about as you design your UI!

#### 2. Fixing Code

When fixing bugs, a good approach is to write a test that reproduces the problem and fails. Fix the code until it passes. You'll then be assured that the bug will never happen again!

#### 3. Code First

The third approach is to write the code first and test it afterwards. Sometimes I take the tests-first approach, and sometimes I take the code-first approach.

If you have an existing code-base that you want to add unit tests for, it can be very difficult as the code is not designed for it. You have to consider whether it will be worth it. It will probably need to be part of a refactoring project.

### Unit Tests vs Integration Tests

There are 2 types of tests to write.

Unit Tests mean that you run quick tests in a virtual environment, where all inputs and outputs are controlled with mocks. These should run very quickly so that you can run them as part of your build process if desired.

Integration Tests mean that you run your code in the real world: with real files and/or with real web services.

If you build a car, you could say that unit testing is running the engine in a garage to ensure it behaves. Integration test is driving the car on the highway. In both cases, you may have a hundred different tests to perform. How should the brakes behave when turning in the snow? Do the new brakes perform as well in the same circumstance?

### Mocks

A Mock is a fake or altered implementation of an interface. For example, to test incomming payments, `IDateTimeService.Now` provides the current time. You cannot use the real time for your tests, as you want all variable to be predictable. What happens if the account is past due? If it was already paid? If it pays for last invoice instead of the latest one?

`IDateTimeService` needs to be implemented in a way that it provides the date you need for your tests. You also need to mock the data that is in your database. All points of contact (inputs and outputs) need to be replaced with mocks.

If a class has complex logic and also relies on some input or output, such as Windows API calls, you can encapsulate the external calls into a new class to isolate the I/O. The new class containing Windows API will only be tested with integration tests.

There are also various types of mocks: Mock and Fake. A Mock is a class implementing the interface and performing altered behaviors. A Fake is a drop-in implementation that doesn't do anything. For example, if testing against a web service, a Mock will simulate the work, return the results, and track your calls. A Fake will just give you the thumbs up.

### Implementing Mocks

There are three ways to implement mocks: manual implementation, Moq, or hybrid.

Manual: You create an interface implementation that suits your needs. This is good for behaviors that need to be re-used often. A great example is [System.IO.Abstractions.TestingHelpers](https://github.com/TestableIO/System.IO.Abstractions/tree/main/src/System.IO.Abstractions.TestingHelpers). It allows you to use a virtual file system without accessing real files.

[Moq](https://github.com/moq/moq): This tool allows creating automatic implementations of interfaces, and configuring its behavior. That way, you can manually configure what each method of `IDateTimeService` will return. It also tracks all the calls, so that you can validate how many times a specific method was called. See Moq documentation for how to use it.

Hybrid: You create a manual interface implementation with virtual members, and create a Moq around it. That way, you can inherits standard behaviors, and customize it and track calls for specific tests. Important: Moq can only alter methods if they are declared as `virtual`. Also note that to use the class' own code, you must set `CallBase` like this: `var mock = new Mock<FakeFileSystemService>() { CallBase = true };`.

As a personal convention, I prefix manual mocks with "Fake" (ex: FakeFileSystemService), and I prefix Moq instances with "mock" (ex: mockFileSystemService).

### Getting Started

Create a new [xUnit](https://xunit.net/) project. Give it the same name as your project with ".Tests" sufix. There should be one such project for each project. For integration tests, you can either create a project ending with ".IntegrationTests", or add a folder called "Integration" within your project.

Update NuGet packages, add Moq, and add a reference to your project. Also make sure that `xunit.runner.visualstudio` is in your project otherwise you will not be able to run the tests!

Create your GlobalUsings.cs file with commonly-used namespaces.

Using this `TestsBase` class can make your life easier.

```c#
public class TestsBase
{
    protected ITestOutputHelper Output { get; }

    public TestsBase(ITestOutputHelper output) => Output = output;

    /// <summary>
    /// Allows using a lambda expression after ??= operator.
    /// </summary>
    protected T Init<T>(Func<T> func) => func();

    /// <summary>
    /// Initializes a mock of specified type.
    /// </summary>
    /// <param name="action">A lambda expression to configure the mock.</param>
    /// <typeparam name="T">The type of mock to create.</typeparam>
    /// <returns>The newly-created mock.</returns>
    protected Mock<T> InitMock<T>(Action<Mock<T>> action)
        where T : class
    {
        var mock = new Mock<T>();
        action(mock);
        return mock;
    } 
}
```

Create folders in your test project with the same strucure as your project folders. Here is how I will start the file `ViewModels\MainViewModelTests`.

I haven't seen anyone that suggests this method of managing dependencies, but that's how I do it. When you got 20 dependencies referencing each other, this makes it a lot more manageable. You don't need to care about what depends on what, or what's needs to be initialized for each test.

I call the main class to be tested as `Model`.

Note: You cannot initialize mocks with inline lamba expressions on the right-side of `??=`, but you can work around that limitation using `Init` and `InitMock` methods!

You can now write debug data to `Output`.

```c#
public class MainViewModelTests : TestsBase
{
    public MainViewModelTests(ITestOutputHelper output) : base(output) { }

    public MainViewModel Model => _model ??=
        new MainViewModel(MockSettingsProvider, Encoder, DialogService, FileSystem,
            FileLocator, MockAppPath.Object, new FakeEnvironmentService(), MockPitchDetector.Object);
    private MainViewModel _model;

    public IEncoderService Encoder =>
        _encoder ??= new EncoderService(FileSystem, DialogService, MockAudioEncoder.Object, new FakeDispatcher());
    private IEncoderService _encoder;
    
    public Mock<IAudioEncoder> MockAudioEncoder => _mockAudioEncoder ??= new Mock<IAudioEncoder>();
    private Mock<IAudioEncoder> _mockAudioEncoder;

    public Mock<IAppPathService> MockAppPath => _mockAppPath ??= InitMock<IAppPathService>(mock =>
    {
        mock.Setup(x => x.AudioExtensions).Returns(new[] { "mp3" });
    });
    private Mock<IAppPathService> _mockAppPath;
    private Mock<IAppPathService> _mockAppPath;
...
}
```

You can then write your tests

```c#
[Fact]
public void FileExistsAction_SetByIndex_SetOnEncoder()
{
    Model.FileExistsActionList.CurrentPosition = 1;

    Assert.Equal(Model.FileExistsActionList.Source[1].Value, Model.Encoder.FileExistsAction);
}

[Theory]
[InlineData(EncodeFormat.Flac, false)]
[InlineData(EncodeFormat.Mp3, true)]
[InlineData(EncodeFormat.Opus, true)]
public void IsBitrateVisible_SetFormat_IsExpectedValue(EncodeFormat format, bool value)
{
    Model.FormatsList.SelectedValue = format;

    Assert.Equal(Model.IsBitrateVisible, value);
}

[Fact]
public async Task Start_NoPath_ThrowsArgumentException()
{
    // Set
    var file = new ProcessingItem("", "") { Destination = "Dest1.mp3" };

    // Act
    var t1 = Model.StartAsync(file, Settings);

    // Verify
    await Assert.ThrowsAsync<ArgumentException>(() => t1);
}
```

### Unit Test Structure

First you have the attribute: Fact or Theory. Fact means that you run a single method. Theory means that you'll pass different sets of parameters to run multiple tests.

Then, you have to name your test. The general convention is: Method_Action_Result. Don't worry about being verbose. When you look at the test name, you should know what it's doing and what it expects.

Normally, the test should test for one specific thing. If you find yourself testing many different things, you can split it into multiple tests. Some argue that the test should test a single thing, but it's OK to check that the overall state is what you would expect, which can often involve a few checks. Everybody got their own rules, but at the end of the day, use your common-sense.

The body of the test comes into 3 sections: Set - Act - Verify. In my sample above, Set is implicitly done by calling Model. Generally, more work needs to be done to prepare for the test.

Set: Configures the state of the mocks and classes in preparation for the tests. Note that all classes are destroyed and recreated between tests.

Act: Run the test.

Verify: Use xunit `Assert` methods to verify the expected result, resulting in a test success or failure.

### Resources

First, read [xUnit documentation](https://xunit.net/), then read [Moq documentation](https://github.com/moq/moq4). Both xunit and Moq have extensive coverage on [StackOverflow](https://stackoverflow.com/), so you can find solution to most problems there.

[> Next: Reactive](7_Reactive.md)
