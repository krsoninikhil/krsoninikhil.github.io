Containers are great. But the more you know about them, more the
terminology gets confusing. Docker, Docker Engine, containerd, runC,
rkt, cri-o are they alternatives or components? From what I understood
from reading on docs and descriptions, I've summarized these terms
below. If you have any suggestion or feedback, please [let me
know](/contact).

Going bottom-up, Open Container Initiative or OCI is a project by
Linux Foundation which standarises the container runtime, image and
distribution spectifications which are released as
[runtime-spec](https://github.com/opencontainers/runtime-spec),
[image-spec](https://github.com/opencontainers/image-spec),
[distribution-spec](https://github.com/opencontainers/distribution-spec)
respectively.

[runC](https://github.com/opencontainers/runc) is one of the many
implementation of `runtime-spec`, other implementations can be found
[here](https://github.com/opencontainers/runtime-spec/blob/master/implementations.md).
Now these implementations have very limited and defined scope in
container world i.e. it will create, start and delete the
container. Generally, a more comprehensive runtime will be required in
real systems which should be implemented on top of `runC` and would be
responsible for things like downloading container images, managing
storage and network inferfaces. Widely used example of this runtime is
`containerd` which can manage multiple containers using runC or any
other OCI implementation. Check this [post by Michael
Crosby](https://blog.docker.com/2016/04/docker-containerd-integration/)
to more how `containerd` integrates `runC`.

Another runtime is [cri-o](https://github.com/kubernetes-sigs/cri-o)
by Kubernetes which is OCI implementation with just enough
functionality that is required by Kubernetes CRI i.e. Container
Runtime Interface, which is an interface for using any runtime with
Kubernetes' `kubelet`. That means, `containerd` and `cri-o` are
alternative runtimes for using with `kubelet`.

`kubelet` or Docker Engine are higher level abstraction that uses that
depends on above mentioned runtimes. Docker Enginer used `containerd`
and adds things like networking, volumes and security.

![Container
 Ecosystem](https://i2.wp.com/blog.docker.com/wp-content/uploads/974cd631-b57e-470e-a944-78530aaa1a23-1.jpg?w=906&ssl=1)
*Source: [https://blog.docker.com/2017/08/what-is-containerd-runtime/](https://blog.docker.com/2017/08/what-is-containerd-runtime/)*

Docker also provides a cli client `docker` to interact with Docker
Engine and what we uses directly on our systems to manange
containers. Don't be confused if Docker, containerd and runC all are
described as runtimes, they are classified as runtimes but with
different scopes.

One more thing -- `rkt`, which implements whole different approach to
manage containers and developed by CoreOS, it has a runtime which
supports OCI images but as of now it does not follow exact
`runtime-spec`. Think of `rkt` as alternative to Docker Engine along
with `containerd` and `runC`. `rkt` also has CRI implementation,
called `rktlet`. Following image captures this comparision with other
details.

![rkt vs
 Docker](https://coreos.com/rkt/docs/latest/rkt-vs-docker-process-model.png)

*Source:
[https://coreos.com/rkt/docs/latest/rkt-vs-other-projects.html#rkt-vs-docker](https://coreos.com/rkt/docs/latest/rkt-vs-other-projects.html#rkt-vs-docker)*
