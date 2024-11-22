# Test Driven Development (TDD) – C# – Faking the Configuration (part 3 of 5)

This follow up article to [part ii of my TDD learning project](2022-12-29-Test-Driven-Development-with-csharp-part-2-api-calling.md) will cover using the Options pattern (with `IOptions`) to read your configuration as well as User Secrets for sensitive storage of keys.

## Third Test – Configuration

This test will use a code-generated version of an `IOptions` implementation which will return fixed responses (a stub) that, when ran in the normal code, will use dependency injection and the reading of configuration files to populate.

### Test Code

The test code is relatively straightforward but will make use of a couple of custom methods to keep the test readable.

```csharp
[Fact]
public async void ReadValuesFromConfigurationToCallAPI()
{
	// Arrange
	Mock<HttpMessageHandler> httpMessageHandler;
	IHttpClientFactory httpClientFactory = MockFactories.GetMockMessageHandler(Constants.RESULT_LIST_EMPTY, out httpMessageHandler);
	var sut = new BlogPost(httpClientFactory, Constants.GetContentfulConfig());
	string expectedQueryList = "https://api.baseurl.com/spaces/SPACE-ID/environments/ENVIRONMENT/entries?access_token=MY-ACCESS-TOKEN";

	// Act
	await sut.ListAsync();

	// Assert
	httpMessageHandler.Protected().Verify(
		"SendAsync",
		Times.Once(),
		ItExpr.Is<HttpRequestMessage>(entry => entry.RequestUri.AbsoluteUri == expectedQueryList),
		ItExpr.IsAny<CancellationToken>()
		);
}
```

Also note how the expected URL is a complete URL and is a single string (i.e. no path combining, string formatting, etc…). This test is just testing what URL is called and doesn’t care about what might come back.

### Mocked MessageHandler Method

This is used to retrieve the mock used to create the HttpClient’s handler. We retrieve this so that we can verify what URL was called. This method will be added to our MockFactories class.

```csharp
public static IHttpClientFactory GetMockMessageHandler(string returnValue, out Mock<HttpMessageHandler> httpMessageHandler)
{
	httpMessageHandler = new();
	Mock<IHttpClientFactory> httpClientFactory = new();

	httpMessageHandler.Protected().Setup<Task<HttpResponseMessage>>(
			"SendAsync",
			ItExpr.IsAny<HttpRequestMessage>(),
			ItExpr.IsAny<CancellationToken>()
		)
		.ReturnsAsync(new HttpResponseMessage(HttpStatusCode.OK) { Content = new StringContent(returnValue) });

	HttpClient httpClient = new(httpMessageHandler.Object);
	httpClientFactory.Setup(factory => factory.CreateClient(It.IsAny<string>())).Returns(httpClient);

	return httpClientFactory.Object;
}
```

### Models for Config

Next, we need to create some models to store the configuration we’ll be using. Personally, I like to include the needed classes in a single file, named to match the top level class.

```csharp
public class ContentfulConfig
{
    public string? BaseUrl { get; set; }
    public string? SpaceId { get; set; }
    public string? Environment { get; set; }
    public ContentfulConfigApiKeys? ApiKeys { get; set; }
}

public class ContentfulConfigApiKeys
{
    public string? PublishedContent { get; set; }
    public string? PreviewContent { get; set; }
}
```

### Fixed Configuration Values

Next, in the Constants class, create a new method that will create an implementation of IOptions<ContentfulConfig> with fixed values:

```csharp
public static IOptions<ContentfulConfig> GetContentfulConfig()
{
	ContentfulConfig contentfulConfig = new()
	{
		BaseUrl = "https://api.baseurl.com",
		SpaceId = "SPACE-ID",
		Environment = "ENVIRONMENT",
		ApiKeys = new ContentfulConfigApiKeys() { PublishedContent = "MY-ACCESS-TOKEN" }
	};

	return Options.Create(contentfulConfig);
}
```

### Fix Compile Errors

The final changes are relating to the new parameter we’ll need to pass to the constructor. In the new test, you’ll see it’s already included as a second parameter but the BlogPost class doesn’t have an overload for this method yet. As this is a brand new app, we will update the existing constructor’s signature. If this was an established app, a new overload may be a better/safer option.

Update the signature for BlogPost to be:

```csharp
public BlogPost(IHttpClientFactory httpClientFactory, IOptions<ContentfulConfig> config)
```

Finally, update the two original tests to pass in the new config:

```csharp
var sut = new BlogPost(httpClientFactory, Constants.GetContentfulConfig());
```

### Making the Test Green

Three changes are needed to make the test go green. The first is to get the config available throughout the class, the second is in the constructor and the third in the ListAsync method.

Start by creating a private read-only variable called `_config`:

```csharp
private readonly ContentfulConfig _config;
```

And then set the value of this in the constructor (place it above the existing lines of code):

```csharp
_config = config.Value;
```

In the constructor, we’ll change the code from a fixed base address to one read from config by swapping out the BaseAddress assignment from `new Uri("https://cdn.contentful.com")` to `new Uri(_config.BaseUrl ?? "")`.

Next, swap out the URL in the `GetStringAsync` call in `ListAsync` and replace it with the following:

```csharp
$"/spaces/{_config.SpaceId}/environments/{_config.Environment}/entries?access_token={_config.ApiKeys?.PublishedContent}"
```

### Making Application Work

The work done so far allows the tests to run but changes must also be made to the main application.

Firstly, add the following to the appsettings.json in Demo.Tdd.Ui in the root element:

```json
  "Contentful": {
    "BaseUrl": "https://cdn.contentful.com",
    "SpaceId": "b27mtzlwgnec",
    "Environment": "production",
    "ApiKeys": {
      "PublishedContent": "",
      "PreviewContent": ""
    }
  }
```

Notice the blank ApiKey sub-values. These will be store in User Secrets to prevent them getting checked into source control.

To add the user secrets, right click on the Demo.Tdd.Ui project and choose the “Manage User Secrets” option. Paste in the following, populating your keys as indicated.

```json
{
  "Contentful": {
    "ApiKeys": {
      "PublishedContent": "your-published-content-key",
      "PreviewContent": "your-preview-content-key"
    }
  }
}
```

Finally, in Program.cs, we need to set up dependency injection for the configuration. Add the following line below the current `builder.Services...` lines:

```csharp
{
  "Contentful": {
    "ApiKeys": {
      "PublishedContent": "your-published-content-key",
      "PreviewContent": "your-preview-content-key"
    }
  }
}
```

Finally, in Program.cs, we need to set up dependency injection for the configuration. Add the following line below the current builder.Services... lines:

```csharp
builder.Services.Configure<ContentfulConfig>(builder.Configuration.GetSection("Contentful"));
```

You will need to add a project reference from the UI to the Contentful class project for the code to compile.

## Fourth Test – Blog List Component

In the [next blog article](2022-12-31-Test-Driven-Development-with-csharp-part-4-Blazor-Server-with-bUnit), we’ll build a blog list component and test it using bUnit.