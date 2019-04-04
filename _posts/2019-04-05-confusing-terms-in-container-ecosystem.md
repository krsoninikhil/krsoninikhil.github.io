Containers are great and software components which enables them are
some of my favourite projects. But the more you try to know about how
they work, more the terminology gets confusing. Docker, Docker Engine,
containerd, runC, rkt, cri-o -- all describing themselves are
container runtime, are they alternatives or complimentary components?
Based on my understanding from trying to contribute to one of these
projects, I've summarized these terms below assuming you've used
containers before. If you have any suggestions, please [let me
know](/contact).

Going bottom-up, Open Container Initiative or OCI is a project by
Linux Foundation which standarises the container runtime, image and
distribution spectifications, these specifications are released as
[runtime-spec][0], [image-spec][1] and [distribution-spec][2]
respectively.

[runC][3] is one of the many implementation of `runtime-spec`, other
implementations can be found [here][4]. Now given the specs, these
implementations have very limited and defined scope in container world
i.e. `runc` can create, start and delete a container. Generally, a
more comprehensive runtime will be required in real systems and can be
implemented on top of `runC`. This runtime would be responsible for
managing multiple containers at a time and things like downloading
container images, managing storage and network inferfaces, etc. Widely
used example of this runtime is `containerd` which can manage multiple
containers using runC or any other OCI implementation. Check this
[post by Michael Crosby][5] to know more about how `containerd`
integrates `runC`.

Another runtime is [cri-o][6] by [Kubernetes][13] which is OCI
implementation with just enough functionality that is required by
Kubernetes CRI. CRI or Container Runtime Interface is a Kubernetes
interface for using any runtime with its container manager called
`kubelet`. That means, `containerd` and `cri-o` are alternative
runtimes for using with `kubelet`.

Now, `kubelet` and Docker Engine are higher level abstraction that
uses and depend on above mentioned runtimes. Docker Engine uses
`containerd` and adds things like networking, volumes and security to
the containers.

Following image shows how these pieces fit together visually.

![Container Ecosystem][7] *Source:
 [https://blog.docker.com/2017/08/what-is-containerd-runtime/][8]*

Docker, the company, also provides an end-user cli client `docker` to
interact with Docker Engine and what we use directly on our local
systems to manange containers. Don't be confused if Docker, containerd
and runC all are described as runtimes, they are classified as
runtimes but with different and very defined scopes.

Another interesting project is the Moby Project, which you might have
came across if you've been reading about these components. So, earlier
Docker used to be a huge piece of software by Docker, the
company. With time, they separated out the major components like
`containerd` which can be used and developed independently. All these
components that Docker now uses as upstream along with some tools and
framework to assemble them, comes under this [Moby Project][9]. [This
introductory post][12] talks about same in detail.

Another parallel container project is called `rkt`, which implements
whole different approach to manage containers and developed by CoreOS.
It has a runtime which supports OCI images but as of now it does not
follow exact `runtime-spec`. Think of `rkt` as alternative to Docker
Engine along with `containerd` and `runC`. `rkt` also has CRI
implementation for using with Kubernetes, called `rktlet`. Following
image captures this comparision along with other details.

![rkt vs Docker][10]

*Source:
[https://coreos.com/rkt/docs/latest/rkt-vs-other-projects.html#rkt-vs-docker][11]*

To summarize, `runC` is OCI spec implementation for creating, starting
and deleting containers. `containerd` uses `runC` and provides a
broader runtime for containers. Docker client uses Docker Engine on
local systems which is built on top of `containerd`. Docker uses Moby
Project as upstream which includes all these components. Kubernetes
can use `containerd`, `cri-o` or `rktlet` as it's container
runtime. `rkt` is an alternative to Docker and has all components of
it's own.

PS: Thanks [Fakabbir Amin][14] for reading draft of this post.

[0]: https://github.com/opencontainers/runtime-spec
[1]: https://github.com/opencontainers/image-spec
[2]: https://github.com/opencontainers/distribution-spec
[3]: https://github.com/opencontainers/runc
[4]: https://github.com/opencontainers/runtime-spec/blob/master/implementations.md
[5]: https://blog.docker.com/2016/04/docker-containerd-integration/
[6]: https://github.com/kubernetes-sigs/cri-o
[7]: https://i2.wp.com/blog.docker.com/wp-content/uploads/974cd631-b57e-470e-a944-78530aaa1a23-1.jpg?w=906&ssl=1
[8]: https://blog.docker.com/2017/08/what-is-containerd-runtime/
[9]: https://mobyproject.org/projects/
[10]: https://coreos.com/rkt/docs/latest/rkt-vs-docker-process-model.png
[11]: https://coreos.com/rkt/docs/latest/rkt-vs-other-projects.html#rkt-vs-docker
[12]: https://blog.docker.com/2017/04/introducing-the-moby-project/
[13]: https://kubernetes.io/
[14]: https://fakabbir.github.io/
