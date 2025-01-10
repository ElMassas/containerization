# Pushing && Registries

Think of a registry as a database for your container images. There are many of them out there, and you might even have an internal registry at your company. In general all you have to do, is authenticate yourself if need be, and do a simple `docker push <image>` command.
Some registries:

- [quay.io](https://quay.io/)
- [docker hub](https://hub.docker.com/)
- [aws ecr](https://docs.aws.amazon.com/AmazonECR/latest/userguide/Registries.html)

**Note:** While most major services have container images available in most known container registries, some of them are not, and while you may think that you found the official container image for the service, you could have just found an unofficial mirror, so be mindful of what you download.
