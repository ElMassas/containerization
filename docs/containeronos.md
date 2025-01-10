# Under the hood

<div align="center" style="font-size: 20px;">
Docker != containers
</div>

Containers are a mix of Linux features with some special configurations sprinkled in. Other Operating Systems have similar concepts to the one's we'll discuss in this page, however, due to obiquity of Linux and it being used as the base for container engines even in Windows - Docker for example runs a Linux VM in which containers are ran.

What are these Linux features?
 - **Namespaces**
 - **CGroups**
The answer is **Namespaces**, which isolate a process's view of the underlying system. When a process is run inside a new namespace, it get it's own isolated instance of global resources

## Namespaces

A feature of the Linux Kernel that limits what resources can be accessed by certain processes.
You can learn more about namespaces by reading the [man page](https://man7.org/linux/man-pages/man7/namespaces.7.html): `man namespaces`

For a brief overview of what namespaces contain you can run this command:
`docker run -it --rm ubuntu bash -c "cd /proc/1/ns && ls -l"`

And it should return something like this:

![namespaces](_media/namespaces.png)

> - <u>PID namespace:</u> Isolates the process ID number space. In other words, a process running in a specific PID namespace has no knowledge of processes running in a different PID namespace.
> - <u>Network namespace:</u> Provides isolation of network controllers, system resources associated with networking, firewall rules, etc.
> - <u>Mount namespace:</u> Isolates the set of filesystem mount points seen by a group of processes. Processes in different mount(mnt) namespaces can have different views of the filesystem hierarchy.
> - <u>User namespace:</u> Isolates the user ID number space. In other words, a process's user and group IDs can be different inside and outside a user namespace. The most important aspect of this is that a process can have a normal unprivileged user ID outside a user namespace while at the same time having a user ID of 0 inside the namespace; in other words, the process has full privileges for operations inside the user namespace, but is unprivileged for operations outside.the namespace. Akin to being root inside the container while not necessarily being root in the base system running the process
> - <u>UTS namespace:</u> Allows each container to have its own hostname and domain name. UNIX timesharing system(UTX)
> - <u>IPC namespace:</u> Isolates certain interprocess communication (IPC) resources, such as System V IPC objects and POSIX message queues.
> - <u>Cgroup namespace:</u> Linux kernel feature to limit, and police resource usage by a set of processes.
> - <u>Time namespace:</u> Isolates system clocks. Each namespace has a different system time.
> - <u>Time/PID for Children namespace:</u> Special namespaces for child processes to make sure they are correctly tracked and configured 

## CGroups

Directly from the [man-page](https://man7.org/linux/man-pages/man7/cgroups.7.html):

> A Linux kernel feature which allow processes to be organized into hierarchical groups whose usage of various types of resources then be limited and monitored.

As mentioned in the quote above, processes are organized into hierarchical groups, a single hierarchy(tree). When the kernel boots up it starts a process named *init*, from which all other processes are created, and in turn, these processes can create their own child-processes and they inherit the environment from it's parent process.

Many different hierarchies of cgroups can exist simultaneously on a system. If the Linux process model is a single tree of processes, then the cgroup model is one or more separate, unconnected trees of tasks (i.e. processes).
Multiple separate hierarchies of cgroups are necessary because each hierarchy is attached to one or more subsystems. A subsystem represents a single resource, such as CPU time or memory.

**Note:** If you want to check for yourself what a container engine is using try and execute: `<container engine> info`


## Quiz time

You are now ready to do a quick quiz about containers.
[Click me](containersquiz.md)

## Resources

[Docker underlying technology](https://www.codementor.io/blog/docker-technology-5x1kilcbow)

[Namespaces article](https://lwn.net/Articles/531114/)

[A great presentation about building a simple container from scratch](https://www.youtube.com/watch?v=8fi7uSYlOdc)

[In depth look into CGroups](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/resource_management_guide/ch01#sec-How_Control_Groups_Are_Organized)
