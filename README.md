Next Generation R-Forge — Draft Proposal

January 28th, 2014

J.J. Allaire (RStudio), Ben Bolker (McMaster University) Dirk Eddelbuettel (Debian), Jay Emerson (Yale University), Nicholas Lewin-Koh (Genentech), Joseph Rickert (Revolution), David Smith (Revolution), Murray Stokely (Google), and Simon Urbanek (AT&T)

The infrastructure available for developing, building, testing, and validating R packages is of critical importance to the R community. R-Forge has traditionally met these needs, however the maintenance and enhancement of R-Forge has significant costs in both money and time. This proposal outlines a possible next-generation of R-Forge that would add capabilities, improve extensibility, and create a platform for community contributions to R-Forge itself.

Since the initial launch of R-Forge in 2007, the landscape of services and infrastructure for open source development has changed considerably. In 2007, it was important for a language-specific open source host to provide mailing lists, web hosting, issue tracking and source code control, as R forge did for the R community. Now there are many organizations that provide these services free to open source projects, including:

CloudForge (Git/Subversion)
Github (Git)
Assembla (Git/Subversion)
Google Code (Git/Mercurial/Subversion)
Bitbucket (Git/Mercurial)
Gitorious (Git)
Google Groups (discussions & mailing list)

Since the core infrastructure for version control and project collaboration is widely available, this proposal focuses on the components of R-Forge that cannot currently be easily replicated elsewhere:

Automated R CMD check
Binary builds
An easy way to install development binaries

This proposal describes an evolution of these core R-Forge capabilities that would provide enhanced automation, interoperability, and platform coverage. Rather than creating a monolithic service, we focus on creating core components that can be reused flexibly for other needs. For example, the components in this proposal also provide win-builder like functionality for every OS. It’s expected that developers could seamlessly integrate their version control system of choice with these components using the commit hooks available from most providers (see below for discussion of how this works).
Build Service Requirements
The goal of the build service is to make it easy to produce binary packages and run automated checks for a package across multiple platforms and versions of R. Such a build service by itself is a valuable contribution to the R community, because the infrastructure required to build R packages across the three main platforms and the released and development versions of R is difficult to set up. 

The build service would be composed of two parts: 

A front-end web application that makes it possible to submit new packages for building and checking, check status on existing submissions and see build and check results. The web app needs to support both human and computer readable output, and should provide a badge suitable for inclusion into a github readme. 
A back-end build queue which manages a queue of submissions and a build farm, allocating packages to be built across multiple servers

Proposed platform coverage for the build service would be Windows, Mac OS X, Linux, and Solaris. To provide coverage for the various permutations of Linux we may want to take advantage of Open Build Service (http://openbuildservice.org/) to allow us to target multiple distros and their variants from a single machine.

Since packages can execute arbitrary untrusted code, package building and checking will need to occur in a secure environment (e.g. Open Build Service sandbox, Linux Container, Solaris Zone, etc.).

Packages would be submitted to the service in three main ways:

Manually, by uploading an R package through the web interface. This is most important for casual R developers and users, and should be designed with a minimum of options.
Via an REST API call, typically initiated from an R function. This should provide more options, ideally allowing selection of OS and R versions, and whether or not the build artifacts should be published.
Automatically initiated from a post-receive hook from a version control system.

The basic process might look like:

Build server receives request to build and check a package.
The server responds with a status url which can be polled to get updates. 
R CMD check and R CMD build are run on the package for {Windows, Linux (various permutations), Solaris, OS X} * {r-release, r-devel}. 
Binaries and CRAN check results are uploaded to a stable storage location (Amazon S3 or Google Storage could be used for this). 
Build artifacts are deleted after 24 hours for arbitrary uploads, or when the next commit is received for github hooks.

The functionality above would reproduce what is currently available via R-Forge, but in addition make the build tools available to anyone (for example, Simon could use the build server for r-forge.net).

We may want to distinguish what binaries are produced and what output is available based on the source of the build request (i.e. ad-hoc builds may not preserve as much output and might be deleted more aggressively).

For builds originating from version control repositories it would be ideal to run R CMD check on every commit to the repository. Building binaries on all platforms could be done on a daily or otherwise more throttled basis. 
Satisfying Dependencies
Dependencies for package builds could be satisfied from R-Forge itself, CRAN, or Bioconductor.  To specify explicitly which repositories to satisfy dependencies from package authors could add a "Repositories" field to their package DESCRIPTION file. We'd expect that fetching dependencies at build time from CRAN will be fast via the use of the CloudFront mirror. For Bioconductor, we may want to consider a local mirror.
Using Version Control Commit Hooks
Many version control systems including Subversion and Git provide for a “post-commit” hook which notifies an arbitrary URL whenever the repository is committed to. This is a useful framework which would make it possible for users to configure automated checks and/or re-builds of their packages whenever they commit to their repository. Here is documentation on the post-commit hooks provided by three popular services:

CloudForge: https://help.cloudforge.com/entries/22483702-Using-Web-Hooks
Github: https://help.github.com/articles/post-receive-hooks
Gitorious: http://blog.gitorious.org/2013/07/12/gitorious-3-0-beta-1/ 

We may also wish to make pre-commit hooks possible, whereby a developer workstation could submit code to the build service for validation (e.g. does it build and pass R CMD check) prior to actually committing code to the repository.
Notes on Implementation
Open Source Software
The system described above would be developed and made available as open source software. Our hope would be to use the GPLv3 license however this is still to be determined based on the composition of other open source software used for the project. Some of the other open-source software components that we'd evaluate for use in the project include:

Open Build Service (http://openbuildservice.org/), a Linux build service (GPL v2)
Docker (https://www.docker.io/), a Linux container engine (Apache v2)
Vagrant (http://www.vagrantup.com/), a tool for creating portable and reproducible development environments (MIT)
Jenkins (http://jenkins-ci.org/), a continuous integration server (MIT)
Chef (http://www.opscode.com/chef/), an infrastructure automation platform (Apache v2)
Puppet (http://puppetlabs.com), an infrastructure automation platform (Apache v2)

Note that while all of the core software will be open source we may make use of selected paid services that are not open-source. This is particularly likely for storage (i.e. Amazon S3 or Google Storage). This is based on capabilities, reach, and cost economies of these services. 
Resources Required
We expect that based on the number of packages and volume of commits to the current instantiation of R-Forge that two build servers per platform (Linux, Solaris, OS X, and Windows) would be more than enough to keep up. We'd also need a front-end server for the web interface and REST API. Since these servers will need to be continually available we think the cheapest way to provision them would be by purchasing dedicated hardware and running them out of a co-location facility. 

Storage for build artifacts would use Amazon S3 or Google Storage, and as a result would be extremely inexpensive. Note that the design of the system will be orthogonal to storage back-ends, so it would be possible to use a local open-source storage platform in preference to Amazon or Google's offerings should that prove desirable.
