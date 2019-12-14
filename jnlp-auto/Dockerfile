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
ENV DOWNLOAD_DIR=/usr/share/jenkins

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
     dumb-init \
     && mkdir -p ${HOME}/.jenkins \
     && mkdir -p ${DOWNLOAD_DIR} \
     && mkdir -p ${AGENT_WORKDIR} \
     && chmod 755 ${AGENT_WORKDIR} \
     && chown -R ${USER}:${GROUP} ${HOME} \
     && chown -R ${USER}:${GROUP} ${DOWNLOAD_DIR} \
     && chown -R ${USER}:${GROUP} ${AGENT_WORKDIR} \
     && chmod +x /usr/local/bin/jenkins-agent \
     && printf "\n### Cleaning up ###\n\n"

VOLUME ${HOME}/.jenkins
VOLUME ${AGENT_WORKDIR}

WORKDIR ${HOME}

USER ${USER}

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["jenkins-agent"]