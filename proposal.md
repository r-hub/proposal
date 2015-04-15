
# Next Generation R-Forge — Draft Proposal (incomplete)

> April 14, 2015

## Introduction

The infrastructure available for developing, building, testing, and
validating R packages is of critical importance to the R community. R-Forge
has traditionally met these needs, however the maintenance and enhancement
of R-Forge has significant costs in both money and time. This proposal
outlines a possible next-generation of R-Forge that would add capabilities,
improve extensibility, and create a platform for community contributions to
R-Forge itself.

## Goals

1. Services that ease _all_ steps the R package development process,
   from creating a package to publishing it and maintaining it.
2. Make these services free for all members of the community.
3. Allow community contributions to R-Forge itself.

## Design principles

1. Reuse as much as possible, especially the tools and services
   already widely used among R package developers.
2. Ease of use for users and developers.
3. Keep compatibility with current systems.
4. Cover as much of the user base as possible.
5. Create open systems: provide APIs.

## Essential services

### Build server and continuous integration for R packages

Check and build R packages. Create binaries for various platforms,
including the currently supported CRAN platforms, and potentially
others:
- MS Windows
- OSX, each flavor supported by CRAN
- Linux flavors
- Solaris

Packages would be submitted to the service in three main ways:

1. Manually, by uploading an R package through the web interface. This is most
   important for casual R developers and users, and should be designed with a
   minimum of options.
2. Via an REST API call, typically initiated from an R
   function. This should provide more options, ideally allowing selection of
   OS and R versions, and whether or not the build artifacts should be
   published.
3. Automatically initiated from a post-receive hook from a version
   control system.

#### Satisfying Dependencies

Dependencies for package builds could be satisfied from R-Forge itself,
CRAN, or Bioconductor. To specify explicitly which repositories to satisfy
dependencies from package authors could add a `Repositories` field to their
package `DESCRIPTION` file. We'd expect that fetching dependencies at build
time from CRAN will be fast via the use of the CloudFront mirror. For
Bioconductor, we may want to consider a local mirror.

#### Using Version Control Commit Hooks

Many version control systems including Subversion and Git
provide for a “post-commit” hook which notifies an arbitrary URL whenever
the repository is committed to. This is a useful framework which would make
it possible for users to configure automated checks and/or re-builds of
their packages whenever they commit to their repository.

### Distribution of R package sources and binaries

Store built source and binary packages in a repository
from which they can be installed easily.

### Community web site

To showcase the packages, and to integrate all services.

## Additional services

### R package manager

Create a package manager R package, to install from CRAN,
BioC and R-Forge2, with ease.

### RSS feeds

RSS feeds for updated/new packages, package authors, topics, etc.

### Binary builds for R itself

These are needed anyway, to do our builds.

### Documentation

Run package documentation and publish it online, on
a searchable site.

### Package search

Create a search engine that searches for packages.

### Code search

Create a search engine that searches for code in the packages.

## Implementation

### Principles

Separation is key, so use containers and reproducible VM images
as much as possible. This usually means running services in Docker
containers, at least for Linux. Windows and OSX would use VMs,
Solaris, too.

### Build server and continuous integration for R packages

Jenkins server, running in a Docker container. We need one master
server to start with, and a couple of slaves. Slaves will run in
containers/VMs:
- Linux slaves run in Docker containers, on DigitalOcean or Google Cloud
  or AWS, or essentially any service that runs Docker containers.
- Windows slaves can run in VMs on AWS, Google Cloud, or the Azure cloud.
- OSX slaves can maybe run on a hosted service, but this is not very
  likely. More likely, we need to buy OSX machine(s) and then run the
  slaves within VMWare VMs.
- Solaris slaves can run on AWS.

### Distribution of R package sources and binaries

We can store the binary and source packages in a CRAN-like repository,
on S3 or Google Storage, or Azure Storage is an option, too, as a
significant portion of binaries will be Windows, and built on the Azure
cloud anyway.

### Community web site

Community web site and the REST APIs will be served from from node.js
within Docker containers, e.g. using Dokku.
