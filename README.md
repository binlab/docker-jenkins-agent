# Jenkins Agent based on Alpine Linux

_Here different versions of `Jenkins Agent` images for different tasks_

##  JNLP Agent + with Docker client and AWS CLI

* [`3.27-jnlp`, `jnlp`, `latest` \(jnlp/Dockerfile\)](https://github.com/binlab/docker-jenkins-agent/blob/master/jnlp/Dockerfile)

* [`3.27-jnlp-docker`, `jnlp-docker` \(jnlp/docker/Dockerfile\)](https://github.com/binlab/docker-jenkins-agent/blob/master/jnlp/docker/Dockerfile)

* [`3.27-jnlp-docker-awscli`, `jnlp-docker-awscli` \(jnlp/docker-awscli/Dockerfile\)](https://github.com/binlab/docker-jenkins-agent/blob/master/jnlp/docker-awscli/Dockerfile)

    This is an image for using `JNLP` to establish a connection behind `NAT` and with built `Docker` client and + `AWS CLI`. Additional for user `jenkins` added group `docker` with `GID` `994` for communication with host Docker by socket `/var/run/docker.sock` without __`root`__ permissions.

    Running by Docker:
    
    ```shell
    docker run binlab/jenkins-agent:jnlp \
        -url "https://jenkins.example.com" \
        -workDir=/home/jenkins/agent \
        <secret> <agent name>
    ```

    Running by Docker Compose:

    ```yaml
    version: '3.3'
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
