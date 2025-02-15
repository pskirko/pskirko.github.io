---
layout: post
title:  "Giving Semantic Kernel a Try"
date:   2025-02-15 14:36:00 -0800
categories: dotnet, semantic-kernel
---
Last weekend, I decided to give [Semantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/overview/)
a try. I am curious how these AI app frameworks are shaping up, especially to see if they provide appreciable value
over just stitching together the root functionality yourself. Is it just saving time to get started
(i.e., "time to productive"), or is there a deeper value (i.e., solving a hard problem, or providing an
industry standard that is widely/dominantly adopted like TypeScript or React)?

So I decided to start kicking the tires on Semantic Kernel. I didn't get terribly far: just through
the intro tutorial, and that was it for the time available. But between trying the intro tutorial, and reading
various docs, I still got a taste of things.

To not bury the lede: I think it's interesting. I'll be giving LangChain a try too at some point, to compare them. But,
I liked the Plugin concept, especially how it can be a useful abstraction for local operations, since
my interest is in building little personal productivity applications to help get things done at home
on my laptop (can LLMs save writing out rules and code?). Of course, LangChain can use local Python code as tools too, so this isn't a unique concept.
I'll just have to try both to really get a taste of the difference.

You might be wondering why I started with Semantic Kernel, instead of LangChain. It's just related
to the fact I spent many years in the past writing C#/.Net code for work, so it feels familiar.
I've used Python mostly to try things at home, less so at work. So as much as I enjoy various aspects
of Python, I feel incrementally faster getting started with a C# SDK, and I'm familiar with how
Microsoft lays out docs and explains things. There are drawbacks, like Microsoft's history of
just putting too much functionality into things (e.g., .NET framework), or just missing the mark or
timing with other efforts (e.g., WPF), or just too many layers of functionality (e.g., ASP.net Core),
but we'll see.

I decided to use the C# variant of Semantic Kernel, because I was convinced there would be gaps in the other languages.
I know, [last week]({% link _posts/2025-02-08-getting-back-on-dev-log-horse.markdown %}) I wrote I wanted to stick with
Java. But Microsoft is C# first and definitely not Java first, so I just had a hunch it would be less smooth.
Lo and behold, I later found this in the docs, on the [AI Services Overview](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/)
page:

![Semantic Kernel Compat](/images/2025-02-15-semantic-kernel-compat.png)

Of course C# has the most support. But that's not necessarily a bad thing. Especially if you're not sure
of API shapes, it's better to figure it out in one language before propagating it to the others,
to minimize churn and change radius, I get it.

The intro tutorial for C# is [here](https://learn.microsoft.com/en-us/semantic-kernel/get-started/quick-start-guide?pivots=programming-language-csharp). I generally type in code and don't copy/paste, but that said,
typing in and running the code was maybe 20% of the time spent. The rest of the time was just reading
through docs and futzing with VS Code.

My code mostly follows the tutorial, modulo some light changes. First, the main file:

{% highlight c# %}
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using Microsoft.SemanticKernel.Connectors.OpenAI;

// See https://aka.ms/new-console-template for more information
Console.WriteLine("Hello World. Let's try an AI example.");

var modelId = "gpt-4o";
var apiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY");

if (apiKey == null) {
    Console.WriteLine("Could not retrieve OpenAI API key");
    System.Environment.Exit(1);
}

var builder = Kernel.CreateBuilder().AddOpenAIChatCompletion(modelId, apiKey);
builder.Services.AddLogging(services => services.AddConsole().SetMinimumLevel(LogLevel.Trace));

var kernel = builder.Build();
var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();

kernel.Plugins.AddFromType<LightsPlugin>("Lights");

OpenAIPromptExecutionSettings settings = new() {
    FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
};

var history = new ChatHistory();

do {
    Console.Write("User > ");
    var userInput = Console.ReadLine();
    if (userInput == null) {
        Console.WriteLine("<Did not receive input from user>");
        System.Environment.Exit(1);
    }

    history.AddUserMessage(userInput);

    var result = await chatCompletionService.GetChatMessageContentAsync(
        history,
        executionSettings: settings,
        kernel: kernel);

    var content = result.Content ?? "<Did not receive content from OpenAI>";
    Console.WriteLine("Assistant > " + content);
    history.AddMessage(result.Role, content);
} while(true);
{% endhighlight c# %}

And here is the plugin file:
{% highlight c# %}
using Microsoft.SemanticKernel;
using System.ComponentModel;
using System.Text.Json.Serialization;

public class LightsPlugin
{
    private readonly List<LightModel> lights = new() {
        new LightModel { Id = 1, Name = "Table Lamp", IsOn = false },
        new LightModel { Id = 2, Name = "Porch Light", IsOn = false },
        new LightModel { Id = 3, Name = "Chandelier", IsOn = true },
    };

    [KernelFunction("get_lights")]
    [Description("Gets the list of lights, including their on/off statuses")]
    public async Task<List<LightModel>> GetLightsAsync()
    {
        return lights;
    }

    [KernelFunction("change_state")]
    [Description("Changes the on/off status of the specified light")]
    public async Task<LightModel?> ChangeStateAsync(int id, bool isOn) {
        var light = lights.FirstOrDefault(l => l.Id == id);
        if (light == null)
        {
            return null;
        }

        light.IsOn = isOn;
        return light;
    }
}

public class LightModel
{
    [JsonPropertyName("id")]
    public int Id { get; set; }

    [JsonPropertyName("name")]
    public required string Name { get; set; }

    [JsonPropertyName("is_on")]
    public bool? IsOn { get; set; }
}
{% endhighlight c# %}

Like I said, I like the Plugin concept. I can see how I can wrap local functionality, be it over
DBs, or filesystem paths, etc. with a uniform interface. Also C# has been progressive with annotations,
which works out well here for the Plugin.

Ok, here's some sample interaction and output:

{% highlight shell %}
(base) pskirko@ps22 semantic-kernel-quick-start %  /Users/pskirko/.vscode/extensions/ms-dotnettools.csharp-2.63.32-darwin-a
rm64/.debugger/arm64/vsdbg --interpreter=vscode --connection=/var/folders/tt/85s29xk105xdj2wwyzlspr4h0000gn/T/CoreFxPipe_vs
dbg-ui-80a8aa52ea3f40019fcdf1973e1a277c
Hello World. Let us try an AI example.
User > When did Semantic Kernel start
trce: Microsoft.SemanticKernel.Connectors.OpenAI.OpenAIChatCompletionService[0]
      ChatHistory: [{"Role":{"Label":"user"},"Items":[{"$type":"TextContent","Text":"When did Semantic Kernel start"}]}], Settings: {"service_id":null,"model_id":null,"function_choice_behavior":{"type":"auto","functions":null,"options":null}}
dbug: Microsoft.SemanticKernel.Connectors.OpenAI.OpenAIChatCompletionService[0]
      Function choice behavior configuration: Choice:auto, AutoInvoke:True, AllowConcurrentInvocation:False, AllowParallelCalls:(null) Functions:Lights-get_lights, Lights-change_state
info: Microsoft.SemanticKernel.Connectors.OpenAI.OpenAIChatCompletionService[0]
      Prompt tokens: 80. Completion tokens: 43. Total tokens: 123.
Assistant > Semantic Kernel was introduced by Microsoft as an open-source project in March 2023. It aims to provide frameworks and tools for building machine learning models and applications that integrate large language models with traditional application development.
User > When did LangChain start
trce: Microsoft.SemanticKernel.Connectors.OpenAI.OpenAIChatCompletionService[0]
      ChatHistory: [{"Role":{"Label":"user"},"Items":[{"$type":"TextContent","Text":"When did Semantic Kernel start"}]},{"Role":{"Label":"Assistant"},"Items":[{"$type":"TextContent","Text":"Semantic Kernel was introduced by Microsoft as an open-source project in March 2023. It aims to provide frameworks and tools for building machine learning models and applications that integrate large language models with traditional application development."}]},{"Role":{"Label":"user"},"Items":[{"$type":"TextContent","Text":"When did LangChain start"}]}], Settings: {"service_id":null,"model_id":null,"function_choice_behavior":{"type":"auto","functions":null,"options":null}}
dbug: Microsoft.SemanticKernel.Connectors.OpenAI.OpenAIChatCompletionService[0]
      Function choice behavior configuration: Choice:auto, AutoInvoke:True, AllowConcurrentInvocation:False, AllowParallelCalls:(null) Functions:Lights-get_lights, Lights-change_state
info: Microsoft.SemanticKernel.Connectors.OpenAI.OpenAIChatCompletionService[0]
      Prompt tokens: 134. Completion tokens: 59. Total tokens: 193.
Assistant > LangChain was introduced in late 2022 as a framework designed to simplify the development of applications using large language models (LLMs). It provides tools and components to streamline the processes of building and deploying applications that leverage LLMs, making it easier for developers to harness their capabilities.
User >
{% endhighlight shell %}

I had to change "Let's" to "Let us" above because the unmatched ' was causing syntax mis-highlighting.

Sigh, I don't understand most people's logging conventions. It's like the worst of both worlds:
 * It's not purely a structued JSON or similar output
 * And it's not purely optimized for human readability

So yea, worst of both worlds. Pick one and optimize for it.

Also, I really dislike the fact that they don't log the prompt by default. I get it that you have
to setup a [filter](https://learn.microsoft.com/en-us/semantic-kernel/concepts/enterprise-readiness/filters?pivots=programming-language-csharp)
to do that. But any attempt to hide prompts these days is just silly, it's like a very important
thing to eyeball. So I'd prefer the tutorial code to do that.

Anyways, TBD on next steps and/or trying LangChain out.