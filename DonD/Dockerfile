# Jenkins in container with docker client
# This image needs to be re-built each time the docker version is updated on
# the host ...
FROM jenkins/jenkins:lts
MAINTAINER Didier Richard <didier.richard@ign.fr>
LABEL       version="1.0.0" \
            jenkins="lts" \
            os="Debian Stretch" \
            description="Jenkins in Docker with the docker client"

# install the same client version as the docker daemon running on the host
# one could use : --build-arg DOCKER_ENGINE_VERSION=$(docker version --format '{{.Server.Version}}')
# then :
# apt-get install docker-ce-cli="^.*${DOCKER_ENGINE_VERSION}.*$"
# but this does seem to work ... workaround with apt-cache !
ARG DOCKER_ENGINE_VERSION
ENV DOCKER_ENGINE_VERSION ${DOCKER_ENGINE_VERSION:-18.09.1}

# set docker group id the same as the host to prevent a "dial unix
# /var/run/docker.sock: connect: permission denied" error. Implies rebuilding
# this image per host ...
# one could use : --build-arg DOCKER_GROUPID=$(getent group docker | cut -d: -f1)
ARG DOCKER_GROUPID
ENV DOCKER_GROUPID ${DOCKER_GROUPID:-128}

USER root
ENV DEBIAN_FRONTEND=noninteractive
RUN \
    apt-get -qy update && \
    apt-get install -qy --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        gnupg \
        curl \
        software-properties-common && \
    curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | apt-key add - && \
    add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
        $(lsb_release -cs) \
        stable" && \
    apt-get -qy update && \
    if [ $(apt-cache madison docker-ce-cli | cut -d\| -f2 | tr -d '[:space:]' | grep -c "${DOCKER_ENGINE_VERSION}") -ne 1 ] ; then \
        echo "docker CLI version does not match ${DOCKER_ENGINE_VERSION}." ; \
    else { \ 
        apt-get install -qy --no-install-recommends \
            docker-ce-cli ; \
        groupadd -r -g ${DOCKER_GROUPID} docker ; \
        usermod -aG docker jenkins ; } ; \
    fi && \
    apt-get -y purge \
        curl && \
    apt-get -y clean && \
    apt-get -y autoremove && \
    rm -fr /var/lib/apt/lists/*

USER jenkins

