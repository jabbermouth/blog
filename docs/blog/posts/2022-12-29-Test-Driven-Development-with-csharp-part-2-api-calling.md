---
title: Test Driven Development (TDD) – C# – API Calling (part 2 of 5)
description: The first test I decided to write was to have an empty result set from the API return an empty list of blog post entries. This would involve the code connecting out to the API, retrieving data and then converting that to a suitable object structure.
date: 2022-12-29
categories:
  - TDD
  - C\#
  - Blazor
  - bUnit
  - Tutorials
---
# Test Driven Development (TDD) – C# – API Calling (part 2 of 5)

## First Test – Empty List

The first test I decided to write was to have an empty result set from the API return an empty list of blog post entries. This would involve the code connecting out to the API, retrieving data and then converting that to a suitable object structure.

I will be using the arrange/act/assert structure for all tests, skipping any section where necessary.

Whilst ultimately the code will need to read settings from a config file, call out to an API, handle working and non-working conditions and return a list, the first test does not need to consider all these scenarios. Adding in some of these scenarios may (will!) require code refactoring but that is to be expected with any iterative design.

### Test Naming

The naming convention I’m using is to name the testing class after the class or component being tested with “Should” on the end (e.g. if the code class is called BlogPost then the test class would be called BlogPostShould).

The individual tests then describe what is being tested but is human readable, potentially in broken English, when including the test name e.g. ReturnAnEmptyListWhenNoBlogsExist would read as BlogPost should return an empty list when no blogs exist.

### First Test Breakdown

The first test is covering the scenario of connecting to the Contentful API, retrieving an empty list of blogs and returning an empty list of blog entry objects. It’s not handling errors, checking Contentful or HttpClient works or even defining what fields will be part of the blog entry class.

### HttpClient Mocking

I had already decided to use IHttpClientFactory for getting my HttpClient instance so began by looking at how to mock the API response. After some Googling and pulling answers from a few sources, I came up with the method outlined below. I placed this in a static class called MockFactories to make consuming it easier.

```csharp
public static IHttpClientFactory GetStringClient(string returnValue)
{
    Mock<HttpMessageHandler> httpMessageHandler = new();
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

This will return the value of `returnValue` and an OK (200) response, regardless of what URL is requested.

### Test Code

The first test code was pretty straightforward thanks to the above mocking class. Realising some constants will be needed, a class was created called “Constants” which will hold any constants. The below test code won’t compile at this stage but it will shortly…

```csharp
[Fact]
public async void ReturnAnEmptyListWhenNoBlogsExist()
{
    // Arrange
    IHttpClientFactory httpClientFactory = MockFactories.GetStringClient(Constants.RESULT_LIST_EMPTY);
    var sut = new BlogPost(httpClientFactory);

    // Act
    var results = await sut.ListAsync();

    // Assert
    results.Should().NotBeNull();
    results.Should().BeEmpty();
}
```

This code won’t compile because the constant (class) doesn’t exist, there is no class of “BlogPost” and therefore no method called “ListAsync” on such a class.

### Filling the Gaps

To make the test compile, some gaps need to be plugged. We’ll tackle these in the order outlined above.

First, create a static class called Constants and in it and a string constant called `RESULT_LIST_EMPTY` which includes an API response example of an empty list. What this will look like will vary depending on the API you’re referencing. Contentful’s documentation is pretty good and includes a live test page that you can use to test API calls and see what an example response will look like. In all cases, it’s the structure we’re after so you may wish to review any example data to make sure sensitive content is replaced (e. IDs and especially personal data).

For an empty result set, the string constant will look like this:

```csharp
public const string RESULT_LIST_EMPTY = """
    {
        "sys": {
        "type": "Array"
        },
        "total": 0,
        "skip": 0,
        "limit": 100,
        "items": []
    }
    """;
```

The `"""` is a C# 11 feature called [raw string literals](https://devblogs.microsoft.com/dotnet/csharp-11-preview-updates/). This makes it much easier to have constants that contain quotes, etc…

Next thing to create is a class called BlogPost (we’ll skip an interface for now). This needs a constructor that accepts an `IHttpClientFactory` as a parameter.

Technically, at this stage, an async method isn’t needed as it’s not actually making an API call but it will be very soon so will create the method in that way:

```csharp
public async Task<IEnumerable<BlogPostEntry>> ListAsync()
{
    throw new NotImplementedException();
}
```

This will now cause another compile issue so create a new model class called called `BlogPostEntry`. For now, it doesn’t need any properties.

Compiling the code and running the tests should now work and give a failing test.

### Making the Test Green

To make the test pass, the easiest thing to do is make the ListAsync method return an empty array so we’ll do this now.

```csharp
public async Task<IEnumerable<BlogPostEntry>> ListAsync()
{
    return new List<BlogPostEntry>();
}
```

At this point, two things could happen, the code could be refactored to make use of the `IHttpClientFactory` that was passed in or a new test could be written to check the code returns results when some items are returned from the API. The advantage of the latter approach is that the new code is a result of a new test which means the change is being more thoroughly tested.

## Second Test – Populated List

As described above, this test covers the scenario when results are present in the API response.

### Failing Test

The code for the second test is similar to the first test but with a different constant and different second expectation/assert.

```csharp
[Fact]
public async void ReturnAListOfBlogs()
{
    // Arrange
    IHttpClientFactory httpClientFactory = MockFactories.GetStringClient(Constants.RESULT_LIST_POPULATED_TWO);
    var sut = new BlogPost(httpClientFactory);

    // Act
    var results = await sut.ListAsync();

    // Assert
    results.Should().NotBeNullOrEmpty();
    results.Should().HaveCount(2);
}
```

To make this compile, create a second string constant in the Constants class with the following value:

```csharp
public const string RESULT_LIST_POPULATED_TWO = """
    {
        "sys": {
        "type": "Array"
        },
        "total": 2,
        "skip": 0,
        "limit": 100,
        "items": [
        {
            "metadata": {
            "tags": []
            },
            "sys": {
            "space": {
                "sys": {
                "type": "Link",
                "linkType": "Space",
                "id": "b27mtzlwgnec"
                }
            },
            "id": "1dP03LAJlww9kikCDLFTXL",
            "type": "Entry",
            "createdAt": "2022-12-26T16:41:06.408Z",
            "updatedAt": "2022-12-26T17:08:57.292Z",
            "environment": {
                "sys": {
                "id": "production",
                "type": "Link",
                "linkType": "Environment"
                }
            },
            "revision": 2,
            "contentType": {
                "sys": {
                "type": "Link",
                "linkType": "ContentType",
                "id": "blogArticle"
                }
            },
            "locale": "en-US"
            },
            "fields": {
            "title": "Test blog article",
            "summary": "This is my __test__ blog article summary.",
            "article": "This is the body __with bold__ of my article."
            }
        },
        {
            "metadata": {
            "tags": []
            },
            "sys": {
            "space": {
                "sys": {
                "type": "Link",
                "linkType": "Space",
                "id": "b27mtzlwgnec"
                }
            },
            "id": "2dP03LAJlww9kikMBLBNPQ",
            "type": "Entry",
            "createdAt": "2022-12-27T16:41:06.408Z",
            "updatedAt": "2022-12-27T17:08:57.292Z",
            "environment": {
                "sys": {
                "id": "production",
                "type": "Link",
                "linkType": "Environment"
                }
            },
            "revision": 2,
            "contentType": {
                "sys": {
                "type": "Link",
                "linkType": "ContentType",
                "id": "blogArticle"
                }
            },
            "locale": "en-US"
            },
            "fields": {
            "title": "Another test blog article",
            "summary": "This is my __second__ blog article summary.",
            "article": "This body __also has bold__ in my article."
            }
        }
        ]
    }
    """;
```

The code will now compile but fail the second test.

### Making the Test Green

Firstly, let’s get the parameter injected into the constructor available as a private variable. As it’s the `HttpClient` we want to have available, we’ll create a private variable for it:

```csharp
private readonly HttpClient _httpClient;
```

Then add the following to the constructor to create the HttpClient and specify a base address (which we’ll hard code for now):

```csharp
public BlogPost(IHttpClientFactory httpClientFactory)
{
    _httpClient = httpClientFactory.CreateClient();
    _httpClient.BaseAddress = new Uri("https://cdn.contentful.com");
}
```

Whilst we aren’t calling anything for real yet, a `BaseAddress` is needed so we might as well set it to the real one.

We now need to call the mock `HttpClient`. We will need the real URL to call to make the actual code work but, for now, we’ll just populate a placeholder URL as we’re not currently running the code other than as part of a test.

```csharp
public async Task<IEnumerable<BlogPostEntry>> ListAsync()
{
    string? posts = await _httpClient.GetStringAsync("/placeholder-path");

    var response = JsonSerializer.Deserialize<ContentfulResponse>(posts);

    if (response is null || response.items is null || !response.items.Any())
    {
        return new List<BlogPostEntry>();
    }

    List<BlogPostEntry> blogPosts = new();

    foreach (var blogPostEntry in response.items)
    {
        blogPosts.Add(new BlogPostEntry());
    }

    return blogPosts;
}
```

Note the ContentfulResponse in the JSON deserialize command, which is currently not defined. Create a new class for this with the following content:

```csharp
internal class ContentfulResponse
{
    public int total { get; set; }
    public List<Dictionary<string, object>>? items { get; set; }
}
```

Note that the code we have written will return a list with empty elements. To remedy this, we’d right another test to make sure content is populated. This could be combined into this test if desired but many simple tests can be helpful when changing or refactoring code as you get a narrower target to look for issues. It also allows for better naming of your tests.

### Refactor The Code

The current test may pass but the code won’t actually work due to the placeholder API URL so we’ll refactor the code slightly to include the real address. This example won’t work for you as space ID, environment and access token will all be different (these aren’t real tokens, obviously!) but if you insert the equivalent into your code, you’ll find the code will run and work, not that we have a visual way to test it yet but we’ll get to that…

Replace:

```csharp
_httpClient.GetStringAsync("/placeholder-path");
```

With:

```csharp
_httpClient.GetStringAsync("/spaces/b27mtzlwgnec/environments/production/entries?access_token=dskuhg87hguogj487g84gkh");
```

Re-run your tests to make sure you haven’t broken something.

## Third Test – Configuration

The next test is to start pulling configuration from a config file. As this article is quite large already, this will be covered in the [next article](2022-12-30-Test-Driven-Development-with-csharp-part-3-faking-the-configuration.md).