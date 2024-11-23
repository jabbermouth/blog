# Test Driven Development (TDD) – C# – Finishing the App (part 5 of 5)

This fifth and final article will add some error handling to the blog list retrieval and also populate the BlogPostEntry object with values from the API.

## Seventh Test – Handle Errors

This is a relatively simple test but does require an additional option creating in our MockFactories class.

### Failing Test
Firstly, let’s create the failing test in Demo.Tdd.Contentful.Tests:

```csharp
[Fact]
public async void ReturnEmptyListIfTimeoutOccurs()
{
	// Arrange
	IHttpClientFactory httpClientFactory = MockFactories.GetTimingOutClient();
	var sut = new BlogPost(httpClientFactory, Constants.GetContentfulConfig());

	// Act
	var blogPosts = await sut.ListAsync();

	// Assert
	blogPosts.Should().NotBeNull();
	blogPosts.Should().BeEmpty();
}
```

We will specifically be throwing a TimeoutException but we’ll write the code to catch any exception when we come to implement it.

### Making the Test Green

To make the test green, we’ll wrap the API call in a try/catch. We’ll also need to move the declaration of posts to outside the try/catch.

```csharp
string? posts = null;

try
{
	posts = await _httpClient.GetStringAsync($"/spaces/{_config.SpaceId}/environments/{_config.Environment}/entries?access_token={_config.ApiKeys?.PublishedContent}");
} 
catch
{
	return new List<BlogPostEntry>();
}
```

You will now also find that the UI project will run successfully, even if an error occurs.

## Eighth Test – Populate Title, Summary and Article from API

This final test will check the data coming back from the API is populating the BlogPostEntry object within the list of blogs returned.

### Failing Test

The test checks that known values (pulled from the constant) are assigned to the properties of the first blog entry in the list.

```csharp
[Fact]
public async void AListOfBlogsHasPopulatedValues()
{
	// Arrange
	IHttpClientFactory httpClientFactory = MockFactories.GetStringClient(Constants.RESULT_LIST_POPULATED_TWO);
	var sut = new BlogPost(httpClientFactory, Constants.GetContentfulConfig());

	// Act
	var results = await sut.ListAsync();

	// Assert
	results.Should().NotBeNullOrEmpty();
	results.First().Title.Should().Be("Test blog article");
	results.First().Summary.Should().Be("This is my __test__ blog article summary.");
	results.First().Article.Should().Be("This is the body __with bold__ of my article.");
}
```

To get the code to compile, add the missing “Article” property to BlogPostEntry:

```csharp
public string? Article { get; set; }
```

### Making the Test Green

To make the test pass, we need to modify the code within the foreach of the ListAsync method as shown below:

```csharp
var fields = (JsonElement)blogPostEntry["fields"];

blogPosts.Add(new BlogPostEntry()
{
	Title = fields.GetProperty("title").GetString() ?? "",
	Summary = fields.GetProperty("summary").GetString() ?? "",
	Article = fields.GetProperty("article").GetString() ?? ""
});
```

If you run the UI again, making sure the spaceId, environment and API keys are set correctly in appsettings.json and user secrets, then the title and summary will be displayed.

## Conclusion

This is obviously not a fully complete application but hopefully these articles have helped illustrate the TDD process, provide a practical demo and potentially some ways to mock or stub items.