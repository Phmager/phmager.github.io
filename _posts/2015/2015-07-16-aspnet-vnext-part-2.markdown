---
layout: post
title: "ASP.NET vNext Part 2/n: Deploy a Web API to a local Docker machine"
date: 2015-07-16 +0100
comments: true
categories: [ASP.NET, vNext, Docker, Docker Machine]
---

{% include outlines/2015-07-09-aspnet-vnext-outline.markdown %}

Last week we talked about getting a very simple Web API running on a local host using different servers (IIS Express, standalone and kestrel).
This week we deploy the API with Docker to a local virtual machine.

We have to do the following steps:

1. Setup our environment
2. Create a server/node
3. Deploy the application

# Setup

First off the needed tools have to be installed:

0. (Optional) ConEmu
1. Docker
2. Docker Machine
3. Visual Studio 2015 RC Tools for Docker - Preview

## ConEmu
I know, you and I, we come from a windows background and we usually try to avoid using the console as much as possible.
For the use of Docker it is currently not possible.
This should not be a big problem, as it tends to be very straightforward, how to use the Docker commands via console.
When we are finished with the series, you won't have to use it very often.

As the windows console (CMD) is very bad regarding copy, paste, sizing and so on, 
I will use ConEmu to make the usage of the console more easy.

You can download it [here](https://conemu.codeplex.com/).

## Docker
Next we need Docker.
They describe their platform as the following:

> An open platform for distributed applications for developers and sysadmins.
> Docker allows you to package an application with all of its dependencies into a standardized unit for software development.

So what does that mean? Basically Docker enables the containerized and standardized deployment of applications, 
services and dependencies to any server or system.
Each application is herby reduced to a container, which can use or can be used by other containers.
The container itself can then be run on any OS which is supported by Docker. Currently Windows is not being supported :(. **But** docker can be run using a virtual machine.
This is what boot2docker is for. It installs amongst other things VirtualBox for running a linux distribution.

Follow the instructions for the docker [homepage](https://docs.docker.com/installation/windows/) for the installation.
Using boot2docker is very easy.

If nothing is happening, when you launch boot2docker, then you can open VirtualBox and try to start the virtual machine **boot2docker-vm** manually.
If it is still not working, try to delete it and start boot2docker again.
If you get some error about VT-x, you might need to disable the Hyper-V windows feature. (Follow [this](http://www.eightforums.com/tutorials/42041-hyper-v-enable-disable-windows-8-a.html) tutorial.)

## Docker Machine
I also recommend installing Docker Machine, as it allows the easy creation of servers/nodes for your application. 
It streamlines the process and you don't have to fiddle around with Client/Server certificates, endpoints and so on.

Follow the instructions [here](https://docs.docker.com/machine/).
We will install it in the following way:
Run the boot2docker.lnk in *ConEmu* and then the following command:

```
curl -L https://github.com/docker/machine/releases/download/v0.3.1-rc1/docker-machine_windows-amd64.exe > /bin/docker-machine
```

When you run boot2docker, you automatically connect to the Docker server and can use it "natively" in this console, as all environment parameters are set.
Running `curl ...` will install docker-machine and you will be able to use it, when connected to the Docker server.

## Visual Studio 2015 RC Tools for Docker - Preview
Last we use the [Tools for Docker](https://visualstudiogallery.msdn.microsoft.com/6f638067-027d-4817-bcc7-aa94163338f0), for deploying our ASP.NET Web API to any server. 
It makes it easy to create a working Dockerfile and a deployment script. 
A Dockerfile is some kind of configuration for the application 
(e.g. which storage drive is available, which environmental variables are defined or which ports are used by our container.).

# Server/Node/Machine Creation
Now we create a node on which we want to run our application.
In this part of the tutorial, this will be a local node, running in VirtualBox.
In the next part we will do a similar creation, only in the cloud.
So let's get started.

First start boot2docker. I usually start it by just dragging the boot2docker.lnk to *ConEmu* and run it. 
This automatically creates/starts the virtual machine on which docker is running and also sets all corresponding environment variables.
This allows the locally installed docker windows client to communicate with the docker virtual machine.
It might sound very complicated, but this is all hidden and we will not need any knowledge about that.

![](https://cloud.githubusercontent.com/assets/9350951/8727417/dc921912-2bdf-11e5-86c7-8c92cc567289.png)

In the startup sequence you can see the Docker Host IP. This might be usefull later, so note it. In my case it was: `192.168.59.103`

To check if you are connected, you can then run `docker info`.

When you get an error like the following:

```
An error occurred trying to connect: Get https://192.168.59.103:2376/v1.19/info: x509: certificate is valid for 127.0.0.1, 10.0.2.15, not 192.168.59.103
```

You can just delete the boot2docker-vm from VirtualBox and restart boot2docker. Then `docker info` should work as expected.

Running `docker-machine ls` should display an empty list:
![](https://cloud.githubusercontent.com/assets/9350951/8727733/8d8d6220-2be1-11e5-9c2a-ca2671ec4b1f.png)

To create a node in VirtualBox you usually need to run this command: <strike>docker-machine create -d virtualbox dev</strike>.
It currently does not work on Windows and seems to be some kind of bug, which hangs the console. 
(By the way, you can cancel a command by using `ctrl + c`.)
So we run the following command, which specifies the hostonly-cidr of VirtualBox. 
This allows us to avoid that bug.
Use the IP you previously saw and add the mask /24:

```
docker-machine create -d virtualbox -virtualbox-hostonly-cidr "192.168.59.103/24" dev
```

![](https://cloud.githubusercontent.com/assets/9350951/8729630/8ca73e3e-2bec-11e5-9af1-c72fdc61384d.png)

After a while, the node has been created. This should not take longer than a couple of minutes.

To show the environment for the node called **dev** run:

```
docker-machine env dev
```
![](https://cloud.githubusercontent.com/assets/9350951/8733548/97f2879a-2c04-11e5-8a9a-01e83252adef.png)

# Deployment
Now, that we created our node, we start our project from [Part 1](/archive/2015/07/09/aspnet-vnext-part-1/) in Visual Studio 2015.

Right-click the project in VS and select **Publish**. 
If *Tools for Docker* were installed correctly, you should be able to see **Docker Containers** as publish target.

![](https://cloud.githubusercontent.com/assets/9350951/8733627/0bc16c2c-2c05-11e5-819e-13495664d033.png)

Choose it and select **Custom Docker Host** -> **Ok**. Now you need to fill out the fields in the following way:

|  Field 			| Value  	
|---				|---
| Server Url	 	|  tcp://192.168.59.101:2376 
| Image Name 		| sharp/webapi-deployment-example:test  
| Dockerfile		| *leave this empty*  | 
| Host Port			| 80  |
| Container Port    | 80  |
| Auth Options		| --tls --tlscert="C:\Users\Mosquito\\\.docker\machine\machines\dev\cert.pem" --tlskey="C:\Users\Mosquito\\\.docker\machine\machines\dev\key.pem"

For the **Server Url** and **Auth Options** use the result from the command: `docker-machine env dev`. 
The **Image Name** has to be following Format: [namespace]/[repository]:[tag].
(namespace: optional, only [a-z0-9\_] allowed, repository: only [a-z0-9\_.-] allowed, tag: optional, only [A-Za-z0-9\_.-] allowed)

When you press **Validate Connection**, the connection should work. 
You can now press **Publish** and let the magic happen.
The first deployment can take some time, as the ASP.NET base image has to be downloaded.
Unfortunately it won't work right away, as we changed to ASP.NET **beta5** and the default base image used by *Tools for Docker (Preview)* uses **beta4**.

To change this we need to open the **Dockerfile** in the Folder *Properties/PublishProfiles*.

![](https://cloud.githubusercontent.com/assets/9350951/8734319/93102156-2c09-11e5-9c8d-ec86d163747d.png)

And need to modify the first line to reference the newer beta5. This is the new Dockerfile:

```
FROM microsoft/aspnet:1.0.0-beta5

ADD . /app

WORKDIR /app/approot/src/{{ProjectName}}

ENTRYPOINT ["dnx", ".", "Kestrel", "--server.urls", "http://localhost:{{DockerPublishContainerPort}}"]
```


Run the **Publish** command again (using the same profile) et voila, the server is running and returning the server time.

In Visual Studio we see the following repeated output:

```
...
VERBOSE: Trying to connect to http://192.168.59.101/, attempt 10
VERBOSE: POST http://192.168.59.101/ with 0-byte payload
```

VS is trying to ping the server and launch the webbrowser, when a ping is successful.
Sadly, it uses *POST* requests to check if the server is already online. 
We only implemented the time method for *GET* requests. 
Let's add the following Method to our `InfoController` and publish it again.

```csharp
[HttpPost]
public string Post()
{
    return Get();    
}
```

The deployment should be quite fast, as only the changes have to be transmitted to the server again.
At the second attempt (or maybe third), the *POST* command worked and Visual Studio opens the webbrowser immediately.
Well done, we have our application running on a Linux server on a virtual mashine and a quite easy deployment process.
If you want the container running on another port, you can change the **Host Port** in the Publish settings. 
It's easy as that.


# Conclusion
We first created a new node and then deployed our Web API application to it. 
Of course we only scratched the surface of what is possible with Docker in combination with Visual Studio.
In the next parts we will see, how to deploy our application directly to a cloud hosting provider like [DigitalOcean](https://www.digitalocean.com/).
This will show one of the great advantages of using Docker instead of more traditional deployment strategies.

**I would be happy if you leave comments or feedback down below. See you next week.**