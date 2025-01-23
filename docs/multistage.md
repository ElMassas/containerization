# Multi-stage builds

Like with many things in tech, it's easier to explain using cooking analogies.

![plate](_media/hamburguer_plate.jpeg)

When we're finished cooking we need somewhere to place the cooked meal and present it, perhaps a plate. What we don't do is use the plate as a cutting board or a heating appliance. We develop the meal in the kitchen using the proper utensils and we deploy the meal to the table in a plate. A container image should follow the same procedure, first we package the application using the best development practices and the required tools, and then we deliver said package without all the tools required to develop it, making sure the application runs as expected.

---

Multi-stage dockerfile:

```multi.Dockerfile
# STAGE 1 (The kitchen)
FROM python:3.13-slim AS kitchen

WORKDIR /app
COPY pyproject.toml /app/
COPY src/ ./src/

# Install necessary packages
RUN apt-get update && \
    apt-get install -y git --no-install-recommends --no-install-suggests && \
    pip --no-cache-dir install --upgrade pip && \
    pip --no-cache-dir install poetry && \
    poetry config virtualenvs.create false && \
    poetry build -f wheel

# STAGE 2 (the plate)
FROM python:3.13-slim

WORKDIR /app
COPY --from=kitchen /app/dist/ /app/

RUN pip --no-cache-dir install --upgrade pip && \
    pip --no-cache-dir install app*.whl

ENTRYPOINT ["app"]
```

> **Note:** We can largely ignore what type of application we are building in this dockerfile, what matters is not the meal being cooked but how we cook it.

Let's break it down.

### Stage 1

In this stage, let's call it the kitchen (`FROM python:3.13-alpine AS kitchen`). We are:

 1. Ensuring we are working in the proper station `WORKDIR /app`
 2. Ensuring we have all the right ingredients `COPY <FROM> <TO>`
 3. We are cooking the meal `RUN <INSTRUCTIONS>`

This essentially finishes whatever may be required in the kitchen, we are finished building the application(`poetry build -f wheel`) and it can leave the kitchen.

### Stage 2
Now that we have finished cooking, all we have to do is plate it. If this was done in a professional kitchen, in this stage would be when the [Sous chef](https://www.select.co.uk/the-hierarchy-of-the-professional-kitchen) receives the food(packaged application) and plates it. Steps:

 1. The Sous chef has their own work environment `FROM python:3.13-alpine`
 2. Retrieves the cooked meal `COPY --from=kitchen /app/dist/ /app/`
 3. Plates the food `RUNS <INSTRUCTIONS>`

 Now, obviosuly I skipped quite a bit of information to get the point accross, but much like with cooking specific meals, they all have their specifics instructions that are more worthy of highlighting when we're discussing the meal in itself, now when discussing how a kitchen works.

## Why use Multi-stage builds

There are plenty of reasons to use this approach, from reduced security risks, reduced image size, and we'll go through each of them but at a conceptual level, why do we use them? If we go back to the kitchen and meal analogy. When we go to a restaurant and ask for some a hamburguer, we don't expect to be served the hamburguer along with all the discarded bones and fat of the pork used to mince the meat, it would unecessary, poor visual presentation and perhaps unsanitary. Comparing a container image that didnn't use multiple stages to be built with one that was illustrates this perfectly:

```standard.Dockerfile
FROM python:3.13-slim

WORKDIR /app
COPY pyproject.toml /app/
COPY src/ ./src/

RUN apt-get update && \
    apt-get install -y git --no-install-recommends --no-install-suggests && \
    pip --no-cache-dir install --upgrade pip && \
    pip --no-cache-dir install poetry && \
    poetry config virtualenvs.create false && \
    poetry build -f wheel && \
    pip --no-cache-dir install --upgrade pip && \
    pip --no-cache-dir install dist/app*.whl

ENTRYPOINT ["app"]
```

This is a standard non Multi-stage build Dockerfile.
What are the differences when we build both these images?

```sh
$ docker build -f multi.Dockerfile -t multi .
Successfully tagged multi:latest
3063de50e8fbe91e2d67246d54dc33e6a3fd6e0aacf632bf42b7121b143c0622

$ docker build -f standard.Dockerfile -t standard . 
Successfully tagged localhost/standard:latest
13fc2232ba40f833d5950fb635618a71098771d960fa1d608c431191ce4afa3d

$ docker images
multi                          latest         3063de50e8fb  5 minutes ago   190 MB
standard                       latest         13fc2232ba40  7 minutes ago   344 MB
docker.io/library/python       3.13-slim      5e458cdbc245  7 weeks ago     151 MB
```

As we can see the multi stage image(190MB) is smaller than the standard(344MB) one, while the base `python:3.13-slim` image is only 151MB.
While the standard.Dockerfile has the source code, the git and poetry applications, and the app.whl package on top of the `python:3.13-slim`. The multi.Dockerfile image only has the app.whl package on top of the `python:3.13-slim` 

## Layers

Now's a good time to learn about image layers.

Basically, a layer, or image layer is a change on an image, or an intermediate image. Every command you specify (FROM, RUN, COPY, etc.) in your Dockerfile causes the previous image to change, thus creating a new layer. You can think of it as staging changes when you're using git: You add a file's change, then another one, then another one...

Let's inspect our standard and multi-stage images:

```sh
$ docker inspect multi | jq '.[0].RootFS.Layers' 
[
  "sha256:6f51610e98184af12bb1cfab40f1ae13928363f1ea9aa19138467f09167e9809",
  "sha256:72f6ab152cffc568bd33f1808ee0c9b4a0bf5ede540fb297595fa5ba51e32ad1",
  "sha256:ca1f7875bf4fa39d9508b480cf5bf4968633aa0ad6c25db322638f28f38e1d5b",
  "sha256:64b9c49a119d20636fe22470679e74e0327a8d4b302a6273a64683d2f4304f93",
  "sha256:fc4855503bc574a09f512aac67481f525f7d2c28cf08fda87d1691551bc5e2a5",
  "sha256:5e486dffcaf081d4afb963c03c58686ba8778a9f1f1db7dddc58bdafcfdaeb5a"
]
# 6 layers


$ docker inspect standard | jq '.[0].RootFS.Layers'          
[
  "sha256:6f51610e98184af12bb1cfab40f1ae13928363f1ea9aa19138467f09167e9809",
  "sha256:72f6ab152cffc568bd33f1808ee0c9b4a0bf5ede540fb297595fa5ba51e32ad1",
  "sha256:ca1f7875bf4fa39d9508b480cf5bf4968633aa0ad6c25db322638f28f38e1d5b",
  "sha256:64b9c49a119d20636fe22470679e74e0327a8d4b302a6273a64683d2f4304f93",
  "sha256:2c9df90e12cf656a196bc16d6980edc385ffc155d62ac53339891a87fbc75567",
  "sha256:386212575300d02c13c373a746db95979784acbe03c086ff1cecfd6c4631803d",
  "sha256:8050064683d9624d0da9e4ba4dd8b5bcd69abee3c1442be652241e66782bf3fe"
]
# 7 layers
```

What would happen if we add another instruction to our standard dockerfile?
```standardV2.Dockerfile
FROM python:3.13-slim

WORKDIR /app
COPY pyproject.toml /app/
COPY src/ ./src/

RUN apt-get update && \
    apt-get install -y git --no-install-recommends --no-install-suggests && \
    pip --no-cache-dir install --upgrade pip && \
    pip --no-cache-dir install poetry && \
    poetry config virtualenvs.create false && \
    poetry build -f wheel && \
    pip --no-cache-dir install --upgrade pip && \
    pip --no-cache-dir install dist/app*.whl

RUN echo "New useless image layer"

ENTRYPOINT ["app"]
```

Let's inspect this new image:
```sh
$ docker inspect standardV2 | jq '.[0].RootFS.Layers | length'
8 
# 8 layers
```

Layers don't automatically increase image size, it depends what we do in those layers. But it does increase the amount of intermediate steps the container engine has to go through to build the image, which can both be good for caching and bad for build times. However, as the newly added `RUN` statement above shows, it's useless, and it's usually smarter and easier to maintain if we aggregate commands. Example:

```sh
# Less optimal (more layers):
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2
RUN pip install requirement1
RUN pip install requirement2

# More optimal (fewer layers):
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    pip install requirement1 requirement2 && \
    rm -rf /var/lib/apt/lists/*
```
