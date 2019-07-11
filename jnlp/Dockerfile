FROM openjdk:8u191-jdk-alpine3.8

LABEL maintainer="Mark <mark.binlab@gmail.com>"

ARG AGENT_VER=3.29

ARG USER=jenkins
ARG GROUP=jenkins
ARG UID=1024
ARG GID=1024

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
    && printf "\n### Cleaning up ###\n\n" \
    && apk del --purge -v \
         curl

VOLUME ${HOME}/.jenkins
VOLUME ${AGENT_WORKDIR}

WORKDIR ${HOME}

USER ${USER}

ENTRYPOINT ["jenkins-agent"]