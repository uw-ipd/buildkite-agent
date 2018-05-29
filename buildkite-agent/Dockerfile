FROM alpine:3.7

RUN apk add --no-cache \
      bash \
      git \
      perl \
      rsync \
      openssh-client \
      curl \
      docker \
      jq \
      su-exec \
      py-pip \
      libc6-compat \
      run-parts \
      tini \
      tzdata \
    && \
    pip install --upgrade pip && \
    pip install docker-compose

ENV BUILDKITE_BUILD_PATH=/buildkite/builds \
    BUILDKITE_HOOKS_PATH=/buildkite/hooks \
    BUILDKITE_PLUGINS_PATH=/buildkite/plugins

RUN mkdir -p /buildkite/builds /buildkite/hooks /buildkite/plugins /buildkite/bin \
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

### Install hooks
ADD buildkite-hooks /buildkite/hooks

VOLUME /buildkite
ENTRYPOINT ["buildkite-agent-entrypoint"]
CMD ["start"]
