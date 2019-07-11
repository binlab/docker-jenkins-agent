FROM openjdk:8-jdk-alpine

LABEL maintainer="Mark <mark.binlab@gmail.com>"

ARG AGENT_VER=3.29
ARG KUBECTL_VER=1.15.0
ARG HELM_VER=2.14.1

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
    && curl --create-dirs \
         -sSLo /usr/share/jenkins/slave.jar \
         https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting/${AGENT_VER}/remoting-${AGENT_VER}.jar \
    && chmod 755 /usr/share/jenkins \
    && chmod 644 /usr/share/jenkins/slave.jar \
    && curl -sSL https://storage.googleapis.com/kubernetes-release/release/v${KUBECTL_VER}/bin/linux/amd64/kubectl -o /usr/local/bin/kubectl \
    && chmod 755 /usr/local/bin/kubectl \
    && curl -sSL https://get.helm.sh/helm-v${HELM_VER}-linux-amd64.tar.gz -o /tmp/helm-linux-amd64.tar.gz \
    && tar -xzf /tmp/helm-linux-amd64.tar.gz  -C /tmp \
    && mv /tmp/linux-amd64/helm  /usr/local/bin/helm \
    && rm -rf /tmp/linux-amd64 \
    && mkdir ${HOME}/.jenkins \
    && mkdir -p ${AGENT_WORKDIR} \
    && chown -R ${USER}:${GROUP} ${HOME} \
    && chmod +x /usr/local/bin/jenkins-agent \
    && apk del curl

VOLUME ${HOME}/.jenkins
VOLUME ${AGENT_WORKDIR}

WORKDIR ${HOME}

USER ${USER}

ENTRYPOINT ["jenkins-agent"]
