% Jenkins - Open Source Automation Server
% Didier Richard
% 2019/08/29

---

revision:
    - 1.0.0 : 2019/01/06 : Jenkins 2.150.1 (only Docker from Docker)  
    - 1.0.1 : 2019/01/14 : Jenkins 2.150.1 (both Docker from Docker and Docker in Docker)  
    - 2.0.0 : 2019/08/29 : Jenkins 2.176.2 (Dind and Dond approches)  

---

# Why ? #

Test a Jenkins with Docker pipeline containered with Docker server on the host and
client in the container (Dond) and Jenkins with Docker pipeline contained with
a Docker server and client in the same container (Dind).

The idea is the be able to :

* build an image from a Dockerfile within a Jenkins' pipeline ;

```groovy
#!/usr/bin/env groovy
...
pipeline {
...
  stages {
    ...
    stage('Build Docker Image for testing') {
      steps {
        script {
          def proxies = getProxies()
          def buildPath = "./Dockerfiles/CI/"
          def DockerCmdCI = "${DockerCmd} ${proxies}"
          echo "Docker options with proxy setting : ${DockerCmdCI}"
          testImage = docker.build(CIImageFullName,"-f ${buildPath}/Dockerfile ${proxies} ${buildPath}")
        }
      }
    }
    ...
  }/* stages */
}/* pipeline */
```

* to run this image for unit and functional testing of an application still
  within a Jenkins' pipeline ;

```groovy
#!/usr/bin/env groovy
...
pipeline {
...
  stages {
    ...
    stage('Test') {
      steps {
        echo 'Testing...'
        script {
          testImage.withRun("${DockerCmd} ${DockerSpecialOpts}","${CmdToExec} run test --env=dev --envFile=${params.ConfigFile}")
        }
      }
    }
    ...
  }/* stages */
}/* pipeline */
```

* to build an image ready for deployment if all went well !

```groovy
#!/usr/bin/env groovy
...
pipeline {
...
  stages {
    ...
    stage('Build Application') {
      steps {
        echo 'Building...'
        script {
          testImage.withRun("${DockerCmd} ${DockerSpecialOpts}","${CmdToExec} run build --env=prod --envFile=${params.ConfigFile}")
        }
        stash includes: "*.tar.gz", name: "servicepp"
      }
    }
    ...
  }/* stages */
}/* pipeline */
```

* and later, deploy !

# Web links #

[do not use docker in docker for ci](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)
[Docker in Docker](https://github.com/jpetazzo/dind)

_fin du document[^pandoc_gen]_

[^pandoc_gen]: document généré via $ `pandoc -V fontsize=10pt -V geometry:"top=2cm, bottom=2cm, left=1cm, right=1cm" -s -N --toc -o jenkins.pdf README.md`{.bash}
