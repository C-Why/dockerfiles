# This image is used to create bleeding edge docker image and is not compatible with any other image
FROM ubuntu:18.04

ENV DEBIAN_FRONTEND=noninteractive

COPY bionic.sources.list /etc/apt/sources.list
# need ca-certificates
RUN apt-get update && apt-get install -y ca-certificates

RUN apt-get update -y && \
    apt-get upgrade -y && \
    apt-get install -y wget apt-transport-https vim nano tzdata git curl && \
    rm -rf /var/lib/apt/lists/*

ARG DOCKER_MACHINE_VERSION=${DOCKER_MACHINE_VERSION:-0.16.2}
ARG DUMB_INIT_VERSION=${DUMB_INIT_VERSION:-1.0.2}
ARG GIT_LFS_VERSION=${GIT_LFS_VERSION:-2.7.1}


# set gitlab-runner
COPY resource/gitlab-runner-linux-arm64 /usr/bin/gitlab-runner
RUN chmod +x /usr/bin/gitlab-runner  && \
    ln -s /usr/bin/gitlab-runner /usr/bin/gitlab-ci-multi-runner && \
    gitlab-runner --version && \
    useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
    
RUN mkdir -p /etc/gitlab-runner/certs && \
    chmod -R 700 /etc/gitlab-runner
    
COPY resource/docker-machine-Linux-aarch64 /usr/bin/docker-machine
RUN chmod +x /usr/bin/docker-machine && \
    docker-machine --version

COPY resource/dumb-init_1.2.2_aarch64 /usr/bin/dumb-init
RUN chmod +x /usr/bin/dumb-init && \
    dumb-init --version
    
COPY resource/git-lfs-linux-arm64-v2.7.1.tar.gz /tmp/git-lfs.tar.gz
RUN mkdir /tmp/git-lfs && \
    tar -xzf /tmp/git-lfs.tar.gz -C /tmp/git-lfs/ && \
    mv /tmp/git-lfs/git-lfs /usr/bin/git-lfs && \
    rm -rf /tmp/git-lfs* && \
    git-lfs install --skip-repo && \
    git-lfs version 
# 最后添加对应的checksums表格
COPY resource/checksums /tmp/
RUN sha256sum --check --strict /tmp/checksums

COPY entrypoint /
RUN chmod +x /entrypoint

STOPSIGNAL SIGQUIT
# Preserve runner's data
VOLUME ["/etc/gitlab-runner", "/home/gitlab-runner"]
ENTRYPOINT ["/usr/bin/dumb-init", "/entrypoint"]
# init sets up the environment and launches gitlab-runner
CMD ["run", "--user=gitlab-runner", "--working-directory=/home/gitlab-runner"]


