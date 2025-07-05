# Chapter 8. Introducing CoreUpdate and Container/Enterprise Registry

In the previous chapter, we saw how to set up a production cluster and deploy our code, how to set up staging using Docker builder, and private Docker registry machines to production servers.

In this chapter, we will overview the CoreOS update strategies, paid CoreUpdate services, and Docker image hosting at the Container Registry and the Enterprise Registry.

In this chapter we will cover the following topics:

*   Update strategies
*   CoreUpdate
*   Container Registry
*   Enterprise Registry

# Update strategies

Before we look at the paid CoreUpdate services from CoreOS, let's overview automatic update strategies that come out-of-the-box.

## Automatic updates

CoreOS comes with automatic updates enabled by default.

As we have mentioned earlier, as updates are released by the CoreOS team, the host will stage them down to a temporary location and install to the passive `usr` partition. After rebooting, active and passive partitions get swapped.

At the time of writing this book, there are four update strategies, as follows:

![Automatic updates](img/image00171.jpeg)

Which update strategy should be used is defined in the `update` part of `cloud-config`:

```
  #cloud-config
  coreos:
    update:
      group: stable
      reboot-strategy: best-effort
```

Let's take a look at what these update strategies are:

*   `best-effort`: This is the default one and works in such a way that it checks whether the machine is part of the cluster. Then it uses `etcd-lock`; otherwise it uses the `reboot` strategy.
*   `etcd-lock`: This allows us to boot only one machine at a time by putting a `reboot` lock on each machine and rebooting them one by one.
*   `reboot`: This reboots the machine as soon as the update gets installed on the passive partition.
*   `off`: The machine will not be rebooted after a successful update install onto the passive partition.

## Uses of update strategies

Here are some examples of what `update` strategies can be used for:

*   `best-effort`: This is recommended to be used in production clusters
*   `reboot`: This can be used for machines that can only be rebooted at a certain time of the day—for example, for automatic updates in a maintenance window
*   `off`: This can be used for a local development environment where the control of reboots is in the user's hands

If you want to learn more about update strategies, take a look at the CoreOS website at [https://coreos.com/docs/cluster-management/setup/update-strategies/](https://coreos.com/docs/cluster-management/setup/update-strategies/).

# CoreUpdate

CoreUpdate is a part of the managed Linux plans ([https://coreos.com/products/](https://coreos.com/products/)).

It is a tool in the commercial offerings of CoreOS. It provides users with their own supported Omaha server and is analogous to tools such as Red Hat Satellite Server and Canonical Landscape:

*   The standard plan is managed and hosted by CoreOS
*   The premium plan can be run behind the firewall, which can be on-premise or on the cloud

CoreUpdate uses exactly the same strategies as the aforementioned `update` strategies, except for a few differences in the `update` portion of the `cloud-config` file:

```
#cloud-config 
  coreos:
    update:
      group: production
      server: https://customer.update.core-os.net/v1/update
```

Here:

*   `group` is what you have set at your CoreUpdate dashboard
*   `server` is the link generated for you after signing in for the managed Linux plan

In our current example, as per `cloud-config`, the servers belong to `https://customer.update.core-os.net/v1/update` and `group` is `production`.

We change via the CoreUpdate UI, as shown in the following screenshot:

![CoreUpdate](img/image00172.jpeg)

The following features are present:

*   Release channel; in our case, it is the stable one
*   Enable/disable automatic updates
*   Time window between machines updates; in our case, it is 90 minutes

The CoreUpdate UI allows you to very easily control your cluster update groups, without any need to perform `ssh` via the terminal to your servers and change there on each server individually update settings.

### Note

You can read more about CoreUpdate at the following pages:

[https://coreos.com/docs/coreupdate/coreos/coreupdate-configure-machines](https://coreos.com/docs/coreupdate/coreos/coreupdate-configure-machines)

[https://coreos.com/docs/coreupdate/coreos/coreupdate-getting-started](https://coreos.com/docs/coreupdate/coreos/coreupdate-getting-started)

# Container Registry

The Container Registry is a hosted CoreOS service for application containers at [https://quay.io](https://quay.io). There, you can host your Docker images if you don't want to run Private Docker Registry yourself:

*   It offers free, unlimited storage and repositories for public container repositories
*   If you want private repositories, it offers a plenty of plans to choose from

## Quay.io overview

Let's go through an overview of what they have there: a nice and easy-to-use UI.

![Quay.io overview](img/image00173.jpeg)

In the following screenshot we see postgres containers image in more details:

![Quay.io overview](img/image00174.jpeg)

As you see from the preceding screenshot, the UI is very easy to use and it's easy to understand the features.

Let's see how the Create Repository feature looks:

![Quay.io overview](img/image00175.jpeg)

When you create a new repository, you can do the following:

*   Make the repository public or private.
*   Empty it if you want to build containers yourself and push them to the Registry.
*   Provide (upload) a `Docker` file.
*   Link to the GitHub repository. This is the preferred choice as it allows you to automate container building when you push changes to your GitHub Repository.

# Enterprise Registry

Enterprise Registry is basically the same as Container Registry, but is hosted on your premises or cloud servers behind your firewall.

It has different plan options and can be found at [https://coreos.com/products/enterprise-registry/](https://coreos.com/products/enterprise-registry/).

It allows you to manage container builds, permissions of your teams and users, and so on.

If your company's requirement is a setup that is very secured and fully controlled by you, then using the Container Registry and Enterprise Registry is the way to go.

# Summary

In this chapter, we overviewed the CoreOS update strategies, CoreUpdate services, the hosted free/paid Container Registry at [https://quay.io](https://quay.io), and the self-hosted Enterprise Registry services.

In the next chapter, you will be introduced to the CoreOS rkt—App Container runtime that can be used instead of Docker containers.