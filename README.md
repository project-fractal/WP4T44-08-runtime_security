# WP4T44-08-runtime_security

Process isolation through containerization and user control

## Getting Started

This document is a set of guidelines and good practices for container application development. Developers working on Fractal components and specifically those developing any kind of containerized technology are encouraged to follow this set of practices to ensure the security in their container images, instances, networks and applications.


### Container image security practices

* The first recommendation on container image development is probably the most obvious, always use the official base images for your developments, if available. Always make sure that the images you are starting your developments from are secure and come from the official repositories.

* Secondly, make sure that your images are as lightweight as possible. This is not only an advantage in terms of disk usage, but also helps keeping your images free of software inconsistencies or incompatibilities. Actions like apt-get upgrade or installing unrequired packages (“just in case” packages) should be avoided, as they increase the image size and make your images unnecessarily complex.

* Running images as the root user: This is a highly risky practice and unfortunately a widely spreaded one. When developing an image from a base OS (e.g. ubuntu:latest), the user running the processes inside that container will be the root user by default. This means that, if that container is given permissions on its host, attackers can gain access to the host machine by mounting specific directories from the root user in the container. Try to avoid this practice by always running containers as a non-root user, or by creating a new user during image creation (in the Dockerfile).

* Secure files, passwords and keys: There are two common practices in image creation developments that are highly risky; The use of ADD command instead of COPY, and the inclusion of arguments and environment variables which contain plain-text passwords or keys. When including your passwords into the image, always try to do it from non plain text files (hashed, or encrypted files), instead of ARG and ENV variables which will be later be potentially read by attackers. For secure file storage into the container, use the COPY command instead of ADD, because the ADD command automatically extracts the content of your files, which can lead to the execution of malicious software inside your container.

* Once you are done creating your image, perform an image scanning. Scanning your container images is crutial to detect potential vulnerabilities. Docker Engine has its own scanning tool (‘docker scan’) but there are other open source alternatives, like Clair (https://github.com/arminc/clair-scanner) and Anchore (https://anchore.com/container-vulnerability-scanning/).

* Keep updated about new vulnerabilities, use other open source tools to scan your network security (Cilium), and don’t overlook your orchestrator’s security. While container orchestrators like Kubernetes provide their own security features, they don’t tackle the container’s inside security aspects.

### Container application security

Once your container images are secure enough, it is time to deploy your containerized applicaition into your hosting environment. Each deployment architecture is potentially different, and this makes it very complex to give a set of rules which apply for every one of them. However, some security common practices can be stablished when deploying your containerized application:

* Do not expose port 22 or SSH on containers. Containers should not be accessed from the outside except for debugging purposes, so enabling SSH communciations in your containers or exposing port 22 is an unnecesary risky practice. Also make sure that only those ports necessary for the functioning of your application are exposed.

* Secure your Container Registries (repositories).

* Perform Role-Based Access Control for the users in your application to avoid unathorized accesses.

* Use TLS to secure communications between services, users, and containers.

* Recycle your containers, they are thought to be ephemeral, and restoring them to your fresh-image state makes them secure again. Long-time running containers are more likely to be infected with malicious software.


## Examples

### Secure Dockerfiles

```
# Use arguments for better image version updating
ARG PYTHON_VERSION=3.9.13
FROM python:${PYTHON_VERSION}

# Create a new non-root user
RUN useradd -ms /bin/bash fractal-user

# Copy files and give permissions to non-priviledged user
COPY --chown=fractal-user:fractal-user my_file.txt /home/fractal-user/my_file.txt

# Change the user and start working as the new non-priviledged user
USER fractal-user
WORKDIR /home/fractal-user

ENTRYPOINT python3
```


### Unsecure Dockerfiles
```
# Don't start from a latest base image
# latest images can be different from one build to another
FROM ubuntu:latest

# Avoid installing non-required final packages, upgrade base-image packages
RUN apt-get update && apt-get upgrade
RUN apt-get install vim ip-tools

# Use ADD to include your files into the container
ADD files /home/root/myfiles

# Try not to COPY full diectories, specially if they include the Dockerfile
COPY . .

# Expose unrequired ports
EXPOSE 22
EXPOSE 9000

# Do not use ENV variables to pass secrets or keys
ENV API_KEY='asdlLJ83dJK1U4m2'
ENV pass='P4$$w0Rd'

ENTRYPOINT bash
```
