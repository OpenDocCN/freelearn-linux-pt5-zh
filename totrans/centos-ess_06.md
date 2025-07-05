# Chapter 6. Building a Deployment Setup

In the previous chapter, you learned how to set up a local CoreOS environment for development on a personal computer and a Test and Staging environment cluster on Google Cloud's Compute Engine VM instances.

In this chapter, we will cover how to deploy code from the GitHub repository to our Test and Staging servers, and how to set up the Docker builder and Docker private registry worker for Docker image building and distribution.

In this chapter, we will cover the following topics:

*   Code deployment on Test and Staging servers
*   Setting up the Docker builder and private Docker registry machine

# Code deployment on Test and Staging servers

In the previous chapter, you learned how to set up your Test and Staging environment on Google Cloud and deploy your web servers there. In this section, we will see how to deploy code to our web servers on Test and Staging environments.

## Deploying code on servers

To deploy code on our `Test1` and `Staging1` servers, we run the following commands:

```
$ cd coreos-essentials-book/chapter6/Test_Staging_Cluster/webserver
$ ./deploy_2_test1.sh

```

You will get this output:

![Deploying code on servers](img/image00146.jpeg)

Then, run this command:

```
$ ./deploy_2_staging1.sh

```

You should see the following result:

![Deploying code on servers](img/image00147.jpeg)

Now open the `tsc-test1` and `tsc-staging1` VM instance external IPs, copying them to your browser (you can check out the IPs at GC Console, Compute Engine, VM Instance).

The output you see depends on the server.

For the Test server, you should see something like this:

![Deploying code on servers](img/image00148.jpeg)

This is what you will see for the Staging server:

![Deploying code on servers](img/image00149.jpeg)

Let's see what has happened here:

```
$ cat deploy_2_test1.sh
#!/bin/bash

function pause(){
read -p "$*"
}

## Fetch GC settings
# project and zone
project=$(cat ~/coreos-tsc-gce/settings | grep project= | head -1 | cut -f2 -d"=")
zone=$(cat ~/coreos-tsc-gce/settings | grep zone= | head -1 | cut -f2 -d"=")

# change folder permissions
gcloud compute --project=$project ssh  --zone=$zone "core@tsc-test1" --command "sudo chmod -R 755 /home/core/share/"

echo "Deploying code to tsc-test1 server !!!"
gcloud compute copy-files test1/index.html tsc-test1:/home/core/share/nginx/html --zone $zone --project $project

echo " "
echo "Finished !!!"
pause 'Press [Enter] key to continue...'
```

As you can see, we used `gcloud compute` to change the permissions for our `home/core/share/nginx/html` folder, as we need to be able to copy files there. We copied a single `index.html` file there.

In real-life scenarios, `git pull` should be used there to pull from the Test and Staging branches.

To automate releases to the `Test1/Staging1` servers, for example, Strider-CD can be used, but this is beyond the scope of this book. You can read about Strider-CD at [https://github.com/Strider-CD/strider](https://github.com/Strider-CD/strider) and practice implementing it.

# Setting up the Docker builder and private Docker registry worker

We have successfully deployed code (`index.html` in our case) in our Test/Staging environment on the cloud with the help of `gcloud compute`, by running it in a simple shell script.

Let's set up a new server in our Test/Staging environment on the cloud. It will build Docker images for us and store them in our private Docker Registry so that they can be used on our production cluster (you will learn how to set this up in the next chapter).

## Server setup

As both Docker builder and Private Docker Registry fleet units will run on the same server, we are going to deploy one more server on the Test/Staging environment.

To install a new server, run the following commands:

```
$ cd coreos-essentials-book/chapter6/Test_Staging_Cluster
$ ls
cloud-config
create_registry-cbuilder1.sh 
dockerfiles
files
fleet
webserver

```

Next, let's install our new server:

```
$ ./create_ registry-cbuilder1.sh

```

You should see output similar to this:

![Server setup](img/image00150.jpeg)

Let's see what happened during the process of script installation:

*   A new `server tsc-registry-cbuilder1` was created
*   The static IP's `10.200.4.1` forward route for the `tsc-registry-cbuilder1` instance was created
*   The external port `5000` was opened for the new server
*   File `reg-dbuilder1.sh` from the `files` folder got copied to `~/coreos-tsc-gce/bin`
*   The `dbuilder.service` and `registry.service` fleet units from the `fleet` folder got copied to `~/coreos-tsc-gce/fleet`

If we check out the GCE VM Instances at the GC console, we should see our new instance there:

![Server setup](img/image00151.jpeg)

We now need to verify that our new server is working fine, so we perform `ssh` on it:

```
$ cd ~/coreos-tsc-gce/bin
$ ./reg-dbuider1.sh

```

![Server setup](img/image00152.jpeg)

Very good! Our new server is up-and-running. Press *Ctrl* + *D* to exit.

Now we need to verify that our server is connected to our cluster. So, run the following command:

```
$ ./set_cluster_access.sh

```

The script's output should look like this:

![Server setup](img/image00153.jpeg)

Perfect! We can see that our new server has successfully connected to our cluster:

![Server setup](img/image00154.jpeg)

Okay, now let's install those two new fleet units:

```
$ cd ~/coreos-tsc-gce/fleet
$ fleetctl start dbuilder.service registry.service

```

![Server setup](img/image00155.jpeg)

Next, let's list the fleet units:

```
$ fleetctl list-units

```

![Server setup](img/image00156.jpeg)

If you see `activating start-pre`, give the `fleet` units a few minutes to pull the remote Docker images.

You can check the status of the `fleet` units using the following command:

```
$ fleetctl status dbuilder.service

```

![Server setup](img/image00157.jpeg)

Suppose we try again in a couple of minutes:

```
$ fleetctl list-units

```

![Server setup](img/image00158.jpeg)

Then we can see that we've successfully got two new `fleet` units on our new `tsc-registry-cbuilder1` server.

You might remember from the previous chapter that the `set_cluster_access.sh` script does the following:

*   It sets `PATH` to the `~/coreos-tsc-gce/bin` folder so that we can access executable files and scripts stored there from any folder
*   It sets `FLEETCTL_TUNNEL` to our `control/etcd` machine's external IP
*   It prints machines at the cluster with `fleetctl list-machines`
*   It prints units at the cluster with `fleetctl list-units`
*   It allows us to work with a remote `etcd` cluster via a local `fleetctl` client

# Summary

In this chapter, you learned how to deploy code on a remote Test/Staging cluster on GCE, and set up the Docker builder and private Docker registry machine.

In the following chapter, we will cover these topics: using our Staging and Docker builder and private registry servers to deploy code from Staging to production, building Docker images, and deploying them on production servers.