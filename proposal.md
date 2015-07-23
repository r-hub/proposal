
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
proposal outlines r-hub, a service that is complementary to CRAN and
R-Forge, that would add capabilities, improve extensibility, and create a
platform for community contributions to r-hub itself.

## Goals

1. Services that ease _all_ steps the R package development process, from
   creating a package to publishing and maintaining it.
2. Make these services free for all members of the community.
3. Allow community contributions to r-hub itself.
4. Make CRAN maintainers' work easier by pre-testing CRAN package
   submissions.

## Design principles

1. Reuse as much existing technology as possible, especially the tools and
   services already widely used among R package developers.
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
other GitHub integrated CI services, but will specialize for R packages:
better startup time using caching and binary builds of popular or all
packages, no or minimal configuration requirements for the users.

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

The actual build submissions to Jenkins will be handled by another process,
so that the web app can handle requests promptly. The web app simply puts
down the file into a shared folder (putting the random Id in the
file name to make sure it is unique), and then adds the a request including
the filename and and the random build id to a RabbitMQ queue. Another
process picks up jobs from the queue, and then communicates with Jenkins
to add a job, and then start the build. Query parameters are also
passed to the queue.

[jenkins]: https://jenkins-ci.org
[dokku]: https://github.com/progrium/dokku
[kubernetes]: https://github.com/googlecloudplatform/kubernetes
[coreos]: https://coreos.com/

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

- `DELETE /worker/<id>`

    Remove a worker.

### Jenkins Linux workers (weeks 2-3), backends

We will support different backends for workers. Initiallly there
will be two backends: local, DigitalOcean or AWS.

The local backend is for development and testing, it creates a
worker on the local machine. 

The DigitalOcean/AWS worker is mainly for production (but can be used for
testing). DigitalOcean has an API to create a droplet with Docker support,
AWS has an API as well.  Jenkins has a Docker plugin that can create the
container, once a Docker environment is set up.

Each platform needs to be able to install a recent version of
R-devel (and R-patched). For the Linux workers, a permanent Jenkins job
will build R-devel daily, in a seperate container, and then distributes the
(successful) build via HTTP. The worker containers download the binary
R-devel versions upon startup, and they also check if they have the most
recent version at the beginning of each build. This is a simple and cheap
HTTP HEAD query.

Current Linux platforms on CRAN:

- `r-devel-linux-x86_64-debian-clang` (Debian testing)
- `r-devel-linux-x86_64-debian-gcc` (Debian testing)
- `r-devel-linux-x86_64-fedora-clang` (Fedora 21)
- `r-devel-linux-x86_64-fedora-gcc` (Fedora 21)
- `r-patched-linux-x86_64` (Debian testing)
- `r-release-linux-x86_64` (Debian testing)

Other popular Linux platforms we want to support:

- `r-devel` on most recent Ubuntu LTS
- `r-release` on most recent Ubuntu LTS
- `r-devel` on stable Debian
- `r-release` on stable Debian

### System requirements (week 4)

Create a database of system requirements, in the spirit of
https://github.com/metacran/sysreqs The workers need to use this
database.

Repositories needed so far:

* `rhub-jenkins` The Docker config for our Jenkins instance. It
    contains the Jenkins config as well.
* `rhub-app` The web app to submit and query jobs, in node.js.
- `rhub-api` Web pages that document the API.
* `rhub-queue` The app that handles the queue, picks up the files,
    creates Jenkins jobs, and starts the build.
* `rhub-worker-*` One repo for the Docker config for each worker type.
    I.e. one for `r-devel-linux-x86_64-debian-clang`, etc.
* `rhub-backend-local` The local backend. All the code needed to create
    a local worker, and also the code needed to destroy it. In node.js,
    probably, so that the app can easily use it.
* `rhub-backend-do` DigitalOcean backend to create/destroy workers on DO.
* `rhub-backend-aws` Backend to create/destroy workers on AWS.
* `sysreqs` Database of system requirement mappings.
* `sysreqs-app` Web app with an API for system requirements.
* `rhub-issues` A repository for user feedback.

### Online submission system (week 5)

Test the Linux builders and the system requirements by building
all CRAN packages (that are available on Linux). Fix errors, 99%
of the CRAN packages are expected to build correctly.

In general, orchestrate, document, test and make public the web app for
submitting builds for Linux machines. At this point, we have the
Linux-builder equivalent of Win-builder, as an open, extensible system.

### Redundant services, logging and monitoring (weeks 6-7)

For logging, redirect all logs through the network to a single log
server. These logs do not include console output from builds, those
are handled separately. This is only about the r-hub service logs.

For build logs, at this point of the project, we will just leave them
in Jenkins for a while, and then remove them (and the whole job, actually)
after a 3-5 days period.

Create a [sensu][sensu] dashboard and the required reporters, to
oversee all services.

Redundancy: workers are disposable, so we don't need to worry about them.
The web app can be replicated on a second server and then load balanced
using AWS services. The same applies to the queue manager.

[sensu]: https://sensuapp.org/

New repositories:

* `rhub-monitor` Config for running the monitoring server.
* `rhub-logger` Config for running the logger.
* `rhub-dashboard` Web-app to view the actual monitoring dashboard.

### Windows workers (weeks 8-9)

Users will be able to select windows in the submission web app, and
builder selection will be part of the r-hub API as well. At this point the
user will have to select a single builder. 

We can also run windows VMs on AWS.

For windows, a daily build of R-devel is available from r-project.org and
the windows workers just download this.

We will use code from the [r-appveyor][r-appveyor] project to build the
windows worker backend.

New repository:

* `rhub-backend-aws-windows` AWS windows backend.

[jenkins-azure]: https://github.com/jenkinsci/azure-slave-plugin
[node-azure]: https://www.npmjs.com/package/azure
[r-appveyor]: https://github.com/krlmlr/r-appveyor

### OSX slaves (weeks 10-11)

For legal reasons, OSX workers need to run on Apple hardware. Many
companies offer OSX as IaaS, and this seems to be the simplest and cheapest
solution for us. Most likely we will rent one or two Mac Mini servers with
16GB memory from https://macstadium.com, unless we get a better offer. This
server comes with a VMware ESXi v6.0 Hypervisor, so we run multiple
instances of OSX on it, in virtual machines. We create snapshot images for
the latest two or three OSX versions, and we start each R package build
from a snapshot image, assuming that startup takes a reasonable time
(1-2 minutes maximum). If it takes longer, then we will look into running
the build as a restricted user, clean up after the build, reuse the VM
for multiple builds, and only reboot/recreate the VM once a day.

Jenkins supports ESXi via a plugin, or we can just use the native ESXi API
directly from our web-app middleware, to manage the VMs.

For OSX, a daily build of R-devel is available from r-project.org and the
workers just download this and install it to a read-only shared area,
so that the worker images do not need to be rebuilt daily.

New repositories:

* `rhub-backend-osx`

### On-demand workers (week 12)

While we strive to have a quick build startup time, we want to make
sure that workers are not idle for long. For OSX at https://macstadium.com,
flexible scaling is not available for the rented Mac Mini servers.
For Linux and Windows workers, they are typically billed by the hour.
We will work out a simple heuristics to shut down idle Windows and Linux
workers. This could work from within Jenkins or the r-hub application.

### CRAN presubmission (starting from week 13)

Work with CRAN on the integration of r-hub into the CRAN submission
process. At present, 80% of the CRAN submissions are turned down. We
set out to reduce this to 20%, by providing feedback to package developers,
prior to submitting to CRAN.

The CRAN submission process will have the following steps:
- User submits through web-app on r-hub.
- Package is built and checked on Linux with r-devel.
  If there are warnings or errors, the submission is rejected.
- Package is built and checked by r-hub, on all CRAN platforms. If there
  are warnings or errors, the package is rejected.
- The submission is sent to CRAN, including all checks. *New* warnings and
  errors are marked for CRAN maintainers. They have the possibility to
  accepts the submission, and then the source package moves to CRAN's
  system. Otherwise CRAN maintainers can send back an email to the
  package maintainer, including all build logs, and possibly their
  own annotation.

### CI for GitHub projects (weeks 14-15)

An R specific CI service, with GitHub integration. It should be completely
automatic, with some sane defaults, e.g. by default build and check
with r-devel, on Linux (Debian), Windows and most recent OSX platform.
Other, more exotic platforms can be selected via configuration, or
as one-time builds from the web app or the API.

Having a CI also means that we will have projects, and need a database
to store their settings, build history, etc. We will use a MongoDB database
for this, except for the build output, which will be compressed and stored
in a key-value datastore, e.g. Redis or something more lightweight.

Repository:

- `rhub-ci` Web app that handles the hooks from GitHub (and possibly other
  services in the future).

Binary and source packages from GitHub will be stored on a storage
server in a CRAN-like repository, and we will open this repository for
public use.

Design a system to handle dependencies among packages stored at CRAN,
r-hub, BioConductor, GitHub, etc. This should be part of the DESCRIPTION
file, and handled by the r-hub CI automatically.

### Open, documented API (week 16-17)

Document and make public the r-hub CI service, including an API to
add/remove jobs, run/query builds. Stabilize services. Make sure that
almost all CRAN packages build on all three major platforms.

Repository:

- `rhub` R package, to query the r-hub API: submit packages to
  build, search packages, etc.

### Community website (weeks 18)

Simple web-site, where users can browse and search packages, both
in the r-hub reposotory and CRAN. It will be integrated into the already
existing web pages.

### Reverse dependency checks (week 19)

We will include the check of reverse dependencies in CRAN pre-submissions.
We compare the new check results to old ones, and include the differences
(i.e. newly appearing warnings and errors), in our email notification to
CRAN, if a package maintainer decides to continue with the submission,
even if reverse dependencies break.

### Solaris builds (week 20)

We will look into having Solaris slaves. This is really only important
for packages with compiled code. It is also non-trivial, for two reasons.
First, Jenkins does not officially support since late 2014. Second, ideally
we would need Solaris Sparc as well, as it has a different byte-order than
all other CRAN platforms, and many errors only come up there.

We can run Solaris x86 on AWS in a VM, and we can rent a Solaris Sparc in
the cloud, and we can use Solaris Zones containers on it.

### Stabilize, publicise r-hub (weeks 21-24)

These four weeks also serve as reserves, if any of the subprojects
run out of time, which is likely.

## Costs

Some facts first, we base our cost estimtes on these:

* There are about 7,000 packages on CRAN, and we estimate that this number
  will double in five years, to about 14000 packages.
* About 30 packages are updated on CRAN every day, we estimate that this
  number will grow to 60 in the next five years.
* A CRAN package has on average 2.65 hard (Depends, Imports) dependencies.
* There are about 100,000 repositories on GitHub that are classified as
  written mainly in R. Out of them about 2,000 are currently using Travis CI.
* On a modern processor (e.g. Xeon 2.9GHz) it takes on average 30 seconds to
  build and check a CRAN package on Linux, and about twice as long on OSX
  and three to five times as long on Windows. These numbers assume that
  binary packages are available, and no container startup time.

Our estimates for the number of daily builds and required CPU time one year
after the start of the project, assuming r-hub is integrated into the CRAN
submission process and it is popular on GitHub as a CI service:

* 35 successful CRAN submissions, and 200 unsuccessful ones, a total number
  of 235 submissions, triggering 235 direct Linux builds on Debian, and
  about 50 builds on other Linux flavours and other OSes. This also
  triggers about 150 reverse dependency checks on the Debian Linux.
* We estimate 1,000 CI builds from GitHub, although this number might be
  highly inaccurate, as we have essentially no real data on this.
  (The mean number of ~5 pushes per R GitHub repository is useless, as
  most of these repositories are essentially inactive.) The 1,000 CI builds
  will be done on Debian Linux, Windows and the most recent OSX platform.
* This is a total of 1,385 builds on the primary Linux platform, 1,050
  builds on the primary Windows and primary OSX platforms, and
  50 builds on other platforms.
* Assuming 2 minutes container startup time, this amounts to about 60 CPU
  hours for the primary Linux platform, 50 CPU hours for the primary OSX
  platform and 70 CPU hours for the primary windows platform. For other
  platforms the required resource are negligible.

These estimates are the lower bounds for running a useful service. They do
not contain builds that are stuck and until the end of the build time limit
(20-40 minutes), and they do not contain CPU time required to build R-devel
daily, etc. We estimate that these are negligible compared to the total
cost.

Cost for development work:
* $100 per hour, total of 1000 hours over 9-12 months. $100,000 total.

Yearly costs:
* Domain name registration, virus scanner for the Windows builder,
  SSL certificates, maximum $500 total.

Ongoing operational costs, from month one:
* Development and production environments, Jenkins server and web
  server hosting, all server side applications, about $100 per month.

From month two to six:
* Linux build servers: $100 per month, $10 per platform.
* OSX build servers: one mac mini server $120 per month.
* Windows servers: four t2.medium instances on AWS, $210 per month.
* Solaris server: Solaris Sparc $300 per month, Solaris x86 $40 per month.
* Storage server for binary and source packages: under $200 per month.

Total monthly operational cost: $1112 per month.
