# Update a Blazor UI when a message is pushed to a RabbitMQ queue

**Scenario:** I’ve pushed a message to RabbitMQ for a long running service and I want to be notified when it’s finished. There are several ways this could be done but the way I wanted to try, given RabbitMQ was already part of the setup, was to have a message appearing on a queue trigger a UI update whilst making use of Blazored.Toast.

It’s quite likely what’s discussed here will work in Blazor WebAssembly but I’ve only tried it in Blazor Server.

The code for this project is [available in GitHub](https://github.com/jabbermouth/blazor-rabbitmq-notifications).

## Prerequesites and Setup

For the purposes of this documentation, it is assumed an instance of RabbitMQ is running locally. If you need to set one up, install Docker (if required) then run the following command:

```powershell
docker run --name rabbitmq -d --restart always -p 5672:5672 -p 15672:15672 rabbitmq:management
```

The admin UI will be accessible on [http://localhost:15672/](http://localhost:15672/) with a username and password of `guest`.

To begin, create a standard Blazor Server app (this can be done in the Visual Studio UI or from the command line:

```powershell
dotnet new blazorserver -n Demo.RabbitMQ.Notifications
```

## Get RabbitMQ Client working

Firstly, install the `RabbitMQ.Client` and `Blazored.Toast` NuGet packages into the Blazor Server project.

Next, create a new class file (e.g. RabbitMQ.cs) and, if desired, place it in a suitable folder (e.g. Messaging) and then populate it with the following, updating the namespace as needed:

```csharp
using RabbitMQ.Client.Events;
using RabbitMQ.Client;
using System.Text;

namespace Demo.RabbitMQ.Notifications.Messaging;

public class RabbitMQ
{
    public static event Func<string, Task> MessageReceived;

    public async void Setup(CancellationToken cancellationToken)
    {
        var factory = new ConnectionFactory() { HostName = "host.docker.internal" };
        using (var connection = factory.CreateConnection())
        using (var channel = connection.CreateModel())
        {
            channel.QueueDeclare(queue: "user-queue",
                                 durable: true,
                                 exclusive: false,
                                 autoDelete: false,
                                 arguments: null);

            var consumer = new EventingBasicConsumer(channel);
            consumer.Received += (model, ea) =>
            {
                var body = ea.Body.ToArray();
                var message = Encoding.UTF8.GetString(body);
                MessageReceived.Invoke(message);
            };
            channel.BasicConsume(queue: "user-queue",
                                 autoAck: true,
                                 consumer: consumer);

            while (!cancellationToken.IsCancellationRequested)
            {
                await Task.Delay(1000);
            }
        }
    }
}
```

Note that in the above snippet, the hostname and queue name are hard coded however these should normally come from config.

In _Imports.razor, add the following two using statements:

```csharp
@using Blazored.Toast
@using Blazored.Toast.Services
```

Within Program.cs, add the a using statement for `Blazored.Toast`:

```csharp
using Blazored.Toast;
```

Then, at the end of the other service registrations, add the following:

```csharp
builder.Services.AddBlazoredToast();
```

Next, within Shared/MainLayout.razor, add the following three lines at the top of the page:

```csharp
@implements IDisposable
@inject IToastService toastService

<BlazoredToasts />
```

Next, add or update the `@code` section with the following content:

```csharp
@code {
    CancellationTokenSource messageConsumerCancellationToken = new();

    protected override void OnAfterRender(bool firstRender)
    {
        if (firstRender)
        {
            Messaging.RabbitMQ.MessageReceived += async (receivedMessage) => toastService.ShowInfo(receivedMessage);

            new Messaging.RabbitMQ().Setup(messageConsumerCancellationToken.Token);
        }
    }

    public void Dispose()
    {
        messageConsumerCancellationToken.Cancel();
    }
}
```

This is set to run in `OnAfterRender` rather than `OnInitialize` or `OnInitializeAsync` so that any messages currently in the queue are displayed without needing to use any kind of caching.

Finally, in Pages/_Layout.cshtml, add a reference to the Blazored.Toast CSS above the css/site.css reference:

```html
<link href="_content/Blazored.Toast/blazored-toast.min.css" rel="stylesheet" />
```

## Try it out!

Now go to [http://localhost:15672/#/queues](http://localhost:15672/#/queues) (log in if prompted) and notice there’s not “user-queue” queue yet. Now run the application and the new queue will appear in the admin UI. Click on it and scroll down to the “Publish message” section. Type in some text and click the “Publish message” button and quickly switch back to your app and a toast notification should be visible in the top right.

If the menu is obscuring the toast, add the following to the bottom of wwwroot/css/site.css:

```css
.blazored-toast-container {
    z-index: 999;
}
```