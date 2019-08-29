# Jenkins in container with docker engine server and client
# Completely inspired from https://github.com/docker-library/docker/blob/master/19.03/Dockerfile
FROM jenkins/jenkins:lts
MAINTAINER Didier Richard <didier.richard@ign.fr>
LABEL       version="1.0.0" \
            jenkins="lts" \
            docker="19.03" \
            os="Debian Stretch" \
            description="Jenkins in Docker with docker server and client in"

# set up nsswitch.conf for Go's "netgo" implementation (which Docker explicitly uses)
# - https://github.com/docker/docker-ce/blob/v17.09.0-ce/components/engine/hack/make.sh#L149
# - https://github.com/golang/go/blob/go1.9.1/src/net/conf.go#L194-L275
# - docker run --rm debian:stretch grep '^hosts:' /etc/nsswitch.conf
RUN [ ! -e /etc/nsswitch.conf ] && echo 'hosts: files dns' > /etc/nsswitch.conf

ENV DOCKER_CHANNEL stable
ENV DOCKER_VERSION 19.03.1

USER root
ENV DEBIAN_FRONTEND=noninteractive
RUN set -eux \
    apt-get -qy update ; \
    apt-get install -qy --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        gnupg \
        curl \
        software-properties-common ; \
    curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | apt-key add - ; \
    add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) ${DOCKER_CHANNEL}" ; \
    apt-get -qy update ; \
    if [ $(apt-cache madison docker-ce | cut -d\| -f2 | tr -d '[:space:]' | grep -c "${DOCKER_VERSION}") -ne 1 ] ; then \
        echo >&2 "${pkg} version does not match ${DOCKER_VERSION}." ; \
        exit 1; \
    fi ; \
    apt-get install -qy --no-install-recommends \
             docker-ce docker-ce-cli ; \
    dockerd --version ; \
    docker version ; \
    usermod -aG docker jenkins ; \
    apt-get -y purge \
        curl ; \
    apt-get -y clean ; \
    apt-get -y autoremove ; \
    rm -fr /var/lib/apt/lists/*

# https://github.com/docker-library/docker/pull/166
#   dockerd-entrypoint.sh uses DOCKER_TLS_CERTDIR for auto-generating TLS certificates
#   docker-entrypoint.sh uses DOCKER_TLS_CERTDIR for auto-setting DOCKER_TLS_VERIFY and DOCKER_CERT_PATH
# (For this to work, at least the "client" subdirectory of this path needs to be shared between the client and server containers via a volume, "docker cp", or other means of data sharing.)
ENV DOCKER_TLS_CERTDIR=/certs
# also, ensure the directory pre-exists and has wide enough permissions for "dockerd-entrypoint.sh" to create subdirectories, even when run in "rootless" mode
RUN mkdir /certs /certs/client && chmod 1777 /certs /certs/client
# (doing both /certs and /certs/client so that if Docker does a "copy-up" into a volume defined on /certs/client, it will "do the right thing" by default in a way that still works for rootless users)

USER jenkins

#add jenkins plugins (See https://plugins.jenkins.io/) :
COPY plugins.txt /usr/share/jenkins/plugins.txt
RUN /usr/local/bin/plugins.sh /usr/share/jenkins/plugins.txt

