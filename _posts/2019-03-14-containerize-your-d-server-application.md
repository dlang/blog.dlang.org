---
author: KaiNacke
comments: false
date: 2019-03-14 12:22:43+00:00
layout: post
link: https://dlang.org/blog/2019/03/14/containerize-your-d-server-application/
slug: containerize-your-d-server-application
title: Containerize Your D Server Application
wordpress_id: 1984
categories:
- Code
- Guest Posts
- Tutorials
- Web Development
permalink: /containerize-your-d-server-application/
redirect_from: /2019/03/14/containerize-your-d-server-application/
---

![](http://dlang.org/blog/wp-content/uploads/2017/03/product-icon.png)

A container consists of an application packed together with all of its required dependencies. The container is run as an isolated process on Linux or Windows. [The Docker tool](https://www.docker.com/) has made the handling of containers very popular and is now the de-facto standard for deploying containers to a cloud environment. In this blog post I discuss how to create a simple [vibe.d application](https://vibed.org/) and ship it as a Docker container.


## Setting up the build environment


I use Ubuntu 18.10 as my development environment for this application. Additionally, I installed the packages `ldc` ([the LLVM-based D compiler](https://github.com/ldc-developers/ldc)), `dub` ([the D package manager and build tool](https://dub.pm/getting_started)), `gcc`, `zlib1g-dev` and `libssl-dev` (all required for compiling my vibe.d application). To build and run my container I use Docker CE. I installed it following the instructions at [https://docs.docker.com/install/linux/docker-ce/ubuntu/](https://docs.docker.com/install/linux/docker-ce/ubuntu/). As the last step, I added my user to the docker group (`sudo adduser kai docker`).


## A sample REST application


My vibe.d application is a very simple REST server. You can call the `/hello` endpoint (with an optional name parameter) and you get back a friendly message in JSON format. The second endpoint, `/healthz`, is intended as a health check and simply returns the string `"OK"`. You can clone my source repository at [https://github.com/redstar/vibed-docker/](https://github.com/redstar/vibed-docker/) to get the source code. Here is the application:

```d
import vibe.d;
import std.conv : to;
import std.process : environment;
import std.typecons : Nullable;

shared static this()
{
    logInfo("Environment dump");
    auto env = environment.toAA;
    foreach(k, v; env)
        logInfo("%s = %s", k, v);

    auto host = environment.get("HELLO_HOST", "0.0.0.0");
    auto port = to!ushort(environment.get("HELLO_PORT", "17890"));

    auto router = new URLRouter;
    router.registerRestInterface(new HelloImpl());

    auto settings = new HTTPServerSettings;
    settings.port = port;
    settings.bindAddresses = [host];

    listenHTTP(settings, router);

    logInfo("Please open http://%s:%d/hello in your browser.", host, port);
}

interface Hello
{
    @method(HTTPMethod.GET)
    @path("hello")
    @queryParam("name", "name")
    Msg hello(Nullable!string name);

    @method(HTTPMethod.GET)
    @path("healthz")
    string healthz();
}

class HelloImpl : Hello
{
    Msg hello(Nullable!string name) @safe
    {
        logInfo("hello called");
        return Msg(format("Hello %s", name.isNull ? "visitor" : name));
    }

    string healthz() @safe
    {
        logInfo("healthz called");
        return "OK";
    }
}

struct Msg
{
    string msg;
}
```
And this is the `dub.sdl` file to compile the application:

    name "hellorest"
    description "A minimal REST server."
    authors "Kai Nacke"
    copyright "Copyright © 2018, Kai Nacke"
    license "BSD 2-clause"
    dependency "vibe-d" version="~>0.8.4"
    dependency "vibe-d:tls" version="*"
    subConfiguration "vibe-d:tls" "openssl-1.1"
    versions "VibeDefaultMain"
Compile and run the application with `dub`. Then open the URL `http://127.0.0.1:17890/hello` to check that you get a JSON result.

A cloud-native application should follow the twelve-factor app methodology. You can read about the twelve-factor app at [https://12factor.net/](https://12factor.net/). In this post I only highlight two of the factors: [III. Config](https://12factor.net/config) and [XI. Logs](https://12factor.net/logs).

Ideally, you build an application only once and then deploy it into different environments, e.g. first to your quality testing environment and then to production. When you ship your application as a container, it comes with all of its required dependencies. This solves the problem that different versions of a library might be installed in different environments, possibly causing hard-to-find errors. You still need to find a solution for how to deal with different configuration settings. Port numbers, passwords or the location of databases are all configuration settings which typically differ from environment to environment. The factor [III. Config](https://12factor.net/config) recommends that the configuration be stored in environment variables. This has the advantage that you can change the configuration without touching a single file. My application follows this recommendation. It uses the environment variable `HELLO_HOST` for the configuration of the host IP and the variable `HELLO_PORT` for the port number. For easy testing, the application uses the default values `0.0.0.0` and `17890` in case the variables do not exist. (To be sure that every configuration is complete, it would be safer to stop the application with an error message in case an environment variable is not found.)

The application writes log entries on startup and when a url endpoint is called. The log is written to `stdout`. This is exactly the point of factor [XI. Logs](https://12factor.net/logs): an application should not bother to handle logs at all. Instead, it should treat logs as an event stream and write everything to `stdout`. The cloud environment is then responsible for collecting, storing and analyzing the logs.


## Building the container


A Docker container is specified with a Dockerfile. Here is the Dockerfile for the application:

    FROM ubuntu:cosmic
    
    RUN \
      apt-get update && \
      apt-get install -y libphobos2-ldc-shared81 zlib1g libssl1.1 && \
      rm -rf /var/lib/apt/lists/*
    
    COPY hellorest /
    
    USER nobody
    
    ENTRYPOINT ["/hellorest"]
A Docker container is a stack of read-only layers. With the first line, `FROM ubuntu:cosmic`, I specify that I want to use this specific Ubuntu version as the base layer of my container. During the first build, this layer is downloaded from Docker Hub. Every other line in the Dockerfile creates a new layer. The RUN line is executed at build time. I use it to install dependent libraries which are needed for the application. The `COPY` command copies the executable into the root directory inside the container. And last, `CMD` specifies the command which the container will run.

Run the Docker command

    docker build -t vibed-docker/hello:v1 .
to build the Docker container. After the container is built successfully, you can run it with

    docker run -p 17890:17890 vibed-docker/hello:v1
Now open again the URL `http://127.0.0.1:17890/hello`. You should get the same result as before. Congratulations! Your vibe.d application is now running in a container!


## Using a multi-stage build for the container


The binary `hellorest` was compiled outside the container. This creates difficulties as soon as dependencies in your development environment change. It is easy to integrate compiliation into the Dockerfile, but this creates another issue. The requirements for compiling and running the application are different, e.g. the compiler is not required to run the application.

The solution is to use a multi-stage build. In the first stage, the application is build. The second stage contains only the runtime dependencies and application binary built in the first stage. This is possible because Docker allows the copying of files between stages. Here is the multi-stage Dockerfile:

    FROM ubuntu:cosmic AS build
    
    RUN \
      apt-get update && \
      apt-get install -y ldc gcc dub zlib1g-dev libssl-dev && \
      rm -rf /var/lib/apt/lists/*
    
    COPY . /tmp
    
    WORKDIR /tmp
    
    RUN dub -v build
    
    FROM ubuntu:cosmic
    
    RUN \
      apt-get update && \
      apt-get install -y libphobos2-ldc-shared81 zlib1g libssl1.1 && \
      rm -rf /var/lib/apt/lists/*
    
    COPY --from=build /tmp/hellorest /
    
    USER nobody
    
    ENTRYPOINT ["/hellorest"]
In my repository I called this file `Dockerfile.multi`. Therefore, you have to specify the file on the command line:

    docker build -f Dockerfile.multi -t vibed-docker/hello:v1 .
Building the container now requires much more time because a clean build of the application is included. The advantage is that your build environment is now independent of your host environment.


## Where to go from here?


Using containers is fun. But the fun diminishes as soon as the containers get larger. Using Ubuntu as the base image is comfortable but not the best solution. To reduce the size of your container you may want to try [Alpine Linux](https://alpinelinux.org/) as the base image, or use no base image as all.

If your application is split over several containers then you can use [Docker Compose](https://docs.docker.com/compose/) to manage your containers. For real container orchestration in the cloud you will want to [learn about Kubernetes](https://kubernetes.io/).



* * *


_A long-time contributor to the D community, Kai Nacke is the author of '[D Web Development](https://www.packtpub.com/web-development/d-web-development)' and a maintainer of LDC, [the LLVM D Compiler](https://wiki.dlang.org/LDC)._
