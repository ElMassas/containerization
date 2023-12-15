# Definition

A container is an executable unit of software, in which a single or multiple applications are packaged into with the required runtime environment, so that it can run "anywhere", wether that be **production**, **staging** or even sa **personal machine**.

**But in reality... what does this mean?**

It means computer goes: "brrr..." and spawns a magical new computer inside it self. By this I mean that, much like when using a VM on your machine, a container takes advantage of the hardware resources on your machine, and it produces a piece of software which is isolated from the rest of your machine.

You'll learn more about how containers run in the next page and in the [OCI](OCI.md) sections.

## Container images

To run a container we need both an application to run this piece of software, and an instruction so that the container engine knows what to run. Container images are typically created from text files known as Dockerfiles, which contain the application code, its dependencies, and configuration details.

Container images are immutable, meaning that once they are created they should not be changed. If we want to update the instruction set that the engine will run, we generate a new container image. This ensures consistency and reproducibility across differing environments where the container might run.

## What's the difference between a Virtual machine and a container engine?

Both aim to create isolated environments, however they go about it using different approaches. Virtual machines utilize [hypervisors](https://www.vmware.com/content/vmware/vmware-published-sites/us/topics/glossary/content/hypervisor.html.html) to achieve this goal while containers utilize underlying linux technologies.

![isolation approaches](_media/isolation_approaches.png)

VM's and container engines often work together in the cloud. For instance, most cloud service providers utilize Type1(bare metal) hypervisors on which people run containerized applications.
