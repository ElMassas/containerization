# Creating

We've seen how to retrieve and run a container image someone else has created but how does one go about creating their own image?

## Dockerfile

As previously stated we require a Dockerfile to list the [instructions](https://docs.docker.com/reference/dockerfile/#overview) that the container engine will execute to assemble the image. Let's go over a really simple dockerfile to run a simple python program.

```Dockerfile
# Use an official Python runtime as a parent image
FROM python:3.11

# Set the working directory in the container to /app
WORKDIR /app

# Install pipx
RUN python3 -m pip install pipx
RUN python3 -m pipx ensurepath

# Add /root/.local/bin to the PATH environment variable
ENV PATH="/root/.local/bin:${PATH}"

# Install cowsay using pipx
RUN pipx install cowsay

# Run cowsay when the container launches
CMD ["cowsay","-t", "Mooo"]

```

To build this image and run it, first create a file named `Dockerfile` in your current project directory and execute this command in your terminal:

`$ docker build -t my_python_cowsay . && docker run my_python_cowsay`

A dockerfile only really has one requirement which is the `FROM` instruction, that's because, without the underlying Operating System that comes with the **base image** you would not have anything. For example you would not be able to sucessfully perform this operation: `WORKDIR /app` because this operation is relying upon the understading that there is a file system with the root of `/`.

Dockerfile instructions to follow this simple syntax: `INSTRUCTION arguments`.

One of the main points of using containers is to standardize development and environments, which means we use containers to package and application. This is where the `COPY`, `WORKDIR`, `RUN` and `CMD/ENTRYPOINT` instructions enter.

The flow for containerizing an application is similar to this:

```Dockerfile
FROM base_image:tag

WORKDIR default_dir_for_instructions_to_start

COPY application_code target_directory_in_container

RUN some_commands_to_build_or_compile_code

CMD ["default_command_to_start_application"]
```

In the previous **my_python_cowsay** dockerfile we didn't `COPY` any application code to the container since we didn't have any, we just pulled some programs from the internet and, added some very ligh configuration with environment variables, and ran the programs inside the container. This is a usual task for building services before deploying them. As developers we wouldn't/shouldn't normally just fetch a service required for our application to work and run it without customizing it our needs.

## Considerations

One thing to note is the size of the image generate by the build command on our Dockerfile. 

```sh
$ docker images
REPOSITORY                                          TAG              IMAGE ID       CREATED         SIZE
my_python_cowsay                                    latest           d5a86f9eb0a8   8 seconds ago   1.05GB
```

You might think *1.05Gb* isn't much but you'd be wrong. There are a lot of moving parts to deploying a service, specially in larger companies with many engineers, so consider the following very real hypothetic of critical failure in system that needs to run in real-time:

<div align="center">
🚨🚨🚨 Incident 🚨🚨🚨
</div>

- Imagine we have a problem in production, which originates in some faulty code, that's packaged in a container image and we don't have a good devops/sre team, that hasn't optimized our CI/CD pipelines and we have no fallback strategy. To patch a fix we might have to go through these steps:
    1. Write code to fix changes
    2. Build and test locally
    3. Code will most likely have to be reviewed
    4. Code will have to pass some tests
    5. Push code to production
    6. Container will have to be built and published in a registry
    7. New version of container has to be published and so the machine running the application will have to be replaced, and in this step, the machine will also have to pull the most recent version of our container image.
- In this imaginary situation we might take somewhere between 30 minutes to several hours to fix a problem, and so it's in our best interest to shorten this time as much as possible. Looking back at the steps, which could honestly be even worse, there are a few steps on which the size of the container might affect the MTR(mean time to recovery), mainly steps: 2,3,4,6 and 7.
  - (2) We might have to build and validate the changes locally several times.
  - (3) Our colleagues might need to do the same.
  - (4) Depending on how the battery of tests at your company is done, the container might have to be re-built in a test environment.
  - (6) This is the most obvious, we build the container to publish it in our registry of choice
  - (7) The time it takes to pull the container image and executing is also affected by it's size, although it's less worrisome in this scenario.

Another very important reason to optimize and shrink your container image is for security purposes. If you have less stuff in your container, there's less stuff to advantage of, so you have a reduced attack surface.
In my case, on my local machine, building from scratch took around 30 seconds. Let's optimize.

## Improvements

There are many aspects to take into consideration, but some quick points to tackle are: Package managers, base images and cache.

We will check how to drastically improve our container images in the following sections, culminating in the best practices section. For now:

```Dockerfile
# Use an official lighter Python runtime as a parent image
FROM python:3.11-alpine

# Install cowsay using pip and without caching
# Normally caching helps with performance, but in this case
# We are only performing this operation once, so there is no
# need to store cache related to this package
RUN pip install --no-cache-dir cowsay

# Run cowsay when the container launches
CMD ["cowsay", "-t","Mooo"]
```

Now check the size of this new container image.

## Multiple architectures

Sometimes you'll have the pleasure of deciding where you want to deploy a service, and you might decide that the CPU architecture you want to deploy is ARM based due to their power efficiency and/or compatability with some service. So you write you're program with this in mind, but you also have to remember to build your container image to run in this architecture.

Nowadays most container engines allow you to build to multiple architectures. Docker has [buildx](https://github.com/docker/buildx) and podman has [buildah](https://github.com/containers/buildah)

It's really simple, here are some examples for you to try:

```sh
$ docker buildx build --platform linux/amd64 --rm -t my_python_cowsay .
$ docker buildx build --platform linux/arm64 --rm -t my_python_cowsay . 
```

## Hands-on

If you want to get more hands-on experience with creating and running containers, take a look at this repository I've created for demonstration purposes. [Link](https://github.com/ElMassas/simple_todo/tree/main/single_component)

## Resources

[Dockerfile documentation](https://docs.docker.com/reference/dockerfile/)
[Even without running a container without an image, you still need an Operatin System](https://iximiuz.com/en/posts/you-dont-need-an-image-to-run-a-container/)
