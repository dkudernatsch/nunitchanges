
# nunit2 to nunit3 changes

_for an official list head over to [nunit braking changes](https://github.com/nunit/docs/wiki/Breaking-Changes)_

## Major breaking changes


### Expected Exception Attribute

The expected exception attribute has been removed and is no longer supported.
The replacement syntax is `Assert.Throws` or `Asser.That(..., Throws...)`.

```csharp
// old expected exception attribute
  [Test]
  [ExpectedException(typeof(TestException), ExpectedMessage =  "Exception message")]
  public void Test()
  {  
   control.SelectOption().WithIndex (1); 
  }
// new assert throws assert
  [Test]
  public void Test()
  {  
    Assert.That (
        () => control.SelectOption().WithIndex (1), 
        Throws.InstanceOf<TestException>()
            .Message.EqualTo ("Exception message");
  }    

```

### TestFixture and SetUpFixture Lifecycle changes

The setup and teardown attributes on test fixtures and setup fixtures changed. In nunit2
the SetUpAttribute was both used for per test setup in test fixtures but also for one time setup methods in SetUpFixtures.

Nunit3 introduces new `[OneTimeSetUp]` and `[OneTimeTearDown]` attributes. They replace the normal `[SetUp]` and `[TearDown]` attributes in SetUpFixtures 
and the now deprecated `[TestFixtureSetUp]` and `[TestFixtureTearDown]`.

To sum up if you want to run a setup/teardown per testcase, use `[SetUp]`/`[TearDown]`. If a setup/teardown should only be run once, use `[OneTimeSetUp]`/`[OneTimeTearDown]`.


|     | TestFixture | SetupFixture |
| --- | --- | --- |
| OneTimeSetUp | ✔️ | ✔️ |
| OneTimeTearDown | ✔️ | ✔️ |
| TestFixtureSetUp | ⚠️ | ❌ |
| TestFixtureTearDown | ⚠️ | ❌ |
| SetUp | ✔️ | ❌ |
| TearDown | ✔️ | ❌ |

__⚠️: Deprecated__  
__❌: Error__

##### Example SetUp Fixture
```csharp
// nunit2 SetUpFixture
[SetUpFixture]
public class SetUpFixture {
    [SetUp]
    public void Setup() {}

    [TearDown]
    public void TearDown() {}   
} 

// nunit3 SetUpFixture
[SetUpFixture]
public class SetUpFixture {
    [OneTimeSetUp]
    public void Setup() {}

    [OneTimeTearDown]
    public void TearDown() {}
} 

``` 
##### Example TestFixture Fixture

```csharp
// nunit3 TestFixture
[TestFixture]
public class SomeExampleTest {

    [OneTimeSetUp] // was [TestFixtureSetUp] in nunit2
    public void FixtureSetup() {
        //called exactly once befor any tests in this fixture
    }

    [OneTimeTearDown] // was [TestFixtureTearDown] in nunit2
    public void FixtureTearDown() 
    {
        //called exactly once after all test in this fixture ran
    }

    [SetUp]
    public void TestSetUp() {
        //called before every test
    }

    [SetUp]
    public void TestTearDown() 
    {
        //called after every test
    }

} 
```
### SetUp and TearDown order

Generally the order in which SetUp and TearDown methods are called has not been changes, 
however ordering changed if an exception happens during setup.


##### Normal exceution:

```    
    BaseSetUp -> 
        DerivedSetup -> 
            DerivedDerivedSetup -> 
                Test ()
            DerivedDerivedTearDown -> 
        DerivedTearDown -> 
    TearDown
```    
##### nunit2 with exception:
```
    BaseSetUp -> 
        DerivedSetUp (*throws*) -> 
    
                DerivedDerivedTearDown -> 
            DerivedTearDown -> 
    TearDown
```
##### nunit3 with exception:
```
    BaseSetUp -> 
        DerivedSetUp (*throws*) -> 
        DerivedTearDown -> 
    TearDown
```

### Current Directory

Nunit3 executes all tests relative to a newly created tempoary folder. This breaks previous test, where files are accessed relative to the test assembly in nunit2.
The current test assembly path can be accessed using `TestContext.CurrentContext.TestDirectory`.

This makes it easier to create tempoary files, that are only used during a testrun, 
but break all existing tests which loaded resources relative to the test assembly.

#### Example

```csharp
    // nunit2 behavior
    public void Test() 
    {
        // load a template file relative to the test assembly
        var templateFile = File.Open ("testresources/template1.txt");
        // create a file in a tempoary location
        var tmpCopy = File.Create (GetTempPath() + "tmpfile.txt");
        ...
    }

    // nunit3 behavior
    public void Test() 
    {
        // load a template file relative to the test assembly
        var templateFile = File.Open (TestContext.CurrentContext.TestDirectory + "/testresources/template1.txt");
        var tmpCopy = File.Create ("tmpfile.txt");
        ...
    }

```

