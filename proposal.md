
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
   accept contributions to it.
5. Automated system. Make the daily operation of r-hub fully automatic,
   without any human intervention needed.
6. Cost-effective services. Rent machines from the cloud, if possible.
   Adapt number and type of machines to the current workload.
7. Make it possible for organizations and individuals to contribute (the
   cost of) machines to the build farm.
8. Make all software open source on GitHub.  GitHub is currently the main
   point for developing open source software, so it is an easy choice.
9. Make services reproducible.  It should be possible to start up an
   instance of any r-hub service within a few hours, for anyone.
10. Separation is key, so use containers and reproducible VM images as much
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

Jenkins [1] server, running in a Docker container. We need one master
server to start with. We will run it in the cloud, initially on
DigitalOcean. We will use dokku [2] to manage it (and also other
microservices in containers). We will probably switch to something more
integrated in the futre, like Kubernetes [3] on CoreOS [4], but initially
Dokku will be enough.

We will not expose Jenkins directly to the users, but both users and admins
will access it through an HTTP API (the r-hub API). The reason for this is
that we might need to use something else than Jenkins for Solaris, and that
in the future we might need multiple Jenkins servers.

Initially the server provoding the r-hub API is very simple, and
will serve as a proxy to Jenkins.

[1] https://jenkins-ci.org__
[2] https://github.com/progrium/dokku

[3] https://github.com/googlecloudplatform/kubernetes

[4] https://coreos.com/

### r-hub API (weeks 1-2)

### Jenkins Linux workers (weeks 2-3)

### Online submission system (week 4)

### Reduntant services, logging and monitoring (weeks 5-6)

### Windows workers (weeks 7-8)

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
