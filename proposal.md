
# r-hub: the everything-builder the R community needs

Gábor Csárdi, based on previous versions by J.J. Allaire (RStudio), Ben
Bolker (McMaster University) Dirk Eddelbuettel (Debian), Jay Emerson (Yale
University), Nicholas Lewin-Koh (Genentech), Joseph Rickert (Revolution),
David Smith (Revolution), Murray Stokely (Google), and Simon Urbanek (AT&T)

> July 21, 2015

## Introduction

The infrastructure available for developing, building, testing, and
validating R packages is of critical importance to the R community.  CRAN
and R-Forge has traditionally met these needs, however the maintenance and
enhancement of R-Forge has significant costs in both money and time.  This
proposal outlines a service that is complementary to CRAN and R-Forge, that
would add capabilities, improve extensibility, and create a platform for
community contributions to r-hub itself.

## Goals

1. Services that ease _all_ steps the R package development process, from
   creating a package to publishing and maintaining it.
2. Make these services free for all members of the community.
3. Allow community contributions to r-hub itself.
4. Make CRAN maintainers' work easier by pre-testing CRAN package
   submissions.

## Design principles

1. Reuse as much as possible, especially the tools and services
   already widely used among R package developers.
2. Keep compatibility with current systems.
3. Cover as much of the user base as possible.
4. Create open systems: provide APIs, open source all code for the service,
   accept contributions to it.  Make all software open source on GitHub.
   GitHub can be used with and svn natively, and also with Mercurial
   with a plugin.
5. Automated system. Make the daily operation of r-hub fully automatic,
   without any human intervention needed.
6. Cost-effective services. Rent machines from the cloud, if possible.
   Adapt number and type of machines to the current workload.
7. Make it possible for organizations and individuals to contribute (the
   cost of) machines to the build farm.
8. Make services reproducible.  It should be possible to start up an
   instance of any r-hub service within minutes, for anyone, even
   on their own laptop. This is crucial for development, testing and
   user contribution.
9. Separation is key, so use containers and reproducible VM images as much
   as possible.

## Essential services

### Build server

To check and build R packages. It also creates binaries for various
platforms, including the currently supported CRAN platforms, and
potentially others:
- MS Windows
- OSX, each flavor supported by CRAN
- Linux flavors
- Solaris

Users can upload their packages through a web page, or throught an HTTP
API, potentially select platforms to build on. They are provided with a
randomized URL where they can check the status of their builds, see their
output, and they can also cancel the build.

### Continuous integration for R packages

Make the build server work as a continuous integration service, free to use
for all R community members, integrated into GitHub and other popular
source code repositories. It will work exactly the same way as Travis or
other GitHub integrated CI services, but specialized for R packages: better
startup time with using caching and binary builds of popular or all
packages, no or minimal configuration for the users.

### Distribution of R package sources and binaries

Store built source and binary packages in a repository from which they can
be installed easily. This service can be used for distributing development
versions of CRAN R packages, or for R packages that are not (yet) fit to be
distributed on CRAN.

### Community web site

To showcase the packages, and to integrate all services. Users can browse
packages, search for packages, search the source code, view build
statistics, link to GitHub issues.

## Implementation details and timeline

### Build server: Jenkins in a container (week 1)

[Jenkins][jenkins] server, running in a Docker container. We need one
master server to start with. We will run it in the cloud, initially on
DigitalOcean or AWS. We will use [dokku][dokku] to manage it (and also other
microservices in containers). We will probably switch to something more
integrated in the future, like [Kubernetes][kubernetes] on
[CoreOS][coreos], but initially dokku will be enough.

We will not expose Jenkins directly to the users, but both users and admins
will access it through an HTTP API (the r-hub API). The reason for this is
that we might need to use something else than Jenkins for Solaris, and that
in the future we might need multiple Jenkins servers.

Initially the server provoding the r-hub API is very simple, and
will serve as a proxy to Jenkins.

[jenkins]: https://jenkins-ci.org
[dokku]: https://github.com/progrium/dokku
[kubernetes]: https://github.com/googlecloudplatform/kubernetes
[coreos]: https://coreos.com/

The actual build submissions to Jenkins will be handled by another process,
so that the web app can handle requests promptly. The web app simply puts
down the file into a shared folder (putting the random Id in the
file name to make sure it is unique), and then adds the a request including
the filename and and the random build id to a RabbitMQ queue. Another
process picks up jobs from the queue, and then communicates with Jenkins
to add a job, and then start the build. Query parameters are also
passed to the queue.

### r-hub API (weeks 1-2)

An HTTP API to manipulate Jenkins and the builds. The protocol is in JSON.
Initial design is very simple, this is sufficient for the web-based
submissions.

#### Builds

- `PUT /jobs/submit/tarball`

    Submit a source R package to be checked and built. The server responds
	with a JSON string including a random ID that can be used to query the
	submission later. Query parameters can be used later to select build
	OS, email notification, etc.

- `GET /jobs/info/<id>`

    Query the status of a submission. The server responds with a status
    code (e.g. SUBMITTED, QUEUED, DONE, etc), and if the build is already
    generating output, a summary of the output as well. The response also
    contains info about the build machine, if that is known already.

- `GET /jobs/output/<id>`

    Query the console output of the check.

#### Worker machines

- `GET /workers`

    Query the list of active worker machines.

- `PUT /worker`

    Instruct Jenkins to create another worker. It's parameters are
    supplied in the body of the request, in JSON.

### Jenkins Linux workers (weeks 2-3), backends

We will support different backends for workers. Initiallly there
will be two backends: local, DigitalOcean or AWS.

The local backend is for development and testing, it creates a
worker on the local machine. DigitalOcean has an API to create a droplet
with Docker support, AWS has one as well, obviously.

The DigitalOcean/AWS worker is mainly for production (but can be used for
testing). Jenkins has a Docker plugin that can create the container, once
a Docker environment is set up.

### System requirements (week 3)

Create a database of system requirements, in the spirit of
https://github.com/metacran/sysreqs The workers need to use this
database.

Repositories needed so far:

* `rhub-jenkins` The Docker config for our Jenkins instance. It
    contains the Jenkins config as well.
* `rhub-app` The web app to submit and query jobs, in node.js.
* `rhub-queue` The app that handles the queue, picks up the files,
    creates Jenkins jobs, and starts the build.
* `rhub-worker-*` One repo for the Docker config for each worker type.
    I.e. one for `r-devel-linux-x86_64-debian-clang`, etc.
* `rhub-backend-local` The local backend. All the code needed to create
    a local worker, and also the code needed to destroy it. In node.js,
    probably, so that the app can easily use it.
* `rhub-backend-do` DigitalOcean backend to create/destroy workers on DO.
* `rhub-backend-aws` Backend to create/destroy workers on AWS.
* `rhub-issues` A repository for user feedback.
* `sysreqs` Database of system requirement mappings.
* `sysreqs-app` Web app with an API for system requirements.

### Online submission system (week 4)

Orchestrate, document, test and make public the web app for submitting
builds for Linux machines. At this point, we have the Linux-builder
equivalent of Win-builder, as an open, extensible system.

### Redundant services, logging and monitoring (weeks 5-6)

For logging, redirect all logs through the network to a single log
server. These logs do not include console output from builds, those
are handled separately. This is only about the r-hub service logs.

For build logs, at this point of the project, we will just leave them
in Jenkins for a while, and then remove them (and the whole job, actually)
after a 3-5 days period.

Create a [sensu][sensu] dashboard and the required reporters, to
oversee all services.

Redundancy: to be worked out.

[sensu]: https://sensuapp.org/

New repositories:

* `rhub-monitor` Config for running the monitoring server.
* `rhub-logger` Config for running the logger.
* `rhub-dashboard` Web-app to view the actual monitoring dashboard.

### Windows workers (weeks 7-8)

Jenkins also has an [Azure plugin][jenkins-azure]. We can maybe use that,
if creating the workers through Jenkins is OK. More likely, we can just
use a client library to create/destroy the workers, e.g. the
[azure package][node-azure] from node.js

Users will be able to select windows in the submission web app, and
builder selection will be part of the r-hub API as well. At this point the
user will have to select a single builder, I think. Multiple builders
with a single submission will be added later.

Alternatively, we can also run windows on AWS. If the Linux deployment
will be on AWS, then it makes sense to use AWS workers for Windows as
well.

New repositories:

* `rhub-backend-azure` The Azure backend.
* `rhub-backend-aws-windows` AWS windows backend.

[jenkins-azure]: https://github.com/jenkinsci/azure-slave-plugin
[node-azure]: https://www.npmjs.com/package/azure

### On-demand workers (week 9)

### OSX slaves (weeks 10-11)

### CRAN presubmission (week 12)

### CI for GitHub projects (weeks 13-14)

### Open, documented API (week 15)

### Community website (weeks 16-17)

### Reverse dependency checks (week 18)

### Solaris builds (week 19)

### Specialized builds (valgrind, ASAN, UB-SAN, etc.) (week 20)

## Costs
