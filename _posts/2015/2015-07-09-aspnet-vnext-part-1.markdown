---
layout: post
title: "ASP.NET vNext Part 1/n: Deploy a Web Api Locally"
date: 2015-07-09 +0100
comments: true
categories: [example]
---

Welcome to the first post in the series about ASP.NET vNext beta5.
Let's take at the steps we want to fulfill:

  1. Deploy a web api locally
  2. Deploy a web api to a local Docker machine
  3. Deploy a web api to [DigitalOcean](https://www.digitalocean.com/) (using a Docker Container)
  4. Upload a Docker Container to a private Docker Container Repository (Tutum) and deploy it
  5. Deploy a web api with dependencies to other Docker Containers
  6. Use GitHub + Travis for a continuous integration
  7. ???
  
For this Post you need Visual Studio 2015 RC installed and ASP.NET vNext beta5.
To update your ASP.NET runtime you need to run ```dnvm upgrade``` in your console.
  
The full source code can be found in [this repository](https://github.com/Phmager/ASP.NET-Deployment-Example).
  
First we create a new ASP.NET Web API Project:    
![](https://cloud.githubusercontent.com/assets/9350951/8603897/49825d94-267b-11e5-9321-31725e3fe7df.png)

You will see, that a very basic web api project has been created. 
Now we need to change the project to beta5, if it isn't already. 
Open the project.json and change all dependencies to beta5.
Right-click the Project->Properties->Application and see, that the DNX version is beta5.
The build number afterwards should not be important.

![](https://cloud.githubusercontent.com/assets/9350951/8604165/0f25e54c-267d-11e5-8c18-0f2b98924176.png)

Next, to enable us to view the environment variables of the running system, we will first create a new controller in the controllers folder.
You can just use the ValuesController and refactor it to the name InfoController. Change the Route-Attribute to "" and remove all Methods.
We will have two Methods, one for showing the environment variables and one to show the current server time.
This allows us to see, if the server is running and online.

To get the server time, we can use the nice new C# 6 Syntax. (For reference see [here](https://github.com/dotnet/roslyn/wiki/New-Language-Features-in-C%23-6).)

```csharp
[HttpGet]
public string Get()
{
    return $"The server time is: {DateTime.UtcNow}";
}
```

Next we create another Method, which will return the environment variables.

```csharp
[HttpGet("environment")]
public JObject GetEnvironmentVariables()
{
    var variables = Environment.GetEnvironmentVariables();
    var result = new JObject();
    foreach (DictionaryEntry variable in variables)
    {
        result[variable.Key] = variable.Value?.ToString();
    }
    return result;
}
```

When we hover over **GetEnvironmentVariables** there is a warning, that it is not available for DNX Core 5.0. 
This doesn't matter to us.

![](https://cloud.githubusercontent.com/assets/9350951/8604388/97e0a5a6-267e-11e5-990b-504f37694185.png)

Before hitting F5 you should probably change the Properties/launchSettings.json file to change the launchURL to "". 
This will display the server time when you start.
Now you can test both of your endpoints.
When you call **/environment** you will see the environment of your local machine. 
Currently it is not very readable, as the JSON is not indented.
We can change this by opening *Startup.cs* and changing the line `services.AddMvc()` to

```csharp
services.AddMvc().Configure<MvcOptions>(options =>
{
    var formatter = options.OutputFormatters
                           .OfType<JsonOutputFormatter>()
                           .FirstOrDefault();
    if (formatter != null)
        formatter.SerializerSettings.Formatting = Formatting.Indented;
});
```
It is changing the MvcOptions on Startup to use indented formatting for JSON.

Now when you take a look at the **/environment** endpoint you will see the JSON in a readable manner.

Until now we ran the web api locally with IIS:
![](https://cloud.githubusercontent.com/assets/9350951/8604691/6e73edca-2680-11e5-8f27-ec413fb73d54.png)
This can be changed to web. 
When running it with web, a console will open and it will be self hosted by it. 
This time it will be hosted on port 5000 of localhost.
You can see this setting when looking at project.json under commands:

```
Microsoft.AspNet.Hosting 
  --server Microsoft.AspNet.Server.WebListener 
  --server.urls http://localhost:5000
```

Last we will do one more thing and create a command called kestrel, which will start our web api with the Kestrel server.
For it we must add the following command:

```
 "kestrel": "Microsoft.AspNet.Hosting --server Kestrel --server.urls http://localhost:5001"
```

and this dependency:

```
"Kestrel": "1.0.0-beta5"
```
 
Save the file and we see the command added to the run menu. This will start the web api similar to the web command as a console.
 
![](https://cloud.githubusercontent.com/assets/9350951/8604866/7d29b970-2681-11e5-852f-edcb938c434a.png)

Well done, we have achieved hosting a asp.net locally using IIS, Standalone and Kestrel.
Further we adjusted the ASP.NET configuration and used some new C# language feature.

Next we will see, how to deploy the application to a local Docker Machine running in VirualBox.