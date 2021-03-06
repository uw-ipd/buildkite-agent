FROM ubuntu:16.04

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

### Install buildkite agent deps
RUN apt-get update && apt-get install -y --no-install-recommends \
      curl \
      ca-certificates \
      bash \
      git \
      perl \
      rsync \
      openssh-client \
      curl \
      jq \
      iptables \ 
      libltdl7 \
    && rm -rf /var/lib/apt/lists/*

RUN curl -Lfs -o /sbin/tini  https://github.com/krallin/tini/releases/download/v0.18.0/tini \
    && chmod +x /sbin/tini

RUN curl -O https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/docker-ce_18.06.1~ce~3-0~ubuntu_amd64.deb && \
    dpkg -i docker-ce_18.06.1~ce~3-0~ubuntu_amd64.deb && \
    rm docker-ce_18.06.1~ce~3-0~ubuntu_amd64.deb

RUN curl -Lfs https://github.com/docker/compose/releases/download/1.21.0/docker-compose-Linux-x86_64 -o /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/docker-compose

### Install admin tools
RUN apt-get update && apt-get install -y --no-install-recommends vim-tiny

### Install conda
ENV PATH /opt/conda/bin:$PATH

RUN apt-get update --fix-missing && \
    apt-get install -y wget bzip2 ca-certificates curl git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Updated to miniconda 4.7.10
RUN wget --quiet https://repo.continuum.io/miniconda/Miniconda3-4.7.10-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh && \
    /opt/conda/bin/conda clean -tipsy && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate base" >> ~/.bashrc

# Install buildkite agent

ENV BUILDKITE_BUILD_PATH=/buildkite/builds \
    BUILDKITE_HOOKS_PATH=/buildkite-hooks \
    BUILDKITE_PLUGINS_PATH=/buildkite-plugins

RUN mkdir -p /buildkite/builds /buildkite-hooks /buildkite-plugins /buildkite/bin \
    && curl -Lfs -o /usr/local/bin/ssh-env-config.sh https://raw.githubusercontent.com/buildkite/docker-ssh-env-config/master/ssh-env-config.sh \
    && chmod +x /usr/local/bin/ssh-env-config.sh

### Install binary from github release
# From buildkite agent source
#COPY ./buildkite-agent /usr/local/bin/buildkite-agent

# Fetch/install from latest
#RUN curl -s https://api.github.com/repos/buildkite/agent/releases/latest | \
#    grep browser_download_url | grep buildkite-agent-linux-amd64 | sed 's/"browser_download_url"://' | \
#    xargs curl -L | tar -xz -C /buildkite/bin ./buildkite-agent
RUN curl -L https://github.com/buildkite/agent/releases/download/v3.2.0/buildkite-agent-linux-amd64-3.2.0.tar.gz | \
    tar -xz -C /usr/local/bin ./buildkite-agent && \
    chmod +x /usr/local/bin/buildkite-agent && \
    cp /usr/local/bin/buildkite-agent /buildkite/bin

### Install entrypoint from github release
RUN curl -Lfs -o /usr/local/bin/buildkite-agent-entrypoint \
      https://raw.githubusercontent.com/buildkite/agent/v3.2.0/packaging/docker/alpine-linux/entrypoint.sh && \
    chmod +x /usr/local/bin/buildkite-agent-entrypoint

# Disable strict host key checking and password auth for ssh
RUN echo 'StrictHostKeyChecking no' >> /etc/ssh/ssh_config && \
    echo 'PasswordAuthentication no' >> /etc/ssh/ssh_config

# Install git_credential_github_app_auth, for access to private repos over https
RUN conda install pyjwt click attrs && \
    pip install git+https://github.com/uw-ipd/git-credential-github-app-auth@v0.2#egg=git_credential_github_app_auth && \
    git config --global credential.helper github-app-auth && \
    git config --global credential.UseHttpPath true

# Install conda-based curl, allowing use of '-H @file' syntax
RUN conda install curl


VOLUME /buildkite
ENTRYPOINT ["buildkite-agent-entrypoint"]
CMD ["start"]
