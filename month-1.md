
# Plans for the first month of r-hub

Note: this document follows mostly the original proposal. One week
in the proposal's timeline corresponds to two weeks here, as I will
work for only 87 hours per month (roughly 50%) on the project.

## Introduction

The goal of the first ~5 weeks is to have a Linux-builder web-application
and back-end up and running. Users can submit builds via a web page,
the build results are stored on a static web server. Users
get e-mail notifications about the results. Very much like the current
win-builder, but for Linux, for now one specific distribution of Linux.

## Deliverables

### 1 Jenkins DockerFile, tailored to r-hub

### 2 r-hub application, front-end

The web application that currently only handles the web uploads. It hands over
the submissions to the back-end, by putting them in a message queue,
and storing the submitted source packages in a shared filesystem.

This is a node.js application and it will run in a Docker container.

### 3 r-hub application, back-end

This is the application that coordinates the builds. Currently, it picks up
submissions from the message queue, one by one, and adds them as a new Jenkins
job. It also "contains" the message queue itself.

For now it works with a fixed number of builders, all added to Jenkins
manually.

The back-end is also responsible for the (temporary) storage of the source and
binary packages.

This is a node.js application, and it will run in a Docker container.

### 4 A running instance of our Jenkins container

Managed via Dokku.

### 5 A running instance of the r-hub front-end

Managed via Dokku.

### 6 A running instance of the r-hub back-end

Managed via Dokku.

### 7 DockerFile for an Ubuntu Linux builder

We can use one with the most recently released R version, i.e. R 3.2.2.
Most probably we can just take the container from https://github.com/rocker-org.

### 8 A running instance of a Linux builder

A generic Linux server running 2-4 Linux builder containers,
all managed via Dokku. The containers connect to the Jenkins server
automatically.

### 9 A GitHub repository for r-hub's documentation

Using markdown documents.

### 10 A GitHub repository for user feedback

This repository is linked to from the submission page, from the
emails sent to users, etc.

### 11 r-hub controller

Work out a method to control the r-hub micro-services. Look for a simple
solution that is possibly compatible with dokku. The controller is responsible
for supplying the configuration information to all r-hub containers.

Possible solutions to evaluate:
 * Google Kubernetes
 * Helios from Spotify
 * Docker compose
 * Flynn
 * Decking
 * Deis

### 12 A method for storing sensitive information on GitHub

There are already methods for this, of course. Ideally we want something
simple and lightweight. I.e. the whole repo is encrypted, and can be decrypted
by any admin keys. https://github.com/hadley/secure contains most of what we
need, but we probably need to access the encrypted secrets from the command
line without using R, and probably also from JavaScript.
