# Chapter 7. Building a Production Cluster

In the previous chapter, we saw how to deploy code on a remote test/staging cluster, and set up the Docker builder and Private Docker Registry server. In this chapter, we will cover how to set up a production cluster on Google Cloud Compute Engine and how to deploy code from the Staging server using the Docker builder and Docker private registry.

We will cover the following topics in this chapter:

*   Bootstrapping a remote Production cluster to GCE
*   Deploying code on the Production cluster servers
*   An overview of the setup of Dev/Test/Staging/Production
*   PaaS based on `fleet`
*   Another cloud alternative to run CoreOS clusters

# Bootstrapping a remote production cluster on GCE

We have already seen how to set up our test/staging environment on Google Cloud. Here, we will use a very similar approach to set up our Production cluster, where the usually tested code is run in a stable environment with more powerful and high-availability servers.

## Setting up the production cluster

Before we install the cluster, let's see what folders/files we have there; type the following commands in your terminal:

```
$ cd coreos-essentials-book/chapter7/Production_Cluster
$ ls
cloud-config 
create_cluster_workers.sh 
fleet 
files 
create_cluster_control.sh 
install_fleetctl_and_scripts.sh
settings

```

As you can see, we have folders/files that are very similar to what we used to set up the Test/Staging Cluster.

### Note

We are not going to print all the scripts and files that we are going to use, as it will take up half the chapter just for that. Take a look at the scripts and other files. They are very well-commented, and it should not be too difficult to understand them.

When you are done with this chapter, you can adopt the provided scripts to bootstrap your clusters. As before, update the `settings` file with your Google Cloud project ID and the zone where you want CoreOS instances to be deployed:

1.  Next let's install our control server, which is our Production cluster's etcd node:

    ```
    $ ./create_cluster_control.sh

    ```

    ![Setting up the production cluster](img/image00159.jpeg)

    We've just created our new Production cluster's control node.

    For learning purposes, we used only one `etcd` server. For a real Production Cluster, a minimum of three `etcd` servers is recommended, and each server should be located in a different cloud availability zone.

    As the Production cluster setup scripts are very similar to the Test/Staging cluster scripts, we are not going to analyze them here.

2.  The next step is to create our Production cluster workers:

    ```
    $ ./create_cluster_workers.sh

    ```

    You should see the following output:

    ![Setting up the production cluster](img/image00160.jpeg)

    For the other cluster workers, you should see something like this:

    ![Setting up the production cluster](img/image00161.jpeg)

    ### Note

    Make a note of the workers' external IPs; we will need them later. Of course, you can always check them out at the Google Cloud Developers Console.

    So, we've got our production servers set up on GCE. If you check out the Google Cloud Developers Console for Compute Engine Instances, you should see a list of servers, like this:

    ![Setting up the production cluster](img/image00162.jpeg)
3.  Now let's install all the necessary scripts to access our cluster:

    ```
    $ ./install_fleetctl_and_scripts.sh

    ```

    This script will create a new folder called `~/coreos-prod-gce`, which will have the same folders as our Test/Staging cluster:

    *   The `bin` folder will have scripts for accessing cluster machines and the `set_cluster_access.sh` script
    *   The `fleet - website1.service fleet` unit file

4.  Let's run `set_cluster_access.sh`:

    ```
    $ cd ~/coreos-prod-gce/bin
    $ ./set_cluster_access.sh

    ```

    ![Setting up the production cluster](img/image00163.jpeg)

Perfect! Our production cluster is up-and-running!

As you can see, we have three servers there, one for the `etcd` services and two workers to run our website.

We already have the `website1 fleet` unit prepared. Let's install it:

```
$ cd ~/coreos-prod-gce/fleet
$ fleetctl start website1.service

```

The following screenshot demonstrates the output:

![Setting up the production cluster](img/image00164.jpeg)

Now we are ready to deploy code on our Production servers.

# Deploying code on production cluster servers

In the previous chapters, we saw how to set up our Test/Staging environment on Google Cloud and deploy our code there, and we did set up our Docker builder and Docker Private Registry server.

In the next section, we will learn how to deploy code on our Web servers in Production cluster using Docker builder and Docker Private Registry.

## Setting up the Docker builder server

Before we deploy our code from staging to production, we need to copy the `Dockerfile` and the `build.sh` and `push.sh` files to our Docker builder.

To do this, run the following commands:

```
$ cd coreos-essentials-book/chapter7/Test_Staging_Cluster/
$ ./install_website1_2_dbuilder.sh

```

You should see something like what is shown in the following screenshot:

![Setting up the Docker builder server](img/image00165.jpeg)

So let's check out what happened—that is, what that script has done. It has copied three files to Docker builder server:

1.  This will be used to build our production Docker image:

    ```
    $ cat Dockerfile:
    FROM nginx:latest
    ## add website code
    ADD website1 /usr/share/nginx/html
    EXPOSE 80

    ```

2.  The following is the Docker image building script:

    ```
    $ cat build.sh
    docker build --rm -t 10.200.4.1:5000/website1 .

    ```

3.  Here is the Docker image push script to our Private Docker Registry:

    ```
    $ cat push.sh
    docker push 10.200.4.1:5000/website1

    ```

Okay, we have prepared our Docker builder server. Let's start cracking the code deployment on the production servers.

## Deploying code on production servers

To deploy code on our production web servers, run the following command:

```
$ cd ~/coreos-prod-gce

```

When we built the production cluster, the install script installed the `deploy_2_production_website1.sh` script. Let's run it, and you should see an output similar to the next two screenshots:

```
$ ./deploy_2_production_website1.sh

```

![Deploying code on production servers](img/image00166.jpeg)

You should also see something like this:

![Deploying code on production servers](img/image00167.jpeg)

Now open `prod-web1` and `prod-web2` in your browser using their external IPs, and you should see something like what is shown in the following screenshot:

![Deploying code on production servers](img/image00168.jpeg)

We see exactly the same web page as on our staging server.

Awesome! Our deployment to production servers is working fine!

Let's see what happened there.

Run the following command:

```
$ cat deploy_2_production_website1.sh
#!/bin/bash
# Build docker container for website1
# and release it

function pause(){
read -p "$*"
}

# Test/Staging cluster
## Fetch GC settings
# project and zone
project=$(cat ~/coreos-tsc-gce/settings | grep project= | head -1 | cut -f2 -d"=")
zone=$(cat ~/coreos-tsc-gce/settings | grep zone= | head -1 | cut -f2 -d"=")
cbuilder1=$(gcloud compute instances list --project=$project | grep -v grep | grep tsc-registry-cbuilder1 | awk {'print $5'})

# create a folder on docker builder
echo "Entering dbuilder docker container"
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no  core@$cbuilder1 "/usr/bin/docker exec docker-builder /bin/bash -c 'sudo mkdir -p /data/website1 && sudo chmod -R 777 /data/website1'"

# sync files from staging to docker builder
echo "Deploying code to docker builder server !!!"
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no core@$cbuilder1 '/usr/bin/docker exec docker-builder rsync -e "ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no" -avzW --delete core@10.200.3.1:/home/core/share/nginx/html/ /data/website1'
# change folder permisions to 755
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no  core@$cbuilder1 "/usr/bin/docker exec docker-builder /bin/bash -c 'sudo chmod -R 755 /data/website1'"

echo "Build new docker image and push to registry!!!"
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no core@$cbuilder1 "/usr/bin/docker exec docker-builder /bin/bash -c 'cd /data && ./build.sh && ./push.sh'"
##

# Production cluster
## Fetch GC settings
# project and zone
project2=$(cat ~/coreos-prod-gce/settings | grep project= | head -1 | cut -f2 -d"=")

# Get servers IPs
control1=$(gcloud compute instances list --project=$project2 | grep -v grep | grep prod-control1 | awk {'print $5'})
web1=$(gcloud compute instances list --project=$project2 | grep -v grep | grep prod-web1 | awk {'print $5'})
web2=$(gcloud compute instances list --project=$project2 | grep -v grep | grep prod-web2 | awk {'print $5'})

echo "Pull new docker image on web1"
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no core@$web1 docker pull 10.200.4.1:5000/website1
echo "Pull new docker image on web2"
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no core@$web2 docker pull 10.200.4.1:5000/website1

echo "Restart fleet unit"
# restart fleet unit
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no core@$control1 fleetctl stop website1.service
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no core@$control1 fleetctl start website1.service
#
sleep 5
echo " "
echo "List Production cluster fleet units"
ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no core@$control1 fleetctl list-units

echo " "
echo "Finished !!!"
pause 'Press [Enter] key to continue...'

```

The steps for deployment to production are as follows:

1.  Creates a folder called `/data/website1` on the Docker builder server.
2.  Use `rsync` via the docker-builder container to sync files from `Staging1` to the Docker builder server.
3.  Run the `build.sh` script via the docker-builder container.
4.  Push a new Docker image to the Private Docker Registry.
5.  Pull a new Docker image onto the `Prod-web1` and `prod-web2` servers.
6.  Restart the `website1.service fleet` unit via the Production cluster's `etcd` server.
7.  And voilà! We have completed the release of a new website to our production cluster.

### Note

**One thing to note**

We are using the docker-builder container to sync and build our Docker container.

This can be done directly on the Docker builder server, but using the container allows us to add any tools required to the container, which gives an advantage. If we need to replicate the Docker Builder server or replace it with a new one, we just have to install our docker-builder container to get things working again.

# An overview of the Dev/Test/Staging/Production setup

Let's overview the advantages of performing the setup of the Dev/Test/Staging/Production environment in the way we did it:

*   Local code development via the CoreOS VM decreases your testing time, as all changes get pushed to a local server on your VirtualBox VM.
*   Cloud-based Test/Staging is good to use for team-shared projects using GitHub or Bitbucket. It also has, in our case, `nginx` containers running as our web servers, and the code is used via the attached `host` folder. This significantly speeds up code deployment from the test and staging `git` branches, as the Docker container does not need to be rebuilt each time we pull code from the `git` repository.
*   For production, a separate cluster is used. It is good practice to separate development and production clusters.
*   For production, we use the same Docker base image as that on the test/staging servers, but we build a new Docker image, with the code baked inside. So, we can, for example, auto-scale our website to as many servers as we want by reusing the same Docker image on all the servers, and all the servers will be running exactly the same code.
*   For Docker image building and our Private Docker Registry, we use the same server, which is accessible only via the internal GCE IP. If you want to expose the Docker Registry to external access, for example, the `nginx` container with authentication should be put in front of the Docker registry to make it secure.
*   This is only one way of setting up the Dev/Test/Staging/Production environment. Each setup scenario is different, but such setup should put you on the right path.

# PaaS based on fleet

In this chapter and in previous chapters, we explained how to use fleet to deploy our different services on our clusters. Fleet is a powerful and easy-to-use low-level cluster manager that controls `systemd` at the cluster level. However, it lacks a web UI, easy orchestration tools, and so on, so this is where PAZ, the nice PaaS, steps in to help us out.

## Deploying services using PAZ

The website at [http://www.paz.sh](http://www.paz.sh) has a very nice and user-friendly web UI, which makes it much easier to set up a CoreOS cluster. PAZ also has an API that you can use if you want to automate things via scripts.

Through its dashboard, you can add and edit your services, check the status of the cluster (viewed by host or by unit), and view monitoring information and logs for the cluster.

![Deploying services using PAZ](img/image00169.jpeg)

It fully leverages `fleet` to orchestrate services across the machines in a cluster. It is built in `Node.js` and all its services run as Docker containers.

The following pointers explain how PAZ works:

*   Users can declare services in the UI
*   Services get stored in the service directory
*   The scheduler is the service that deploys things
*   You can manually tell the scheduler to deploy, or have it triggered at the end of your CI process
*   Paz supports the post-push Docker Hub web hooks
*   By using `etcd` and service discovery, your containers are linked together

Of course, it will keep evolving and getting new features but, at the time of writing this book, only the services in the preceding list were available.

Giving a complete overview of PAZ is beyond the scope of this book, but you can read more about the Paz architecture at [http://paz.readme.io/v1.0/docs/paz-architecture](http://paz.readme.io/v1.0/docs/paz-architecture).

# Another cloud alternative for running CoreOS clusters

To bootstrap our Test/Staging and Production clusters, we used the Google Cloud Compute Engine's virtual instances, but sometimes you might want to run your servers on real servers (bare-metal servers) that are not stored at your premises.

There are a number of different bare-metal server providers out there, but one that caught my eye was [https://www.packet.net](https://www.packet.net).

I recently came across these while I was investigating hosting solutions for CoreOS and containers. They're interesting in the sense that, instead of going the typical cloud/hypervisor route, they've created a true, on-demand, and bare-metal cloud solution. I'm able to spin up a CoreOS server from scratch in less than 5 minutes, and they have a pretty comprehensive API and accompanying documentation.

Here's an example of a packet project dashboard:

![Another cloud alternative for running CoreOS clusters](img/image00170.jpeg)

# Summary

In this chapter, we saw how to set up a Production cluster and deploy our code staging using the Docker builder and private Docker registry machines. Finally, we overviewed a PaaS based on `fleet`—`Paz.sh`.

In the next chapter, we will overview the CoreOS update strategies and CoreUpdate for our servers. We will also make use of hosted public/private Docker repositories at [https://quay.io](https://quay.io) and the self-hosted CoreOS Enterprise Registry.