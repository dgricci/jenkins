% Jenkins - Open Source Automation Server
% Didier Richard
% 2019/01/06

---

revision:
    - 1.0.0 : 2019/01/06 : Jenkins 2.150.1  

---

# Building #

```bash
$ docker build -t dgricci/jenkins:lts .
```

## Behind a proxy (e.g. 10.0.4.2:3128) ##

```bash
$ docker build \
    --build-arg http_proxy=http://10.0.4.2:3128/ \
    --build-arg https_proxy=http://10.0.4.2:3128/ \
    -t dgricci/jenkins:lts .
```

## Build command with arguments default values ##

```bash
$ docker build \
    --build-arg DOCKER_ENGINE_VERSION= $(docker version --format '{{.Server.Version}}') \
    --build-arg DOCKER_GROUPID=$(getenv group docker | cut -d: -f1) \
    -t dgricci/jenkins:lts .
```

# Use #

This image based on `jenkins/jenkins:lts` (Jenkins in Docker) adds the
docker's client.

This image needs to be re-built each time the docker version is updated on
the host as it tries to get the same version for the client and the host
docker deamon. It also creates the `docker` group with the same id as the
one on the host in order to allow the client to get access to the
`/var/run/docker.sock` socket without permission denied.


First, one creates a volume for the service :

```bash
$ docker volume create jenkins-vol
```

Second, launch the service (say port `9076` on the host bound to the port
`8080` on the container' side and agents on port `25263` -- usually it is
50000 --). Bind the docker's socket with the container. Mount the volume on
`/var/jenkins_home` as expected by the service and bind your git repositories
with the container (here : `/repos`). The later will allow to give
`file:///repos/a_local_git_repository` as a repository URL in the service :

```bash
$ docker run -d -p 9076:8080 -p 25263:25263 --env JENKINS_SLAVE_AGENT_PORT=25263 -v /var/run/docker.sock:/var/run/docker.sock --mount type=volume,source=jenkins-vol,target=/var/jenkins_home --mount type=bind,source=/data/git/repos,target=/repos,readonly dgricci/jenkins:lts
```

Collect the container'ID the first time (say `containerID`) and collect the
admin password to be given to the service through `http://localhost:9076/`) :

```bash
$ docker logs containerID
```

Then reply to the forms and just install "preferred plugins" !

To remove the service, first halt it then destroy it :

```bash
$ docker stop containerID
$ docker rm containerID
```


_fin du document[^pandoc_gen]_

[^pandoc_gen]: document généré via $ `pandoc -V fontsize=10pt -V geometry:"top=2cm, bottom=2cm, left=1cm, right=1cm" -s -N --toc -o jenkins.pdf README.md`{.bash}
