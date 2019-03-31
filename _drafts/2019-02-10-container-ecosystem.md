Containers are great and the more you try know about how they work,
more the terminology gets confusing. Docker, Docker Engine,
containerd, runC, rkt, cri-o -- all describing themselves are
container runtime, are they alternatives or complimentary components?
From what I understood from reading on docs and descriptions, I've
summarized these terms below. If you have any suggestion or feedback,
please [let me know](/contact).

Going bottom-up, Open Container Initiative or OCI is a project by
Linux Foundation which standarises the container runtime, image and
distribution spectifications which are released as [runtime-spec][0],
[image-spec][1] and [distribution-spec][2] respectively.

[runC][3] is one of the many implementation of `runtime-spec`, other
implementations can be found [here][4].  Now these implementations
have very limited and defined scope in container world i.e. it will
create, start and delete the container. Generally, a more
comprehensive runtime will be required in real systems which should be
implemented on top of `runC` and would be responsible for things like
downloading container images, managing storage and network
inferfaces. Widely used example of this runtime is `containerd` which
can manage multiple containers using runC or any other OCI
implementation. Check this [post by Michael Crosby][5] to know more
about how `containerd` integrates `runC`.

Another runtime is [cri-o][6] by Kubernetes which is OCI
implementation with just enough functionality that is required by
Kubernetes CRI i.e. Container Runtime Interface, which is an interface
for using any runtime with Kubernetes' `kubelet`. That means,
`containerd` and `cri-o` are alternative runtimes for using with
`kubelet`.

`kubelet` or Docker Engine are higher level abstraction that uses and
depend on above mentioned runtimes. Docker Enginer uses `containerd`
and adds things like networking, volumes and security.

![Container Ecosystem][7] *Source:
 [https://blog.docker.com/2017/08/what-is-containerd-runtime/][8]*

Docker, the company, also provides an end-user cli client `docker` to
interact with Docker Engine and what we uses directly on our local
systems to manange containers. Don't be confused if Docker, containerd
and runC all are described as runtimes, they are classified as
runtimes but with different scopes.

If you've reading about these components, you might have came across
something called the Moby Project . So, earlier Docker used to be a
huge piece of software by Docker, the company and with time they
separated out the major components like `containerd` which can be used
and developed independently. All these components that Docker now uses
as upstream along with other tools and framework to assemble them,
comes under this [Moby Project][9]. [This introductory post][12] talks
about same in detail.

Another parallel container project is called `rkt`, which implements
whole different approach to manage containers and developed by CoreOS,
it has a runtime which supports OCI images but as of now it does not
follow exact `runtime-spec`. Think of `rkt` as alternative to Docker
Engine along with `containerd` and `runC`. `rkt` also has CRI
implementation for using with Kubernetes, called `rktlet`. Following
image captures this comparision with other details.

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
