# Jenkins Agent based on Alpine Linux

#### Current versions included in a latest builds:

| Tool              | Version               |
|-------------------|-----------------------|
| Java JDK          | `8u191-jdk-alpine3.8` |
| Jenkins Agent     | `3.29`                |
| Docker CLI        | `18.09.1`             |
| AWS CLI           | `1.16.197`            |
| Kubectl           | `1.15.0`              |
| Helm              | `2.14.1`              |
| Kubernetes-deploy | `0.26.7`                  |
| Kapp              | `0.9.0`               |

_Here different versions of `Jenkins Agent` images for different tasks_

##  JNLP Agent + with Docker client and AWS CLI

* [`3.29-jnlp`, `jnlp`, `latest` \(jnlp/Dockerfile\)](https://github.com/binlab/docker-jenkins-agent/blob/master/jnlp/Dockerfile)

* [`3.29-jnlp-docker`, `jnlp-docker` \(jnlp/docker/Dockerfile\)](https://github.com/binlab/docker-jenkins-agent/blob/master/jnlp/docker/Dockerfile)

* [`3.29-jnlp-docker-awscli`, `jnlp-docker-awscli` \(jnlp/docker-awscli/Dockerfile\)](https://github.com/binlab/docker-jenkins-agent/blob/master/jnlp/docker-awscli/Dockerfile)

    This is an image for using `JNLP` to establish a connection behind `NAT` and with built `Docker` client and + `AWS CLI`. Additional for user `jenkins` added group `docker` with `GID` `994` for communication with host Docker by socket `/var/run/docker.sock` without __`root`__ permissions.

* [`3.29-jnlp-docker-awscli-kubectl-helm`, `jnlp-docker-awscli-kubectl-helm` \(jnlp/docker-awscli-kubectl-helm/Dockerfile\)](https://github.com/binlab/docker-jenkins-agent/blob/master/jnlp/docker-awscli-kubectl-helm/Dockerfile)

    This image is similar to the `jnlp-docker-awscli` image but has Kubernetes specific tools `kubectl` and `helm`

* [`3.29-jnlp-docker-awscli-kubectl-helm-kubernetes-deploy-kapp`, `jnlp-docker-awscli-kubectl-helm-kubernetes-deploy-kapp` \(jnlp/docker-awscli-kubectl-helm/kubernetes-deploy-kapp/Dockerfile\)](https://github.com/binlab/docker-jenkins-agent/blob/master/jnlp/docker-awscli-kubectl-helm-kubernetes-deploy-kapp/Dockerfile)

    This image is similar to the `jnlp-docker-awscli-kubectl-helm` image but has additional Kubernetes  tools `kubernetes-deploy` and `kapp`


    Running by Docker:
    
    ```shell
    docker run binlab/jenkins-agent:jnlp \
        -url "https://jenkins.example.com" \
        -workDir=/home/jenkins/agent \
        <secret> <agent name>
    ```

    Running by Docker Compose:

    ```yaml
    version: "3.3"
    services:
      jenkins-agent:
        image: binlab/jenkins-agent:jnlp-docker
        container_name: jenkins-agent
        hostname: jenkins-agent
        restart: unless-stopped
        environment:
          JENKINS_URL: "https://jenkins.example.com"
          JENKINS_AGENT_WORKDIR: "/home/jenkins/agent"
          JENKINS_SECRET: "jenkins-agent-secret" 
          JENKINS_AGENT_NAME: "project-agent" 
        volumes:
          - jenkins-agent:/home/jenkins/agent:rw
          - /var/run/docker.sock:/var/run/docker.sock:rw
        networks:
          - jenkins-agent
        logging:
          driver: "json-file"
          options:
            max-size: "10m"
            max-file: "3"

    volumes:
      jenkins-agent:
        driver: local

    networks:
      jenkins-agent:
        driver: bridge
    ```

    Optional environment variables:

    * `JENKINS_URL`: url for the Jenkins server, can be used as a replacement to `-url` option, or to set alternate jenkins URL
    * `JENKINS_TUNNEL`: (`HOST:PORT`) connect to this agent host and port instead of Jenkins server, assuming this one do route TCP traffic to Jenkins master. Useful when when Jenkins runs behind a load balancer, reverse proxy, etc.
    * `JENKINS_SECRET`: agent secret, if not set as an argument
    * `JENKINS_AGENT_NAME`: agent name, if not set as an argument
    * `JENKINS_AGENT_WORKDIR`: agent work directory, if not set by optional parameter `-workDir`

## Configuration specifics

### Enabled JNLP protocols

By default, the [JNLP3-connect](https://github.com/jenkinsci/remoting/blob/master/docs/protocols.md#jnlp3-connect) is disabled due to the known stability and scalability issues.
You can enable this protocol on your own risk using the 
`JNLP_PROTOCOL_OPTS=-Dorg.jenkinsci.remoting.engine.JnlpProtocol3.disabled=false` property (the protocol should be enabled on the master side as well).

In Jenkins versions starting from `2.27` there is a [JNLP4-connect](https://github.com/jenkinsci/remoting/blob/master/docs/protocols.md#jnlp4-connect) protocol. 
If you use Jenkins `2.32.x LTS`, it is recommended to enable the protocol on your instance.

### Amazon ECS

Make sure your ECS container agent is [updated](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-agent-update.html) before running. Older versions do not properly handle the entryPoint parameter. See the [entryPoint](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definitions) definition for more information.
