FROM openjdk:8u191-jdk-alpine3.8

LABEL maintainer="Mark <mark.binlab@gmail.com>"

ARG DOCKER_VER=18.09.1
ARG AWSCLI_VER=1.16.197
ARG KUBECTL_VER=1.15.0
ARG HELM_VER=2.14.1

ARG USER=jenkins
ARG GROUP=jenkins
ARG UID=1024
ARG GID=1024

ARG DOCKER_GROUP=docker
ARG DOCKER_GID=994

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
     dumb-init \
     procps \
     shadow \
     && mkdir ${HOME}/.jenkins \
     && mkdir -p ${AGENT_WORKDIR} \
     && mkdir -p ${DOWNLOAD_DIR} \
     && chmod 755 ${AGENT_WORKDIR} \
     && chown -R ${USER}:${GROUP} ${HOME} \
     && chown -R ${USER}:${GROUP} ${DOWNLOAD_DIR} \
     && chown -R ${USER}:${GROUP} ${AGENT_WORKDIR} \
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
     && printf "\n### Installing Kubectl v${KUBECTL_VER} ###\n\n" \
     && curl -sSL https://storage.googleapis.com/kubernetes-release/release/v${KUBECTL_VER}/bin/linux/amd64/kubectl \
     -o /usr/local/bin/kubectl \
     && chmod +x /usr/local/bin/kubectl \
     && printf "\n### Installing Helm v${HELM_VER} ###\n\n" \
     && curl -fsSL https://get.helm.sh/helm-v${HELM_VER}-linux-amd64.tar.gz \
     | tar xzv \
     --strip 1 \
     -C /usr/local/bin \
     linux-amd64/helm \
     && chmod +x /usr/local/bin/helm \
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
     && echo "Docker CLI: $(docker --version)" \
     && echo "Kubectl: $(kubectl version --client --short)" \
     && echo "Helm: $(helm version --client --short)"

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["jenkins-agent"]