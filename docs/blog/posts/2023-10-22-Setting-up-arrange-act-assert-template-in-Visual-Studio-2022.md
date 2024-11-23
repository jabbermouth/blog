---
title: Setting up an Arrange, Act, Assert comment template in Visual Studio 2022
description: Adding a simple code snippet that allows a set of arrange, act and assert comments to be inserted using the short code of `aaa`.
date: 2023-10-22
categories:
  - Visual Studio
  - Quick Tips
---
# Setting up an Arrange, Act, Assert comment template in Visual Studio 2022

To add a simple code snippet that allows a set of arrange, act and assert comments to be inserted using the short code of `aaa`, firstly, create a simple XML file with the following contents and save it as `ArrangeActAssert.snippet`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<CodeSnippets xmlns="http://schemas.microsoft.com/VisualStudio/2005/CodeSnippet" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <CodeSnippet Format="1.0.0">
        <Header>
            <Title>Arrange Act Assert</Title>
            <Author>Neil Docherty</Author>
            <Description>Adds an arrange, act, assert to a test</Description>
            <HelpUrl />
            <Keywords />
            <SnippetTypes />
            <Shortcut>aaa</Shortcut>
        </Header>
        <Snippet>
            <Declarations />
            <References />
            <Code Kind="any" Language="CSharp"><![CDATA[// Arrange
$end$

// Act


// Assert]]></Code>
        </Snippet>
    </CodeSnippet>
</CodeSnippets>
```

Then, to import it into Visual Studio, perform the following steps:

1. Go to the `Tools` menu
2. Select `Code Snippets Manager`
3. Choose the `Import` button and locate and select your `ArrangeActAssert.snippet` file and click Open.

Now, when editing some C# code, typing aaa and hitting tab twice will create a block of text similar to:

```csharp
// Arrange


// Act


// Assert
```