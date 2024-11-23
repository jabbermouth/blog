---
title: C# HTTP Client Factory (IHttpClientFactory) mock using NSubstitute
description: Mocking the IHttpClientFactory using NSubstitute.
date: 2023-10-05
categories:
  - C\#
  - NSubstitute
  - Quick Tips
---
# HTTP Client Factory (IHttpClientFactory) mock using NSubstitute

Following the controversy around [Moq](https://www.devlooped.com/moq/), I decided to look into using [NSubstitute](https://nsubstitute.github.io/) as an alternative mocking library.

The example below is an NSubstitute version of the class created as part of my [C# TDD blog article](https://www.jabbermouth.co.uk/2022/12/29/test-driven-development-tdd-using-blazor-server-and-calling-an-api-ii/#httpclient-mocking). The only change in functionality is the optional passing in of an HTTP status code.

```csharp
public static IHttpClientFactory GetStringClient(string returnValue, HttpStatusCode returnStatusCode = HttpStatusCode.OK)
{
	var httpMessageHandler = Substitute.For<HttpMessageHandler>();
	var httpClientFactory = Substitute.For<IHttpClientFactory>();

	httpMessageHandler
		.GetType()
		.GetMethod("SendAsync", BindingFlags.NonPublic | BindingFlags.Instance)
		.Invoke(httpMessageHandler, new object[] { Arg.Any<HttpRequestMessage>(), Arg.Any<CancellationToken>() })
		.Returns(Task.FromResult(new HttpResponseMessage(returnStatusCode) { Content = new StringContent(returnValue) }));

	HttpClient httpClient = new(httpMessageHandler);
	httpClientFactory.CreateClient(Arg.Any<string>()).Returns(httpClient);

	return httpClientFactory;
}
```