<p align="center">
  <a href="http://www.uisleuth.com">
    <img src="docs/images/logo.png" width="128" height="128">
  </a>

  <h3 align="center">UI Sleuth</h3>

  <p align="center">
    A Xamarin.Forms Inspector
    <br>
    <br>
    <a href="https://uisleuth-downloads.herokuapp.com/">Desktop Client Download</a>
    &middot;
    <a href="https://www.nuget.org/packages/UISleuth/">NuGet Package Download</a>
    &middot;
    <a href="https://github.com/michaeled/uisleuth/wiki/Installation-Guide">Installation Guide</a>
    &middot;
    <a href="https://github.com/michaeled/uisleuth/wiki">Wiki</a>
  </p>
</p>
<br>

# What is UI Sleuth?

UI Sleuth is a Xamarin.Forms debugging tool. If you’ve ever made a web site, it’s similar to Microsoft’s F12 tools or Chrome Developer Tools. You can use it to efficiently track down layout issues, prototype a new design, and remotely control a device.

# What can you do with this code?

UI Sleuth could _easily_ be more than a "UI Inspector." I can see the .NET library powering best practice analyzers, UI automation engines, remote app viewer services, and more. If you want to ditch the electron desktop client, go for it. Create a new, custom app that communicates with your app server. 

## How can it do that?

The desktop client communicates with your mobile app via WebSockets and a simple JSON protocol. Once you understand that, you know that the desktop client and mobile app aren't irrevocably tied together. 

The library that you add to your Xamarin.Forms application is a workflow engine. It's implemented using a `BlockingCollection` that dispatches incoming messages to listeners, called `Reaction`s. Incoming message types are directly associated to a reaction. When the corresponding reaction is determined, its instantiated and invoked. The implementing reaction can read the incoming message, take an action, and return a response. 

Here's an example.
Below is the code handles the "Take Screenshot Request" from the desktop client.

### 1) We define the request and response types

```
namespace UISleuth.Messages
{
    internal class ScreenShotRequest : Request {}

    internal class ScreenShotResponse : Response
    {
        public byte[] Capture { get; set; }
    }
}
```

### 2) We create a custom reaction class

```
    internal class ScreenShotReaction : Reaction
    {
        protected override void OnExecute(UIMessageContext ctx)
        {
            var request = ctx.Get<ScreenShotRequest>();
            if (request == null) return;

            var screenshot = InspectorContainer.Current.Resolve<IScreenShot>();
            byte[] capture = null;

            Thread.Invoke(() =>
            {
                capture = screenshot.Capture();
            });

            ctx.SetResponse<ScreenShotResponse>(r =>
            {
                r.Capture = capture;
                r.Suggest<GetVisualTreeRequest>();
            });
        }
    }
```

### 3) We associate message request type to the reaction

```
Reaction.Register<ScreenShotRequest, ScreenShotReaction>();
```

## Why WebSockets?

I chose WebSockets because at the beginning of this project project, Xamarin.Forms was a UI toolkit for mobile apps. Now that we're seeing Xamarin.Forms target WPF, GTK#, and macOS a whole new level of possibilites are emerging. So, let's say you no longer want to use IPC instead of WebSockets for out-of-process communication. That's great; start by extending the `InspectorSocket` type.

# Documentation

This project site is a work in progress. 
I currently have minimal documentation explaining the API. However, the majority of the wiki pages I write in the beginning will cover the workflow engine.

You can find all the documentation on the [project's Wiki](https://github.com/michaeled/uisleuth/wiki).

Enjoy!
[@mykldavis](https://twitter.com/mykldavis)
