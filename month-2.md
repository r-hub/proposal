
# Plans for the second month of r-hub

Note: this document follows mostly the original proposal. One week
in the proposal's timeline corresponds to two weeks here, as I will
work for only 87 hours per month (roughly 50%) on the project.

## Introduction

This month will focus on three themes:
* Bringing up a working web-submission based system, with several Linux
  builders.
* Handling system requirements.
* Redundant services, logging and monitoring.

## Deliverables

### 1 A running instance of our Jenkins container

From month 1.

### 2 A running instance of the r-hub front-end

From month 1.

### 3 A running instance of the r-hub back-end

From month 1.

### 4 DockerFile for an Ubuntu Linux builder

From month 1.

### 5 A running instance of a Linux builder

From month 1.

### 6 Builder for several Linux flavours

Current Linux platforms on CRAN:

* `r-devel-linux-x86_64-debian-clang` (Debian testing)
* `r-devel-linux-x86_64-debian-gcc` (Debian testing)
* `r-devel-linux-x86_64-fedora-clang` (Fedora 21)
* `r-devel-linux-x86_64-fedora-gcc` (Fedora 21)
* `r-patched-linux-x86_64` (Debian testing)
* `r-release-linux-x86_64` (Debian testing)

Other popular Linux platforms we want to support:

* `r-devel` on most recent Ubuntu LTS
* `r-release` on most recent Ubuntu LTS
* `r-devel` on stable Debian
* `r-release` on stable Debian

### 7 Database of system requirements

Create a database of system requirements for CRAN packages. This work
has already started at https://github.com/r-hub/sysreqs.
The builders need to use this database to install additional software
in the container. Caching these software would be ideal but looks challenging.

### 8 Precompiling and caching CRAN packages

Create a volume on AWS and/or Azure that stores the precompiled
binaries of all CRAN packages, for all supported Linux platforms.
Fix the system requirement database and the build system based
on the errors we get while compiling the CRAN packages.

### 9 Logging

Implement logging for all services, to a centralized logger machine.

### 10 Monitoring

Design a sensu dashboard for the r-hub services.
