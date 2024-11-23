---
title: Test Driven Development (TDD) – C# – Blazor Server with bUnit (part 4 of 5)
description: In this third article on building an app using TDD, we’ll build a blog list component and add it to the default page. We’ll be using bUnit for the testing of this component.
date: 2022-12-31
categories:
  - TDD
  - C\#
  - Blazor
  - bUnit
  - Tutorials
---
# Test Driven Development (TDD) – C# – Blazor Server with bUnit (part 4 of 5)

In this third article on building an app using TDD, we’ll build a blog list component and add it to the default page. We’ll be using bUnit for the testing of this component.

## Fourth Test – Blog List UI When Empty
The first test we’ll write will cover a blog list when no values are available. We’ll need to mock the BlogPost class created in the previous articles and we’ll do this by creating an interface of the class.

The test will look like this:

```csharp
[Fact]
public void ShowNoBlogsFoundOnEmptyList()
{
	// Arrange
	IEnumerable<BlogPostEntry> blogPostEntries = new List<BlogPostEntry>();
	Mock<IBlogPost> blogPost = new();
	blogPost.Setup(method => method.ListAsync()).Returns(Task.FromResult(blogPostEntries));

	Services.AddSingleton<IBlogPost>(blogPost.Object);
	var cut = RenderComponent<BlogList>();

	// Act

	// Assert
	cut.Find("p").MarkupMatches("<p>There are no blogs to display.</p>");
}
```

This will require [bUnit](https://www.nuget.org/packages/bunit/) and [Microsoft.Extensions.DependencyInjection](https://www.nuget.org/packages/Microsoft.Extensions.DependencyInjection/7.0.0?_src=template) installing. You will also need to create a project reference to Demo.Tdd.Contentful. The class should inherit from TestContext.

The next step is to create the interface for BlogPost and inherit from it. In Visual Studio, you can do this by going into the class, right clicking on the class name, choosing “Quick Actions and Refactorings…” and choosing the “Extract Interface” option. The default values should be sufficient. Click OK and the interface will be created and the current class set to inherit from it.

In the UI project, create a “Shared” folder and within it create a new Razor component called “BlogList.razor”.

### Making the Test Green

Populate the BlogList with the following:

```html
<p>There are no blogs to display.</p>

@code {

}
```

This will satisfy the test but is obviously never going to return anything dynamic and isn’t using the mocked object we’ve created in the test, etc… As done previously, we’ll create a new test that handles the scenario where data exists and make use of the mock we’ve created.

## Fifth Test – Listing Blogs

This test will require a few changes to make it pass and will require a couple of properties adding to BlogPostEntry just to make the code compile. This test should also spawn other tests such as one that says the ListAsync method returns title, summary and article when called.

## Failing Test

The new test should look like:

```csharp
[Fact]
public void ListMostRecentBlogs()
{
	// Arrange
	IEnumerable<BlogPostEntry> blogPostEntries = new List<BlogPostEntry>() { new BlogPostEntry() { Title = "Test blog article", Summary = "A bit of blurb with some __bold__ text." } };
	Mock<IBlogPost> blogPost = new();
	blogPost.Setup(method => method.ListAsync()).Returns(Task.FromResult(blogPostEntries));

	Services.AddSingleton<IBlogPost>(blogPost.Object);
	var cut = RenderComponent<BlogList>();

	// Act
	cut.WaitForState(() => cut.Find("ul") is not null);

	// Assert
	var ul = cut.Find("ul");
	ul.Children[0].MarkupMatches("<li><b>Test blog article</b><p>A bit of blurb with some <strong>bold</strong> text.</p></li>");
}
```

To make the code compile, add the following code to the BlogPostEntry class in Demo.Tdd.Contentful:

```csharp
public string? Title { get; set; }
public string? Summary { get; set; }
```

### Making the Test Green

To make both tests pass, the BlogPost component needs to be updated to accept the injected IBlogPost and iterate through any results. As we are storing Markdown in Contentful, we’ll need to use a package to convert this to HTML. The package I recommend is Markdig. Also, to make Blazor render the HTML as HTML and not a string, we’ll cast the value to MarkupString.

Update the BlogList component as follows:

```cshtml
@using Demo.Tdd.Contentful;
@using Demo.Tdd.Contentful.Models;
@using Markdig;

@inject IBlogPost _blogPost;

@if (blogList is null || !blogList.Any())
{
    <p>There are no blogs to display.</p>
}
else
{
    <ul>
    @foreach (var blogEntry in blogList)
    {
        <li>
            <b>@blogEntry.Title</b>
            @((MarkupString)Markdown.ToHtml(blogEntry.Summary ?? ""))
        </li>
    }
    </ul>
}

@code {
    IEnumerable<BlogPostEntry>? blogList = null;

    protected override async Task OnInitializedAsync()
    {
        blogList = await _blogPost.ListAsync();
    }
}
```

The test will now pass but the code will fail to run.

## Sixth Test – Add BlogList to Index Page

The final UI test for this series of blogs is to add the component to the Index.razor page. We’ll write a test for this to confirm the component is present.

### Failing Test

Firstly, create a new class within the UI testing project called HomePageShould, make the class public and inherit from TestContext. Then populate it with the following test:

```csharp
[Fact]
public void IncludeBlogListComponent()
{
    // Arrange
    Mock<IBlogPost> blogPost = new();
    Services.AddSingleton<IBlogPost>(blogPost.Object);
    var put = RenderComponent<Pages.Index>();

    // Act

    // Assert
    Assert.True(put.HasComponent<BlogList>());
}
```

No additional changes are required to make the code compile and test fail.

### Making the Test Green

Firstly, in the _Imports.razor page, add the following line:

```csharp
@using Demo.Tdd.Ui.Shared
```

This removes the need to add this to each page using a shared component.

The add the following to Index.razor:

```html
<BlogList></BlogList>
```

This will result in a passing test. The code won’t run but we’ll cover that in the refactor stage.

### Refactor to Make Code Run

To allow the code to run, in Program.cs, under the other service declarations, add the following:

```csharp
builder.Services.AddHttpClient();
builder.Services.AddSingleton<IBlogPost, BlogPost>();
```

Be aware that this will only work if no errors occur (i.e. the API key is correct and the space and environment values are correct. Also, as no code has been added to BlogPost.cs to return the title and summary, if you’ve added a blog entry, you won’t see the title or summary.

## Seventh Test – Handle an Error from the API
The final tests we’ll write are to return an empty list if an error occurs and to return values. This will be covered in the [next blog article](2022-12-31-Test-Driven-Development-with-csharp-part-5-Finishing-the-app.md).