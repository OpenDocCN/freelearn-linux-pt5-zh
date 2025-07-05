# Chapter 9. Introduction to CoreOS rkt

In the previous chapter, we overviewed CoreUpdate, free and paid container repositories, and the hosting and enterprise registry provided by CoreOS.

In this chapter, you will learn about CoreOS's `rkt`, a container runtime for applications. We will cover the following topics in this chapter:

*   Introduction to `rkt`
*   Running streamlined Docker images with `rkt`
*   Converting Docker images to ACI

# An introduction to rkt

rkt (pronounced "rock it") is a container runtime for applications made by CoreOS and is designed for composability, speed, and security. It is an alternative to Docker and is designed to be run on servers with the most rigorous security and production environments.

rkt is a standalone tool, compared to Docker's client and central daemon version, which makes it a better alternative to Docker, as it has fewer constraints and dependencies. For example, if the `docker` central daemon crashes, all running `docker` containers will be stopped; in the case of `rkt`, however, this can affect only the particular rkt process responsible for running `rkt` containers in its pod. As each `rkt` process gets its **process identification number** (**PID**), if one `rkt` process dies, it will not affect any other `rkt` process.

## Features of rkt

We will overview the main `rkt` features, as follows:

*   It can be integrated with `init` systems, as `systemd` and `upstart`
*   It can be integrated with cluster orchestration tools, such as `fleet` and `Kubernetes` (which we will cover in the next chapter)
*   It is compatible with other container solutions as Docker
*   It has an extensible and modular architecture

## The basics of App container

`rkt` is an implementation of **App Container** (**appc**: [https://github.com/appc/spec/](https://github.com/appc/spec/)), which is open source and defines an image format, the runtime environment, and the discovery mechanism of application containers:

*   `rkt` uses images of the **Application Container Image** (**ACI**) format as defined by the App Container (appc) specifications ([https://github.com/appc/spec](https://github.com/appc/spec)). An ACI is just a simple `tarball` bundle of different `rootfs` files and an image manifest.
*   A pod (the basic unit of execution in `rkt`) is a grouping of one or more app images (ACIs), with some optionally applied additional metadata on the pod level—for example, applying some resource constraints, such as CPU usage.

## Using rkt

As rkt comes preinstalled with CoreOS, running ACI images with rkt is easy and it is very similar to `docker` commands. (I would love to write more on this, but rkt does not provide many options yet, as it is constantly changing and innovating, which was also the case at the time of writing this book).

As `rkt` has no running OS X client, you need to log in to your CoreOS VM host directly to run the following example commands:

1.  First, we need to trust the remote site before we download any ACI file from there, as `rkt` verifies signatures by default:

    ```
    $ sudo rkt trust –prefix example.com/nginx

    ```

2.  Then we can fetch (download) an image from there:

    ```
    $ sudo rkt fetch example.com/nginx:latest

    ```

3.  Then running the container with `rkt` is simple:

    ```
    $ sudo rkt run example.com/nginx:v1.8.0

    ```

As you see, `rkt` appropriates ETags—as in our case v1.8.0 will be run.

## rkt networking

By default `rkt` run uses the host mode for port assignments. For example, if you have `EXPOSE 80` in your Dockerfile, run this command:

```
$ sudo rkt run example.com/nginx:v1.8.0

```

The `rkt` pod will share the network stack and interfaces with the host machine.

If you want to assign a different port/private IP address, then use `run` with these parameters:

```
sudo rkt run --private-net --port=http:8000 example.com/nginx:v1.8.0

```

## rkt environment variables

Environment variables can be inherited from the host using the `--inherit-env` flag. Using `flag --set-env`, we can set individual environment variables.

Okay, let's prepare a few environment variables to be inherited using these two commands:

```
$ export ENV_ONE=hi_from_host
$ export ENV_TWO=CoreOS

```

Now let's use them together with `--set-env` in the command, as follows:

```
$ sudo rkt run --inherit-env --set-env ENV_THREE=hi_nginx example.com/nginx:v1.8.0

```

## rkt volumes

For host volumes, the `-volume` flag needs to be used. Volumes need to be defined in the ACI manifest when creating the new ACI image and converting Docker images. You will get an output like this:

![rkt volumes](img/image00176.jpeg)

The following command will mount the `host` directory on the `rkt` Pod:

```
$ sudo rkt run –volume volume-/var/cache/nginx,kind=host,source=/some_folder/nginx_cache example.com/nginx:v1.8.0

```

Note that the `rkt` volume standard was not completed at the time of writing this book, so the previous example might not work when `rkt` reaches its final version.

Next let's see how `rkt` plays nicely with docker images.

# Running streamlined Docker images with rkt

As there are thousands of docker images on the public Docker hub, `rkt` allows you to use them very easily. Alternatively, you might have some docker images and would like to run them with `rkt` too, without building new `rkt` ACI images, to see how they work with `rkt`.

Running Docker images is very much the same as it was in previous examples:

1.  As Docker images do not support signature verification yet, we just skip the verification step and fetch one with the `--insecure-skip-verify` flag:

    ```
    $ sudo rkt --insecure-skip-verify fetch docker://nginx

    ```

    ![Running streamlined Docker images with rkt](img/image00177.jpeg)
2.  The last line shown in the preceding screenshot represents the `rkt` image ID of the converted ACI, and this can be used to `run` with `rkt` :

    ```
    $ sudo rkt --insecure-skip-verify run sha512-13a9c5295d8c13b9ad94e37b25b2feb2

    ```

3.  Also we can run in this way, where the image will be downloaded and then run:

    ```
    $ sudo rkt --insecure-skip-verify run docker://nginx

    ```

4.  If we want to use volumes with Docker images, we run this line:

    ```
    $ sudo rkt --insecure-skip-verify run \
    --volume /home/core/share/nginx/html:/usr/share/nginx/html \
    docker://nginx

    ```

    This is very similar to the `docker` command, isn't it?

5.  Okay, let's update our local development `nginx.service` to use `rkt`:

    ```
    [Unit]
    Description=nginx
    [Service]
    User=root
    TimeoutStartSec=0
    EnvironmentFile=/etc/environment
    ExecStart=/usr/bin/ rkt --insecure-skip-verify run \
     -volume /home/core/share/nginx/html:/usr/share/nginx/html \
    docker://nginx
    #
    Restart=always
    RestartSec=10s
    [X-Fleet]

    ```

As you see, there is no `ExecStop=/usr/bin/docker stop nginx`. It is not needed because `systemd` takes care of stopping the `rkt` instance when the `systemctl`/`fleetctl` stop is used by sending the running `nginx` process a `SIGTERM`.

Much simpler than docker, right?

In the next section, we will see how to convert a docker image into an ACI image.

# Converting Docker images into ACI

With CoreOS comes another file related to `rkt`—`docker2aci`. It converts a docker image to an ACI image (an application container image used by `rkt`).

Let's convert our `nginx` image. Run the following command:

```
$ docker2aci docker://nginx

```

![Converting Docker images into ACI](img/image00178.jpeg)

We can also save a docker image in a file and the convert it. Run the following command:

```
$ docker save -o nginx.docker nginx
$ docker2aci nginx.docker

```

![Converting Docker images into ACI](img/image00179.jpeg)

Finally, you can try to use the generated ACI files by updating the preceding `nginx.service fleet` unit:

```
[Unit]
Description=nginx
[Service]
User=root
TimeoutStartSec=0
EnvironmentFile=/etc/environment
ExecStart=/usr/bin/ rkt --insecure-skip-verify run \
 --volume volume-/usr/share/nginx/html,kind=host,source=/usr/share/nginx/html \
 full_path_to/nginx-latest.aci
#
Restart=always
RestartSec=10s

[X-Fleet]

```

# Summary

In this chapter, we overviewed the main features of CoreOS rkt, the `rkt` application container, and the image format. You also learned how to run images based on `aci` and `docker` as containers with `rkt`.

In the next chapter, you will get an introduction to Google Kubernetes, an open source orchestration system for application containers.