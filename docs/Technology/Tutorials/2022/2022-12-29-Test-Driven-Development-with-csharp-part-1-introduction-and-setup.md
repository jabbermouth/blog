# Test Driven Development (TDD) – C# – Introduction and Setup (part 1 of 5)

This series of articles documents what I have learnt building the initial stages of a relatively simple application to retrieve information from Contentful’s API (I know they have an SDK, but handling HttpClient within the context of TDD was part of what I wanted to learn) and then displaying it on a Blazor Server frontend.

I’m documenting this, for the most part, in the order I did it. Due to the size of the topic, I’m splitting it into multiple parts with links to the next article included at the bottom of each article.

The primary purpose of this project was to learn and try TDD but, ultimately, I also wish to move my blog off WordPress so this is the first building block in developing that real-world application.

The code from this blog is [available on GitHub](https://github.com/jabbermouth/demo-tdd) with branches covering the various stages to allow you to follow along if desired. The default branch will only include the initial projects and NuGet packages referenced in “Initial Setup” below.

## TDD TL;DR

I am following the commonly used red, green, refactor approach normally used with TDD. This basically means writing a test (it fails/shows as red in test runners because the code isn’t implemented yet), writing enough code to make the test pass (green in test runners) then refactor to improve the test or code without fundamentally changing either. This process is then repeated for each new function or requirement to be added.

For the first (red) stage, some code will be needed to make the test code actually compile (e.g. creating a basic class with a method stub) but nothing more should be done. Throwing a not implemented exception in the added methods is a good way to do this.

## Getting Started

This section covers an overview of what the articles and, ultimately, code will cover and the initial setup of the solution. Whilst I’ve listed all the technologies used, I did not install all of these at the outset as I wanted to build out the solution iteratively as would be done on a larger project.

### Frameworks, Tools and Technologies Used

All versions used are the latest available versions at the time the projects were created or packages installed.

- .NET 7
- Blazor Server
- bUnit
- Contentful
- FluentAssertions
- Markdig
- Moq
- Visual Studio 2022
- xUnit

## Contentful

This project will be using [Contentful](https://www.contentful.com/) for it’s API-based data source. You can set up a free account to allow you to follow along with the process. To make life easy, set up a “Blog Article” content model and add three fields to it:

- Title (Short text)
- Summary (Long text)
- Article (Long text)

You can then create a piece of content of the type “Blog Article” for testing later.

## Initial C# Setup

Firstly, I created a new solution (Demo.Tdd) with four projects:

- Blazor Server Project (Empty Template) – Demo.Tdd.Ui
- Class library (to do connect to Contentful API) – Demo.Tdd.Contentful
- 2 xUnit projects – one for each of the above – as above with .Tests on the end

I then applied any updated and then installed [Moq](https://www.nuget.org/packages/Moq/4.18.3?_src=template) and [FluentAssertions](https://www.nuget.org/packages/FluentAssertions/6.8.0?_src=template) into both xUnit projects. Finally, I created project references between the respective test and code projects.

For the most part, I won’t specifically mention some changes such as switching to top-level namespaces or tidying up using statements however you will see these changes in the GitHub repo code.

## The Tests

In total, eight tests will be created throughout these articles. For convenience, I’ve listed these below and put direct links to both the article and the GitHub red-state branch (i.e. test written but failing).

1. Empty List ([Article](2022-12-29-Test-Driven-Development-with-csharp-part-2-api-calling.md) | [GitHub](https://github.com/jabbermouth/demo-tdd/tree/test1/1-red))
2. Populated List ([Article](2022-12-29-Test-Driven-Development-with-csharp-part-2-api-calling.md) | [GitHub](https://github.com/jabbermouth/demo-tdd/tree/test2/1-red))
3. Configuration ([Article](2022-12-30-Test-Driven-Development-with-csharp-part-3-faking-the-configuration.md) | [GitHub](https://github.com/jabbermouth/demo-tdd/tree/test3/1-red))
4. Blog List UI When Empty ([Article](2022-12-31-Test-Driven-Development-with-csharp-part-4-Blazor-Server-with-bUnit.md) | [GitHub](https://github.com/jabbermouth/demo-tdd/tree/test4/1-red))
5. Listing Blogs ([Article](2022-12-31-Test-Driven-Development-with-csharp-part-4-Blazor-Server-with-bUnit.md) | [GitHub](https://github.com/jabbermouth/demo-tdd/tree/test5/1-red))
6. Add BlogList to Index Page ([Article](2022-12-31-Test-Driven-Development-with-csharp-part-4-Blazor-Server-with-bUnit.md) | [GitHub](https://github.com/jabbermouth/demo-tdd/tree/test6/1-red))
7. Handle Errors ([Article](2022-12-31-Test-Driven-Development-with-csharp-part-5-Finishing-the-app.md) | [GitHub](https://github.com/jabbermouth/demo-tdd/tree/test7/1-red))
8. Populate Title, Summary and Article from API ([Article](2022-12-31-Test-Driven-Development-with-csharp-part-5-Finishing-the-app.md) | [GitHub](https://github.com/jabbermouth/demo-tdd/tree/test8/1-red))