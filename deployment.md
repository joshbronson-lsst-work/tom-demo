# Overview

These instructions will assist you in creating an externally facing
TOM. After following these instructions, you should have a TOM
listening at the address of your choice.

# Concepts

The following is a rough system diagram of the TOM system when it is
dedployed to a Kubernetes cluster. Each layer is implemented, roughly,
in terms of the lower-level constructs beneath it. Our deployment of
TOM, here, is implemented in terms of Kubernetes.

From a high-level perspective, Kubernetes allows for the orchestration
databases, servers, and other familiar programs. Orchestration means
managing the lifecycle of these programs, managing their
configuration, handling the creation of connections between them, and
otherwise facilitating their creation and communication with one
another.

These programs are defined in containers, which, when they are running
in Kubernetes, are called pods. Pods can be annotated so that
Kubernetes knows which of them run on ports. Kubernetes uses Docker, a
lightweight framework for containerized processes, which allows it to
treat relatively small pieces of code as independent machines with
independent operating systems, complete with all of the dependencies
they need and the ability to communicate with each other like physical
machines.

This flexiblity allows for a good deal of control over the environment
of the processes deployed to Kubernetes clusters. It is easy to "turn
it off and on," which helps make deployments reproducible. Kubernetes
and Docker, similar to Java, seem to want to allow their users to
"write once and run anywhere," and while they do not succeed, they
eliminate some of the complexity of worrying about the underlying
system's package versions, networking architecture, and other details.

Kubernetes and Docker are, in turn, implemented in terms of
lower-level platforms. While differences between these platforms will
occasionally govern the use of different Kubernetes structures, the
idea behind Kubernetes is to abstract over these differences as much
as possible, minimizing the pain of migration.

                                                +---------+    +-------------+    +----------------+
                    +------+     +---------+  /-+ service +----+ TOM backend +--+-+ object storage |
                    | user +-----+ ingress +--  +---------+    +-------------+  | +----------------+
                    +------+     +---------+                                    |
																			    |
                                                                                | +----------+
    Deployment                                                                  +-+ Postgres |
                                                                                  +----------+



                                                       +---------------------+
    Orchestration                                      | Kubernetes / Docker |
                                                       +---------------------+



                        +-------------------------+ +-----------------+ +---------------------+ +---------------+
    Platform            | Google Compute Platform | | Microsoft Azure | | Amazon Web Servcies | | Local Cluster |
                        +-------------------------+ +-----------------+ +---------------------+ +---------------+

Helm is a relatively thin layer on top of Kubernetes that improves the
ability to package and configure Kubernetes deployments.

# Setup

## Squarespace for DNS

If you have the ability to create DNS A records, don't worry about
this step. Furthermore, alternatives to Squarespace exist, and it will
be much easier to follow these instructions with an alternative DNS
provider than an alternative cloud provider.

But, if you need a domain, visit https://domains.squarespace.com, or
Google for "squarespace domains." Then search for a domain you'd like
to use; evaluate the terms, conditions, and price; and, if you'd like,
buy it!

## Google Compute Platform

These instructions are, for now, heavily tied to Google Compute
Engine. It will be tough to use alternative resources, so to follow
along, you will need an account.

### Account and Billing Setup

Once you've decided you would like to create a Google Compute Platform
account, navigate to https://cloud.google.com, or search for Google
Compute Platform. If the terms and conditions are acceptable to you,
create your account.

After you've created an account, set up billing. When the author
created an account in September 2025, billing could be started by
clicking on a button that said "Try for free." Setting billing up
still required a credit card.

In the dropdown boxes, Google asked the following questions, an the
author answered them as follows:

* Question: How would you like to get started today? Answer: Build
  production-ready solutions.
* Question: What do you want to do with Google Cloud first? Answer:
  Build or deploy web or mobile applications.
* Question: What are you trying to do with apps or websitse? Answer: I
  want to host a website.

The author is not sure what, if anything, would have been different if
the questions had been answered differently.

### Gcloud Commandline Tool Installation

Next, install the [https://cloud.google.com/sdk/docs/install](gcloud
commandline tool). There are multiple options available. I chose to
click on the targball for my system,
gcloud-cloud-cli-linux-x86_64.tar.gz, and untar it and install:

    tar zxf google-cloud-cli-linux-x86_64.tar.gz
    bash ./google-cloud-sdk/install.sh

Ensure that the tarball you choose is correct for your platform. After
that, you should have access to the `gcloud` command.

### Gcloud Commandline Tool Authorization

In order to use the `gcloud` command to control Google Compute
Platform objects, you'll need to authenticate, with the following
command:

    gcloud auth login

# Configuration

First, you may choose to edit the configuration file, which you can
find at [scripts/common_config.sh](common_config.sh). Most of the
defaults in there should be reasonable for a demo, but at a minimum
you will need to edit the tom_hostname variable. It should be the
fully qualified domain name of your server.

# Run the Scripts

The orchestration scripts, which reside in the scripts directory of
this repository, can be run standalone, but they also include comments
that help understand the process of deploying to the Google Compute
Platform and running Kubernetes on top of that.

After everything above has been run and configured, it should be
possible to simply run the scripts. First, create the Kubernetes
cluster inside Google Compute Engine. This is a blank slate on which
Kubernetes can deploy its objects:

    bash scripts/create_kubernetes_cluster_gcp.sh

It is possible that something in that script will fail due to changes
in the Google Compute Platform API, differences in your environment,
changed configuraiton, or other issues. If it fails, look at the
comments near the commandline that failed. If you are able to resolve
the issue, you should be able to simply rerun the script, which will
pick up where it left off. If the error messages you see are related
to timeouts, for example, it may make sense to simply try rerunning
the script once.

Once that is complete, deploy the Kubernetes cluster:

    export tom_hostname=TARGET_HOSTNAME_HERE
    export certmanager_email=YOUR_EMAIL_HERE 
    bash scripts/launch_kubernetes.sh

Similarly, it should be possible to continuously rerun the script
while fixing any issues that arise while running it.

# Create the Administrative Account

Create the administrative account! kubectl transparently manages
authentication for you. This command runs the manage.py
createsuperuser command, which creates a superuser for django. After
this, it should be possible to log in as this superuser.

	. scripts/common_config.sh
    kubectl -n "$kubernetes_namespace" exec -it deploy/demo-tom-demo -c tom-demo -- sh -lc 'python manage.py createsuperuser'

You only need to perform this action once after the server is running.

# Connecting to Your Instance

## Retrieving Your Static IP

After your cluster has started, you can use the following command to
retrieve the static IP address assigned to your django server

    . scripts/common_config.sh ; kubectl -n "$kubernetes_namespace" get ingress 

The IP address will be in the ADDRESS column.

## Configuring DNS

In Squarespace, or in the DNS tool of you choice, you can create a DNS
A (for Address) record that maps a human-readable domain name to the
address above.

If you are using Squarespace, log in, navigate to your account's
domains, click on the domain you want to use, and click "DNS." There
you should be presented with DNS settings. There is a section for
"Custom Records" at the bottom, and there is a button that says "ADD
RECORD." Click that button.

- There is an text box for HOST. If your domain is foo.com and you
  want to create a DNS entry for bar.foo.com, just enter "bar"
  here. You don't need the fully qualified name, at least not for
  Squarespace.
- In the "TYPE" selector, choose "A".
- In the "TTL" selector, choose 4 hours.
- In the "DATA" text box, enter the IP address.

The TTL selection will control how long the record is cached. If you
change the IP address for this record, various caches between you and
the main DNS server, including caches on your computer, may store the
record for this long.

## Connecting to Your Instance

It may take a moment for your instance to work. When you navigate to
the site, you should initially see a privacy warning. Clicking through
to retrieve information about the certificat, you should see that the
name of the certificate authority has (STAGING) in its name. That's
because the Let's Encrypt certificate we are using by default points
to the staging environment. To change that, edit
[scripts/common_config.sh](common_conifg.sh) and edit the
`letsyncrypt_env` variable. Change its default to
`prod`. Alternatively, you can export it, but you must remember to do
so each time you run launch_kubernetes.sh

It will take a few minutes for the production TLS cert to function,
but at this point you should be able to navigate to the hostname you
chose and log in with the administrative username and password you
selected.
