# `shibboleth-mda-docker`

A container image for use in experimenting
with the [Shibboleth][] Metadata Aggregator product
([MDA][]).

[Shibboleth]: https://shibboleth.net
[MDA]: https://shibboleth.atlassian.net/wiki/spaces/MA1/overview

The container image is available for `linux/amd64` and
`linux/arm64` platforms and incorporates:

- Amazon Linux 2
- Amazon Corretto 17
- The current SNAPSHOT version of Shibboleth MDA 0.10.0, at
  `/opt/mda`.
- `git`
- `openssl`
- `wget`

If you'd find additional tools useful for evaluations, please
raise an issue on the GitHub repository.

When you run the container, an initialisation script
sets up a user environment (in `/home/user`,
as `user` 1000:1000) and opens a command prompt
from which the MDA's command-line can be invoked as
`mda.sh` in the usual way.

## Quick start

You can run a container based on this image as follows:

```bash
docker run --rm -it ianayoung/shibboleth-mda-docker
```

Or, using `podman` instead of `docker`:

```bash
podman run --rm -it docker.io/ianayoung/shibboleth-mda-docker
```

In either case, you should receive output similar to the following:

```text
openjdk version "17.0.8.1" 2023-08-22 LTS
OpenJDK Runtime Environment Corretto-17.0.8.8.1 (build 17.0.8.1+8-LTS)
OpenJDK 64-Bit Server VM Corretto-17.0.8.8.1 (build 17.0.8.1+8-LTS, mixed mode, sharing)
Agent pid 39
user@C17: ~ $
```

At the prompt, invoke `mda.sh` to confirm that everything is working:

```bash
user@C17: ~ $ mda.sh --version
0.10.0-SNAPSHOT
```

You can now experiment with the latest MDA snapshot in a non-persistent
environment: everything you do will be removed when you exit the
container.

## Persistent state

If you would like to maintain state between runs, or work with
files created in the host environment, you should create a work
directory on the host and mount it as `/home/user` in the container.

The details of this depend on your host environment.

### Persistence with Docker Desktop for Mac

Docker Desktop for Mac runs your containers inside a custom
virtual machine, and handles all the details of mapping host
files automatically. This means that you can simply create a
host directory as your normal user and mount that:

```bash
$ mkdir user
$ touch user/from-the-host
$ docker run --rm -it -v ./user:/home/user \
    ianayoung/shibboleth-mda-docker
user@C17: ~ $ ls -l
total 0
-rw-r--r-- 1 user user 0 Sep 25 14:32 from-the-host
```

Similarly, files created within the container will be accessable
from the host once the container has been stopped:

```bash
user@C17: ~ $ touch from-the-container
user@C17: ~ $ logout
$ ls -l user
total 0
-rw-r--r--  1 iay  staff  0 Sep 25 15:34 from-the-container
-rw-r--r--  1 iay  staff  0 Sep 25 15:32 from-the-host
```

### Persistence with Docker and Linux

If running `docker` on Linux, you're probably aware of how
interesting binding host directories can get. One simple route
is to run the container as `root` with the `user` directory
and files set to be owned by the 1000:1000 user within the container:

```bash
root@pi03:~# mkdir user
root@pi03:~# touch user/from-the-host
root@pi03:~# chown -R 1000:1000 user
root@pi03:~# docker run --rm -it -v ./user:/home/user \
    ianayoung/shibboleth-mda-docker
openjdk version "17.0.8.1" 2023-08-22 LTS
OpenJDK Runtime Environment Corretto-17.0.8.8.1 (build 17.0.8.1+8-LTS)
OpenJDK 64-Bit Server VM Corretto-17.0.8.8.1 (build 17.0.8.1+8-LTS, mixed mode, sharing)
Agent pid 49
user@C17: ~ $ ls -l
total 0
-rw-r--r-- 1 user user 0 Sep 25 14:47 from-the-host
user@C17: ~ $ touch from-the-container
user@C17: ~ $ logout
root@pi03:~# ls -ln user
total 0
-rw-rw-r-- 1 1000 1000 0 Sep 25 14:48 from-the-container
-rw-r--r-- 1 1000 1000 0 Sep 25 14:47 from-the-host
```

There are other options, but this isn't the place to learn
how to deploy rootless Docker.

## Persistence with Podman and Enterprise Linux

Using `podman` on Red Hat Enterprise Linux 8 or 9 (or a
rebuild distribution such as Rocky Linux) works very similarly
to using Docker on other Linux systems with one additional
concern: you probably want to at least start out
by disabling SELinux.

```bash
[root@r9d ~]# setenforce 0
[root@r9d ~]# mkdir user
[root@r9d ~]# touch user/from-the-host
[root@r9d ~]# chown -R 1000:1000 user
[root@r9d ~]# podman run --rm -it -v ./user:/home/user \
    docker.io/ianayoung/shibboleth-mda-docker
openjdk version "17.0.8.1" 2023-08-22 LTS
OpenJDK Runtime Environment Corretto-17.0.8.8.1 (build 17.0.8.1+8-LTS)
OpenJDK 64-Bit Server VM Corretto-17.0.8.8.1 (build 17.0.8.1+8-LTS, mixed mode, sharing)
Agent pid 39
user@C17: ~ $ ls -l
total 0
-rw-r--r--. 1 user user 0 Sep 25 15:13 from-the-host
user@C17: ~ $ touch from-the-container
user@C17: ~ $ logout
[root@r9d ~]# ls -l user
total 0
-rw-rw-r--. 1 ansible ansible 0 Sep 25 15:15 from-the-container
-rw-r--r--. 1 ansible ansible 0 Sep 25 15:13 from-the-host
[root@r9d ~]# ls -ln user
total 0
-rw-rw-r--. 1 1000 1000 0 Sep 25 15:15 from-the-container
-rw-r--r--. 1 1000 1000 0 Sep 25 15:13 from-the-host
```

Running `podman` in a rootless mode and with
SELinux active is probably possible but outside the
scope of this README.
