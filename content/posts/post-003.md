---
title: "NEWWW"
date: 2022-08-15T17:47:46+05:30
draft: false
---

Docker images are great, they are easy to work with — but they can get blown up to huge sizes when you are not paying attention on building them right.

Apart from just the size, this can also end up bringing a serious security vulnerability to your production Kubernetes cluster just because you had a package that you never used.

Section 3 is a must read if you have a large python app docker image that needs to be optimised and made smaller.

![[Image Credit](https://images.pexels.com/photos/225769/pexels-photo-225769.jpeg?auto=compress&cs=tinysrgb&dpr=2&h=750&w=1260): *Markus Spiske*](https://cdn-images-1.medium.com/max/11520/1*cAx3fzADLYjc8ufALCoQew.jpeg)*[Image Credit](https://images.pexels.com/photos/225769/pexels-photo-225769.jpeg?auto=compress&cs=tinysrgb&dpr=2&h=750&w=1260): *Markus Spiske**

This article will help you write smaller docker images that will lead to more efficient and secure docker containers.

**Buckle up ! — We are going to be reducing the docker image size from a whopping 536MB to 263MB and then finally to 65.8MB.**

The application that we are dockerizing here is a simple flask application that runs on python with these [requirements](https://github.com/sanjayBala/flask-docker-example/blob/master/requirements.txt).

## 0. Getting Ready

Assuming you have a recent version of docker installed on your computer, do the following

<iframe src="https://medium.com/media/4c714e7fb0b59ca0a2d09440e2654c34" frameborder=0></iframe>

You can also build, and run the docker containers to test out the application

<iframe src="https://medium.com/media/ea46a4739cee25ba56b99bc0c3e7fec7" frameborder=0></iframe>

When the container is running, you should see a simple page like so.

![](https://cdn-images-1.medium.com/max/2364/1*OG0r_1WYClfNX57BCAFd5w.png)

## 1. Docker — the “wrong” way

We are going to use a *naive* approach — a Debian image.

Even this dockerfile has something good going, ***there is only one RUN command***, just one layer by chaining all the shell commands you want to run.

<iframe src="https://medium.com/media/9176e4826b02854faf717fc9afb7af84" frameborder=0></iframe>

There is nothing wrong with this Dockerfile, our application works great and is easy to understand. Although the size of this image is 500MB+, which can definitely grow exponentially when we install more pip packages.

<iframe src="https://medium.com/media/bde415179c331ef94bf59760be916d05" frameborder=0></iframe>

To fix this, we could choose an alternative docker image to build from, perhaps an alpine based image, which has the least number of pre-installed packages and so makes the image as smaller.

Which brings us to our next section…

## 2. Using the right image

Choosing the python:3-alpine image(python 3 is pre-installed here so less work), leads us to minor changes to the dockerfile, like adding build tools g++, etc since that does not come shipped with the alpine image and then doing the actual pip install.

<iframe src="https://medium.com/media/f94e60991821627a4fc652d808a21501" frameborder=0></iframe>

This reduces the image size to 263 MB, which is great, but the alpine image was only 45 MB to start with. So we added about 200MB of packages.

<iframe src="https://medium.com/media/a182b216dc19b99d3c6eced52f7b87ec" frameborder=0></iframe>

## 3. Leveraging multi-stage builds

Taking it one step further, thinking about our situation for a minute — all we need is just the pip packages, not the gcc/g++ build dependencies.

Thankfully, there is a way to use only the files we need and push them into another image — Enter Multi-stage builds.

Multi-stage builds are a nifty way to build an application’s packages in a container, and then copy only the files/artifacts/binaries to the final container without the dependencies — this helps us make our image size a lot lot smaller!

<iframe src="https://medium.com/media/5fc264bb8fde4d6199982a6427332cdc" frameborder=0></iframe>

This Dockerfile essentially makes two “containers”,

### The first container (alias builder)
The first one installs the build dependencies, but this time we install all the pip requirements to a python virtual environment directory /opt/venv .

### The second and final container
The second container is fresh from the alpine image, and only copies the /opt/venv directory which contains all the pip packages our application needs without any other build-time dependencies.

    # this copies the virt-env folder from the first container into the second
    COPY --from=builder /opt/venv /opt/venv ENV 

    # this directs the final container to use the /opt/venv packages
    PATH="/opt/venv/bin:${PATH}"
> The second container is what makes the final image and is shipped to your Docker Image Registry, with no binaries or build tools — just the bare minimum needed to run your application.

***This method reduces the image size to an amazing 65MB, when we started at 500+ MB, this looks like a great leap.***

<iframe src="https://medium.com/media/cc14102d9936096d2b13dfa73d7b92ff" frameborder=0></iframe>

## Summing Up

Multi-stage builds can be used even when building from source, and are not only limited to python virtual-environments.

Just a few simple practices to follow:

* Pick the right image

* Only install the packages your application absolutely needs

* Use multi-stage builds if possible

* Always use a .dockerignore (this makes sure that unwanted files are not copied into the docker image, like zips example)

* Always try to use the least number of RUN directives in the Dockerfile

## Resources

Code can be viewed here: [Github Code Link](https://github.com/sanjayBala/flask-docker-example)
