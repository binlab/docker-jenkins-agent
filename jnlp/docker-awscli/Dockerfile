FROM openjdk:8u191-jdk-alpine3.8

LABEL maintainer="Mark <mark.binlab@gmail.com>"

ARG AGENT_VER=3.29
ARG DOCKER_VER=18.09.1
ARG AWSCLI_VER=1.16.197

ARG USER=jenkins
ARG GROUP=jenkins
ARG UID=1024
ARG GID=1024

ARG DOCKER_GROUP=docker
ARG DOCKER_GID=994

ARG AGENT_WORKDIR=/home/${USER}/agent
ENV HOME /home/${USER}
ENV AGENT_WORKDIR=${AGENT_WORKDIR}

COPY jenkins-agent /usr/local/bin/jenkins-agent

RUN addgroup -g ${GID} ${GROUP} \
    && adduser -D -h ${HOME} -g "${USER} service" \
         -u ${UID} -G ${GROUP} ${USER} \
    && set -x \
    && apk add --update --no-cache \
         curl \
         bash \
         git \
         openssh-client \
         openssl \
         procps \
         shadow \
    && printf "\n### Installing Jenkins agent v${AGENT_VER} ###\n\n" \
    && curl --create-dirs \
         -sSLo /usr/share/jenkins/slave.jar \
         https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting/${AGENT_VER}/remoting-${AGENT_VER}.jar \
    && chmod 755 /usr/share/jenkins \
    && chmod 644 /usr/share/jenkins/slave.jar \
    && mkdir ${HOME}/.jenkins \
    && mkdir -p ${AGENT_WORKDIR} \
    && chown -R ${USER}:${GROUP} ${HOME} \
    && chmod +x /usr/local/bin/jenkins-agent \
    && printf "\n### Installing AWS CLI v${AWSCLI_VER} ###\n\n" \
    && apk add --update --no-cache \
         python \
         py-pip \
    && pip install awscli==${AWSCLI_VER} \
    && printf "\n### Installing Docker CLI v${DOCKER_VER} ###\n\n" \
    && curl -fsSL https://download.docker.com/linux/static/stable/x86_64/docker-${DOCKER_VER}.tgz \
         | tar xzv \
             --strip 1 \
             -C /usr/local/bin \
             docker/docker \
    && chmod +x /usr/local/bin/docker \
    && addgroup -S -g ${DOCKER_GID} ${DOCKER_GROUP} \
    && usermod -a -G ${DOCKER_GROUP} ${USER} \
    && printf "\n### Cleaning up ###\n\n" \
    && apk del --purge -v \
         shadow \
         py-pip

VOLUME ${HOME}/.jenkins
VOLUME ${AGENT_WORKDIR}

WORKDIR ${HOME}

USER ${USER}

RUN printf "\n### Testing versions, permissions and dependencies of binaries ###\n\n" \
      && aws --version \
      && echo "Docker CLI: $(docker --version)"

ENTRYPOINT ["jenkins-agent"]