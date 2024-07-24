<!--
GENERATED FILE - DO NOT EDIT
This file was generated by [MarkdownSnippets](https://github.com/SimonCropp/MarkdownSnippets).
Source File: /readme.source.md
To change this file edit the source file and then run MarkdownSnippets.
-->

# <img src="/src/icon.png" height="30px"> Verify.Blazor

[![Discussions](https://img.shields.io/badge/Verify-Discussions-yellow?svg=true&label=)](https://github.com/orgs/VerifyTests/discussions)
[![Build status](https://ci.appveyor.com/api/projects/status/spyere4ubpl1tca8?svg=true)](https://ci.appveyor.com/project/SimonCropp/Verify-Blazor)
[![NuGet Status](https://img.shields.io/nuget/v/Verify.Bunit.svg?label=Verify.Bunit)](https://www.nuget.org/packages/Verify.Bunit/)
[![NuGet Status](https://img.shields.io/nuget/v/Verify.Blazor.svg?label=Verify.Blazor)](https://www.nuget.org/packages/Verify.Blazor/)

Support for rendering a [Blazor Component](https://docs.microsoft.com/en-us/aspnet/core/blazor/#components) to a
verified file via [bunit](https://bunit.egilhansen.com) or via raw Blazor rendering.

**See [Milestones](../../milestones?state=closed) for release notes.**

## Component

The below samples use the following Component:

<!-- snippet: BlazorApp/TestComponent.razor -->
<a id='snippet-BlazorApp/TestComponent.razor'></a>
```razor
<div>
    <h1>@Title</h1>
    <p>@Person.Name</p>
    <button>MyButton</button>
</div>

@code {

    [Parameter]
    public string Title { get; set; } = "My Test Component";

    [Parameter]
    public Person Person { get; set; }

    public bool Intitialized;

    protected override Task OnInitializedAsync()
    {
        Intitialized = true;
        return Task.CompletedTask;
    }

}
```
<sup><a href='/src/BlazorApp/TestComponent.razor#L1-L23' title='Snippet source file'>snippet source</a> | <a href='#snippet-BlazorApp/TestComponent.razor' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

## Verify.Blazor

Verify.Blazor uses the Blazor APIs to take a snapshot (metadata and html) of the current state of a Blazor component. It
has fewer dependencies and is a simpler API than [Verify.Bunit approach](#verifybunit), however it does not provide many
of the other features, for
example [trigger event handlers](https://bunit.egilhansen.com/docs/interaction/trigger-event-handlers.html).

### NuGet package

* https://nuget.org/packages/Verify.Blazor/

### Usage

#### Render using ParameterView

This test:

<!-- snippet: BlazorComponentTestWithParameters -->
<a id='snippet-BlazorComponentTestWithParameters'></a>
```cs
[Fact]
public Task PassingParameters()
{
    var parameters = ParameterView.FromDictionary(
        new Dictionary<string, object?>
        {
            {
                "Title", "The Title"
            },
            {
                "Person", new Person
                {
                    Name = "Sam"
                }
            }
        });

    var target = Render.Component<TestComponent>(parameters: parameters);

    return Verify(target);
}
```
<sup><a href='/src/Verify.Blazor.Tests/Samples.cs#L16-L40' title='Snippet source file'>snippet source</a> | <a href='#snippet-BlazorComponentTestWithParameters' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

#### Render using template instance

This test:

<!-- snippet: BlazorComponentTestWithTemplateInstance -->
<a id='snippet-BlazorComponentTestWithTemplateInstance'></a>
```cs
[Fact]
public Task PassingTemplateInstance()
{
    var template = new TestComponent
    {
        Title = "The Title",
        Person = new()
        {
            Name = "Sam"
        }
    };

    var target = Render.Component(template: template);

    return Verify(target);
}
```
<sup><a href='/src/Verify.Blazor.Tests/Samples.cs#L42-L61' title='Snippet source file'>snippet source</a> | <a href='#snippet-BlazorComponentTestWithTemplateInstance' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

#### Result

Both will produce:

The component rendered as html `...verified.html`:

<!-- snippet: Verify.Blazor.Tests/Samples.PassingParameters.verified.html -->
<a id='snippet-Verify.Blazor.Tests/Samples.PassingParameters.verified.html'></a>
```html
<div>
  <h1>The Title</h1>
  <p>Sam</p>
  <button>MyButton</button>
</div>
```
<sup><a href='/src/Verify.Blazor.Tests/Samples.PassingParameters.verified.html#L1-L5' title='Snippet source file'>snippet source</a> | <a href='#snippet-Verify.Blazor.Tests/Samples.PassingParameters.verified.html' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

And the current model rendered as txt `...verified.txt`:

<!-- snippet: Verify.Blazor.Tests/Samples.PassingParameters.verified.txt -->
<a id='snippet-Verify.Blazor.Tests/Samples.PassingParameters.verified.txt'></a>
```txt
{
  Instance: {
    Intitialized: true,
    Title: The Title,
    Person: {
      Name: Sam
    }
  }
}
```
<sup><a href='/src/Verify.Blazor.Tests/Samples.PassingParameters.verified.txt#L1-L9' title='Snippet source file'>snippet source</a> | <a href='#snippet-Verify.Blazor.Tests/Samples.PassingParameters.verified.txt' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

## Verify.Bunit

Verify.Bunit uses the bUnit APIs to take a snapshot (metadata and html) of the current state of a Blazor component.
Since it leverages the bUnit API, snapshots can be on a component that has been manipulated using the full bUnit feature
set, for example [trigger event handlers](https://bunit.egilhansen.com/docs/interaction/trigger-event-handlers.html).

### NuGet package

* https://nuget.org/packages/Verify.Bunit/

### Usage

Enable at startup:

<!-- snippet: BunitEnable -->
<a id='snippet-BunitEnable'></a>
```cs
[ModuleInitializer]
public static void Initialize() =>
    VerifyBunit.Initialize();
```
<sup><a href='/src/Verify.Bunit.Tests/ModuleInitializer.cs#L3-L9' title='Snippet source file'>snippet source</a> | <a href='#snippet-BunitEnable' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

This test:

<!-- snippet: BunitComponentTest -->
<a id='snippet-BunitComponentTest'></a>
```cs
[Fact]
public Task Component()
{
    using var context = new TestContext();
    var component = context.RenderComponent<TestComponent>(
        builder =>
        {
            builder.Add(
                _ => _.Title,
                "New Title");
            builder.Add(
                _ => _.Person,
                new()
                {
                    Name = "Sam"
                });
        });
    return Verify(component);
}

[Fact]
public Task MarkupFormattable_NodeList()
{
    using var context = new TestContext();
    var component = context.RenderComponent<TestComponent>(
        builder =>
        {
            builder.Add(
                _ => _.Title,
                "New Title");
            builder.Add(
                _ => _.Person,
                new()
                {
                    Name = "Sam"
                });
        });
    return Verify(component.Nodes);
}

[Fact]
public Task MarkupFormattable_single_Element()
{
    using var context = new TestContext();
    var component = context.RenderComponent<TestComponent>(
        builder =>
        {
            builder.Add(
                _ => _.Title,
                "New Title");
            builder.Add(
                _ => _.Person,
                new()
                {
                    Name = "Sam"
                });
        });
    return Verify(component.Nodes.First()
        .FirstChild);
}
```
<sup><a href='/src/Verify.Bunit.Tests/Samples.cs#L3-L66' title='Snippet source file'>snippet source</a> | <a href='#snippet-BunitComponentTest' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

Will produce:

The component rendered as html `...Component.verified.html`:

<!-- snippet: Verify.Bunit.Tests/Samples.Component.verified.html -->
<a id='snippet-Verify.Bunit.Tests/Samples.Component.verified.html'></a>
```html
<div>
  <h1>New Title</h1>
  <p>Sam</p>
  <button>MyButton</button>
</div
```
<sup><a href='/src/Verify.Bunit.Tests/Samples.Component.verified.html#L1-L5' title='Snippet source file'>snippet source</a> | <a href='#snippet-Verify.Bunit.Tests/Samples.Component.verified.html' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

And the current model rendered as txt `...Component.verified.txt`:

<!-- snippet: Verify.Bunit.Tests/Samples.Component.verified.txt -->
<a id='snippet-Verify.Bunit.Tests/Samples.Component.verified.txt'></a>
```txt
{
  Instance: {
    Intitialized: true,
    Title: New Title,
    Person: {
      Name: Sam
    }
  },
  NodeCount: 9
}
```
<sup><a href='/src/Verify.Bunit.Tests/Samples.Component.verified.txt#L1-L10' title='Snippet source file'>snippet source</a> | <a href='#snippet-Verify.Bunit.Tests/Samples.Component.verified.txt' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

### Exclude Component

Rendering of the Component state (Samples.Component.verified.txt from above) can be excluded by
using `excludeComponent`.

<!-- snippet: BunitEnableExcludeComponent -->
<a id='snippet-BunitEnableExcludeComponent'></a>
```cs
[ModuleInitializer]
public static void Initialize() =>
    VerifyBunit.Initialize(excludeComponent: true);
```
<sup><a href='/src/Verify.Bunit.ExcludeComponentTests/ModuleInitializer.cs#L3-L9' title='Snippet source file'>snippet source</a> | <a href='#snippet-BunitEnableExcludeComponent' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

## Scrubbing

### Integrity check

In Blazor an integrity check is applied to the `dotnet.*.js` file.

```
<script src="_framework/dotnet.5.0.2.js" defer="" integrity="sha256-AQfZ6sKmq4EzOxN3pymKJ1nlGQaneN66/2mcbArnIJ8=" crossorigin="anonymous"></script>
```

This line will change when the dotnet SDK is updated.

### Noise in rendered template

Blazor uses `<!--!-->` to delineate components in the resulting html. Some empty lines can be rendered when components
are stitched together.

### Resulting scrubbing

<!-- snippet: scrubbers -->
<a id='snippet-scrubbers'></a>
```cs
// remove some noise from the html snapshot
VerifierSettings.ScrubEmptyLines();
BlazorScrubber.ScrubCommentLines();
VerifierSettings.ScrubLinesWithReplace(s =>
{
    var scrubbed = s.Replace("<!--!-->", "");
    if (string.IsNullOrWhiteSpace(scrubbed))
    {
        return null;
    }

    return scrubbed;
});
HtmlPrettyPrint.All();
VerifierSettings.ScrubLinesContaining("<script src=\"_framework/dotnet.");
```
<sup><a href='/src/Verify.Blazor.Tests/ModuleInitializer.cs#L10-L28' title='Snippet source file'>snippet source</a> | <a href='#snippet-scrubbers' title='Start of snippet'>anchor</a></sup>
<!-- endSnippet -->

## Credits

* [Unit testing Blazor components - a prototype - Steven Sanderson](https://blog.stevensanderson.com/2019/08/29/blazor-unit-testing-prototype/)
* [Bunit - Egil Hansen](https://bunit.egilhansen.com)

## Icon

[Helmet](https://thenounproject.com/term/helmet/9554/) designed
by [Leonidas Ikonomou](https://thenounproject.com/alterego) from [The Noun Project](https://thenounproject.com).