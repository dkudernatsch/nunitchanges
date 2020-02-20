## New features in Nunit3

### [Parallel tests](https://github.com/nunit/docs/wiki/Parallelizable-Attribute)

### [Assert.Multiple](https://github.com/nunit/docs/wiki/Multiple-Asserts)

#### File attachments

```csharp
[Test]
public void CheckBrianPromotionToCEO()
{
    // test code (preparation, assertions, etc.)
    // put your logic here: save log or take a screenshot
    TestContext.AddTestAttachment(@"D:\myFile.txt");
}
```

```xml
<test-case id="0-1001" name="AddAttachment"  ... >
    <attachments>
        <attachment>
            <filePath>D:\myFile.txt</filePath>
        </attachment>
    </attachments>
</test-case>
```