# Orchestration

Many times, we will need to have multiple services and applications running together - the days of generalized monolithic applications are behind us - the applications need to talk to eachother, perhaps they should be in the same network, but they're isolated and packaged into containers, how the hell do we manage to make them work? We will need to be like a conductor and orchestrate the individual pieces.

---

![jw_conduncting](_media/jw_conducting.jpg)

Not everyone will need to bring out their inner John Williams, and be the Superman to their DevOps team. There are many levels of Orchestration and complexity, the higher levels are usually reserved for teams specific to those orchestration engines, but it's always useful to understand how things work and be able to perform the basics locally on your machine so you can try new services, debug and test.

The simplest and most basic tool for conteiner orchestration is:

## Compose

Compose is an abstraction that makes it easier to run multiple containers at the same time, take a look at this example:

```docker-compose.ym
name: ollana_open_webui

services:
  ollama:
    volumes:
      - ollama:/root/.ollama
    container_name: ollama
    pull_policy: always
    tty: true
    ports:
      - 11434:11434
    restart: unless-stopped
    image: ollama/ollama:latest

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    volumes:
      - open-webui:/app/backend/data
    depends_on:
      - ollama
    ports:
      - ${OPEN_WEBUI_PORT-3000}:8080
    environment:
      - 'OLLAMA_BASE_URL=http://ollama:11434'
      - 'WEBUI_SECRET_KEY='
    extra_hosts:
      - host.docker.internal:host-gateway
    restart: unless-stopped

volumes:
  ollama: {}
  open-webui: {}
```

Here we are creating 2 containers. One is a front-end service simillar to chatgpt that allows us to interact with AI models running in [Ollama](https://ollama.com/). To run these containers we simply must run this command in the terminal: `docker-compose up -d`

```sh
❯ docker ps
CONTAINER ID  IMAGE                                 COMMAND        CREATED         STATUS         PORTS                    NAMES
88ccc9a42c1b  ghcr.io/open-webui/open-webui:main    bash start.sh  2 minutes ago   Up 2 minutes   0.0.0.0:3000->8080/tcp   open-webui
f2021140c78e  docker.io/ollama/ollama:latest                       2 minutes ago   Up 2 minutes   0.0.0.0:11424            ollama
```

This *docker-compose.yml* simplifies our life. If we were not using this compose file, we would have to run these containers individually, which although not impossible:
```sh
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
docker run -d --network slirp4netns:allow_host_loopback=true -p 3000:8080 -e OLLAMA_BASE_URL=http://host.docker.internal:11434 -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```
It can get increasingly more complex, and running each container individually becomes highly impractical. Can you imagine running [all these containers](https://github.com/grafana/loki/blob/main/examples/getting-started/docker-compose.yaml) one at a time?


Docker compose eases the pain of configuration, there are many things that can be set up with docker-compose([documentation](https://docs.docker.com/reference/compose-file/build/)), however, you will most likely not have to use much of the available options. Most development teams will most of the time, a docker-compose file very similar to the one above, except maybe with more services, some more service specific configuration and perhaps some networking. Even the network configuration is doubtful, because if you want these containers to be able to comunicate with eachother, you will use the default [network driver](https://docs.docker.com/engine/network/#drivers), that enables exactly that.



## Docker compose alternatives

As you very well know at this point, there are many other alternatives to Docker, like containerd and podman, which have their own compose alternatives:

1. [podman-compose](https://docs.podman.io/en/v5.1.1/markdown/podman-compose.1.html)
2. [containerd-compose](https://github.com/containerd/nerdctl)

At higher levels of complexity with a larger scope and more features there are:

1. [nomad](https://github.com/hashicorp/nomad)
2. [kubernetes](https://kubernetes.io/)
3. [Docker swarm](https://docs.docker.com/engine/swarm/)

There are specific sections for these orchesctration engines further in this workshop.

## Exercises

Test out your knowdledge with this exercise here: [link](https://github.com/ElMassas/simple_todo/tree/main/multiple_components/compose), and this quizz: [link](basicquiz.md)

## Resources

- [Some compose examples](https://github.com/Haxxnet/Compose-Examples/tree/main)
