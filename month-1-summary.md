
# Summary of the first month

## Deliverables

### 1 Jenkins DockerFile, tailored to r-hub

Available at https://github.com/r-hub/rhub-jenkins and
https://github.com/r-hub/dokku-jenkins. This configuration is not final,
but most challenges related to configuring Jenkins in a Docker container and
running it from Dokku are solved.

### 2 r-hub application, front-end

A minimal web application is at https://github.com/r-hub/rhub-frontend.
It needs a better response page, otherwise it is complete for this stage
of the project. (Will change with other OSes and configuration options.)

### 3 r-hub application, back-end

It is at https://github.com/r-hub/rhub-backend. The principles are
done. The Jenkins job configuration templates are also included in this
repository, we will extend these substantially.

### 4 A running instance of our Jenkins container

This is not done yet, we only have local deployment.

### 5 A running instance of the r-hub front-end

This is not done yet, we only have local deployment.

### 6 A running instance of the r-hub back-end

This is not done yet, we only have local deployment.

### 7 DockerFile for an Ubuntu Linux builder

We have two Linux builder configurations at
https://github.com/r-hub/rhub-linux-builders.

### 8 A running instance of a Linux builder

This is not done yet, we only have local deployment.

### 9 A GitHub repository for r-hub's documentation

It is at https://github.com/r-hub/docs. To be extended as we move on.

### 10 A GitHub repository for user feedback

A currently empty repository is at https://github.com/r-hub/feedback.

### 11 r-hub controller

After considering and evaluating various VM and container management
solutions, I decided to use a simple Vagrant based approach for now.
The controller code and configuration is included in an R package:
https://github.com/r-hub/rhubctrl

### 12 A method for storing sensitive information on GitHub

We use https://github.com/hadley/secure to store secrets in the repository
on the controller, and environment variables to propagate them to the
virtual machines and containers. Somewhat more details of the configuration
is at https://github.com/r-hub/rhubctrl#configuration.
