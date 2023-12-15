# Pulling

Let's get out hands dirty and find an image we might want to run.

First things first, we need to search for the image, for which we have a few options. We can either go to an official container image registry or we can search for it via the CLI:

> Note: If you are not yet authenticated, it might be requested, and if so just run `docker login` and follow the steps

```sh
$ docker search debian
NAME                                  DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
ubuntu                                Ubuntu is a Debian-based Linux operating sys…   16679     [OK]
debian                                Debian is a Linux distribution that's compos…   4877      [OK]
neurodebian                           NeuroDebian provides neuroscience research s…   105       [OK]
```

We have searched for a debian image but we've gotten a few results, how do we know which one to choose?

```sh
$ docker search --format "{{.Name}}: {{.StarCount}}" --filter stars=5 debian
ubuntu: 16679
debian: 4877
neurodebian: 105
kasmweb/debian-bullseye-desktop: 6
pihole/debian-base: 5

```

In this example we are searching for debian images that have at least 5 stars so that we can be more certain we are getting the proper image. However, searching via the Docker CLI is not the best, since we can't see the tags or change the registry we query in the command. In this case, podman has a better search command:

```sh
$ podman search --limit 5 --list-tags --compatible quay.io/prometheus/prometheus
NAME                           TAG
quay.io/prometheus/prometheus  v2.5.0
quay.io/prometheus/prometheus  v2.6.0-rc.0
quay.io/prometheus/prometheus  v2.6.0-rc.1
quay.io/prometheus/prometheus  v2.6.0
quay.io/prometheus/prometheus  v2.6.1

```

Now that we've seen that the dockerhub registry has a debian image, let's pull the most recent version:
`docker pull debian:latest`.

After the download is finished, we can validate we have the expected image by doing:
`docker images | grep debian`.

```sh
$ docker images | grep debian
debian                                              trixie-slim      bfeffe4da138   3 weeks ago    98.1MB
debian                                              latest           a588e7890234   3 weeks ago    139MB
```

Here we see that we have multiple images present. Try and pull the other one.

## Running

To run a container is very simple:
```sh
$ docker run --help

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
...
...

# Let's run the most basic example
$ docker run hello-world
```

Did you notice? When running a container, if the image is not found locally, docker will try to find one in the remote registry and pull what it thinks is the correct one.

Now let's interact enter a container with: `docker run -it ubuntu /bin/bash`

Running in interactive mode let's us "enter" the container and play around. This can be very useful for debugging purposes. But what if I just to want to run the container in the background? Let's try with a load balancer

```
$ docker pull traefik:2.10
...
...

# Let's check the image ID 
$ docker images traefik:v2.10 --format "{{.ID}}"
cb72c2dc074a

# Now let's run it
$ docker run -d -p 80:80 -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock --name reverse-proxy cb7 --api.insecure=true --providers.docker
```

When running a container image there are multiple ways to specify the image we want to execute. In this case I used the first 3 characters of the image ID which is a very standard way to do this locally, however be familiar with work with tags, which would change the command from `cb7` to traefik:v2.10

There are multiple options when running containers. Play around with the debian image and the *exec* command to see if you can figure out to print to *stdout* the phrase: "platypus are real life pokemon". 

**Hint:** Exec is used for containers that are already running.

<details>
<summary>Solution</summary> 

```sh
$ docker run --name mycontainer -d -i -t alpine /bin/sh
$ docker exec -it mycontainer echo "platypus are real life pokemon"
```

</details>


## Resources

[Docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
[Docker run](https://docs.docker.com/engine/reference/commandline/run/)