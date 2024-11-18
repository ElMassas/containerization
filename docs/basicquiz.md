# Basics 

Not all the question asked here have been talked about before. Remember this is workshop to get you familiar and teach how to work with containers. **Take this opportunity to research**.

## Question 1

What's the main difference between a Dockerfile and a Docker-compose file?

<input type="radio" name="question1"> Dockerfiles are text files with configurations about a container and docker-compose are used to create container images<br>
<input type="radio" name="question1"> Dockerfiles are text files with instructions on how to create a container image and a docker-compose file is a configuration file for container images<br>
<input type="radio" name="question1"> There is no difference<br>
<input type="radio" name="question1"> Will power<br>

<details>
<summary>Solution</summary> 

> **Dockerfiles are text files with instructions on how to create a container image and a docker-compose file is a configuration file for container images** 

</details>

## Question 2

What language is used to write Dockerfile?

<input type="radio" name="question2"> JSON<br>
<input type="radio" name="question2"> YML<br>
<input type="radio" name="question2"> XML<br>
<input type="radio" name="question2"> HTML<br>

<details>
<summary>Solution</summary> 

> **YML**

</details>

## Question 3

What is arguably the best to search for a container image?

<input type="radio" name="question3"> The docker cli search command<br>
<input type="radio" name="question3"> The podman cli search command<br>
<input type="radio" name="question3"> Quay.io <br>
<input type="radio" name="question3"> Docker hub<br>

<details>
<summary>Solution</summary> 

> **Dockerhub** due to the fact that is the most widely used and so it has the most amount of mainstream services present, and has some very robust security checks.

</details>

## Question 4

What is the correct way to build and push a container image for multiple CPU architectures?

<input type="radio" name="question4"> `docker buildx build --push --platform linux/arm64,linux/amd64 -t my-image/multi-arch:latest`<br>
<input type="radio" name="question4"> `docker buildx build --platform linux/arm64 -t my-image/multi-arch:latest`<br>
<input type="radio" name="question4"> `docker build --pull --platform linux/arm64,linux/amd64 -t my-image/multi-arch:latest`<br>
<input type="radio" name="question4"> `docker build buildx --push --platform linux/arm64,linux/amd64 -t my-image/multi-arch:latest`<br>

<details>
<summary>Solution</summary> 

> ```
docker buildx build --push --platform linux/arm64,linux/amd64 -t my-image/multi-arch:lates```

</details>

## Question 5

What happens if you attempt to perform a Docker build based on an image that has not been previously referenced on your machine?

<input type="radio" name="question5"> Docker will return a 404<br>
<input type="radio" name="question5"> Docker will return a 400<br>
<input type="radio" name="question5"> Docker will attempt to pull the image from Docker hub<br>
<input type="radio" name="question5"> Docker will return a 418<br>

<details>
<summary>Solution</summary> 

> **Dockerhub** due to the fact that is the most widely used and so it has the most amount of mainstream services present, and has some very robust security checks.

</details>