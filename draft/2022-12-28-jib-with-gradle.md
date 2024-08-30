---
title: Jib with Gradle
date: 2022-12-28 20:00:00 +0900
categories: [SlipBox, Container]
tags: [Gradle, Jib]
---

# What is Jib?
Jib builds optimized Docker and OCI images for your Java applications without a Docker daemon - and without deep mastery of Docker best-practices. 

> The Open Container Initiative is an open governance structure for the express purpose of creating open industry standards around container formats and runtimes. New tools for building container images aimed to improve Docker's speed or ease of use. To make sure that all container runtimes could run images produced by any build tool, the community started the Open Container Initiative — or OCI — to define industry standards around container image formats and runtimes. Given an OCI image, any container runtime that implements the OCI Runtime Specification can unbundle the image and run its contents in an isolated environment.
{: .prompt-info }

# Containerize your Gradle Java project

## Setup

In your Gradle Java project, add the plugin to your `build.gradle`:

```gradle
plugins {
  id 'com.google.cloud.tools.jib' version '3.3.1'
}
```

You can containerize your application easily with one command:

```bash
gradle jib --image=<MY IMAGE>
```

To build to a Docker daemon, use:

```bash
gradle jibDockerBuild
```

## configuration

### Using Google Container Registry (GCR)

- Make sure you have the docker-credential-gcr command line tool.

- To build the image gcr.io/my-gcp-project/my-app, the configuration would be:

```gradle
jib.to.image = 'gcr.io/my-gcp-project/my-app'
```

### Using Docker Hub Registry

- Make sure you have a docker-credential-helper set up

- For example, to build the image my-docker-id/my-app, the configuration would be:

```gradle
jib.to.image = 'my-docker-id/my-app'
```

- It is available as plugins for Maven and Gradle and as a Java library.

## Build Your Image

Build your container image with:

```bash
gradle jib
```

## Build to Docker daemon

- Jib can also build your image directly to a Docker daemon.
- Requires that you have docker available on your PATH.

```bash
gradle jibDockerBuild
```

## Run `jib` with each build

You can also have `jib` run with each build by attaching it to the `build` task:

```gradle
tasks.build.dependsOn tasks.jib
```

## Multi Module Projects

The example project consists of two microservices and a library:

1. `name-service` - responds with a name

2. `shared-library` - a project dependency used by `name-service`

3. `hello-service` - calls name-service and responds with a greeting

- The Gradle project is set up with a parent `build.gradle` that sets some common configuration up for all projects, 
- with each sub-project containing its own `build.gradle` with some custom configuration. 
- `settings.gradle` defines which modules to include in the overall build.



## Extended Usage

to : Configures the target image to build your application to.

- image: The image reference for the target image. This can also be specified via the `--image` command line option. If the tag is not present here `:latest` is implied.

from : Configures the base image to build your application on top of.

- image: The image reference for the base image. e.g. `openjdk:11-jre`, `docker://busybox`

container : Configures the container that is run from your built image.

### System Properties

- Each of these parameters is configurable via commandline using system properties.

- Jib's system properties follow the same naming convention as the configuration parameters, with each level separated by dots

```bash
gradle jib \
    -Djib.to.image=myregistry/myimage:latest \
    -Djib.to.auth.username=$USERNAME \
    -Djib.to.auth.password=$PASSWORD

gradle jibDockerBuild \
    -Djib.dockerClient.executable=/path/to/docker \
    -Djib.container.environment=key1="value1",key2="value2" \
    -Djib.container.args=arg1,arg2,arg3
```

## Example

```gradle
jib {
    from {
        image = "openjdk:alpine"
    }
    to {
        image = "localhost:5000/my-image/built-with-jib"
        <!-- credHelper = 'osxkeychain' -->
        tags = setOf("tag2", "latest")
    }
    container {
        jvmFlags = listOf("-Dmy.property=example.value", "-Xms512m", "-Xdebug")
        mainClass = "mypackage.MyApp"
        args = listOf("some", "args")
        ports = listOf("1000", "2000-2003/udp")
        <!-- labels = [key1:'value1', key2:'value2']
        format = 'OCI' -->
    }
}
```

In this configuration, the image:

- Is built from a base of `openjdk:alpine` (pulled from Docker Hub)
- Is pushed to localhost:5000/my-image:built-with-jib, localhost:5000/my-image:tag2, and localhost:5000/my-image:latest
- Runs by calling `java -Dmy.property=example.value -Xms512m -Xdebug -cp app/libs/*:app/resources:app/classes mypackage.MyApp some args`
- Exposes port 1000 for tcp (default), and ports 2000, 2001, 2002, and 2003 for udp

## Adding Arbitrary Files to the Image

- You can add arbitrary, non-classpath files to the image without extra configuration by placing them in a `src/main/jib` directory.
- This will copy all files within the `jib` folder to the target directory (`/` by default) in the image, maintaining the same structure
> Jib does not follow symbolic links in the container image. If a symbolic link is present, it will be removed prior to placing the files and directories.
{: .prompt-tip }


# Reference
- https://github.com/GoogleContainerTools/jib
- https://github.com/GoogleContainerTools/jib/tree/master/jib-gradle-plugin
