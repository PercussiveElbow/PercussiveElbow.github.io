---
layout: post
title:  "Lightweight Crystal Docker images"
date:   2020-10-01 00:00:00 +0000
categories: posts
---

{:refdef: style="text-align: center;"}
![Crystal Logo]({{ site.url }}/assets/images/posts/crystal/crystal_logo.png)
{: refdef}

I've recently been writing a few scripts/applications of varying sizes and where possible I've been doing so with [Crystal](https://crystal-lang.org/). 

If you aren't familar with Crystal it's a relatively new object-orientated programming language with syntax very similar to Ruby. However, unlike Ruby, Crystal is compiled instead of interpreted and is statically rather than dynamically typed. 

I won't go on a tangent about why Crystal is a really promising language. The tl;dr is it's *much* faster than Ruby and in my opinion encourages more manageable coding practices, particuarily with larger codebases.

Instead I'm going to briefly talk about how we can use Crystal and a lesser-known Docker feature to build nice, lightweight images for our applications.

# Statically Linked Crystal builds

Similar to GoLang, Crystal provides the ability to statically compile our application. This links all required dependencies into our compiled executable binary to make it much more portable, at the cost of a slightly bigger file size. We should then be able to run our application anywhere provided the operating system and processor architecture match.

To instruct the Crystal compiler to statically compile a program we can use the `--static` flag during compilation like so `crystal build --static our_program.cr`. 

One thing to note about Crystal is that it only supports static linking *officially* through the Crystal Alpine Docker image `crystallang/crystal:latest-alpine`. This is because static linking at present [only plays well with musl](https://github.com/crystal-lang/crystal/wiki/Static-Linking) and not glibc. Another thing to note is at the time of writing static builds are not compatible with multithreaded applications - though this should work soon.

![Dockerfile](/assets/images/posts/crystal/dockerfile_1.png)

The above Dockerfile demonstrates a statically compiled build of a Crystal [client](https://github.com/PercussiveElbow/crobat-sdk-crystal) I've previously written for [Crobat](https://sonar.omnisint.io/). This is a straightforward and small program. It pretty much just makes a few HTTP requests to an API, so should be a good candidate for demonstrating just how small we can make our images.


# Multi-stage Docker builds 

One pet peeve I have with Docker is the huge image sizes which are prevelant. This slows down download times, wastes disk space and probably indirectly contributes to killing the dolphins. 

For example, using the official Ubuntu-based Crystal image with our program uses **694MB** which is absolutely huge for such a tiny program. We can slim this down with an Alpine based image, but it's still fairly large. 

There's a trick we can use to reduce Docker image sizes that compliments Crystal's static build option - [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/).

To successfully compile Crystal programs we need a bunch of prerequisites in including the Crystal compiler and all the usual system libraries. Unfortunately this makes our final image with our compiled application fairly large in file size.

We can instead use multi-stage builds to create our final Docker image in two (or more) stages, as shown in the Dockerfile below. One stage can use an appropriate base image to compile our application while the next and final stage should require only the bare necessities needed to run our application.

![Dockerfile](/assets/images/posts/crystal/dockerfile.png)


To start, we can inherit from an image containing all the usual pre-requisites to compile our app. In this case this is the previously mentioned Crystal Alpine image. After installing updates and compiling our application this image weighs in at a pretty hefty 395MB. 


Next we can use `FROM SCRATCH` command to instruct Docker to start with the most minimal container image possible. Using the `--from` flag on `COPY` instructions we can copy our statically compiled binary from the final Alpine layer to our new minimal image layer. 

One final thing we'll need to do is copy any CA certificates from the final builder layer into this lightweight image, otherwise any TLS connectivity in our application will go kaput.

# Image size reductions

Upon comparing our final image sizes the file size reduction is obvious. The normal Crystal Ubuntu-based image with our app uses 694MB, the Alpine image uses 395MB and our tiny `FROM SCRATCH` image ended up using only 16MB (!) - CA certificates and all.  

![Docker image sizes](/assets/images/posts/crystal/docker_images.png)

From just choosing the right base image and swapping around a few Dockerfile commands we've managed to make an image taking up only 2% the size of our Ubuntu image and 4% of the size of our Alpine image.

This will shorten image pull/push times and decrease disk usage. Not only that but our image is now more secure - we don't have dozens of utilities associated with a default Ubuntu based image lying around in our container. Instead we only have our compiled binary and the absolute necessities.

In conclusion, while there can be some initial teething issues trying to create these lightweight images I believe the payoff of smaller, more secure images are really worth it and something I'd recommend trying.