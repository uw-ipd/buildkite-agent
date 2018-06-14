# Host

A standard docker installation is required to run the buildkite agent.
A daemonized agent is launched via [`up -d --build`](./up) and stopped via
[`down`](./down). 

## Build Steps

The agent runs within a minimal, `alpine`-based docker image and is likely not
a suitable environment for any meaningful build steps. The agent has access to
the host docker engine, and may use this engine to execute build steps within
custom containers via the buildkite [docker plugin](https://github.com/uw-ipd/docker-buildkite-plugin). 

## Private Repositories

The buildkite agent can use github app credentials to manage access to
private repositories via
[`git-credential-github-app-auth`](https://github.com/uw-ipd/git-credential-github-app-auth).
To enable private access, create a private, organization-scoped github
application with "Repository contents" permission, copy an application
private key to `buildkite-secrets/buildkite-agent.private-key.pem` and set
the application id in `buildkite-secrets/buildkite-agent.id`.

## CUDA

Executing cuda within a container requires installing
[nvidia-docker](https://github.com/NVIDIA/nvidia-docker) matching the current
docker installation version. See the nvidia-docker quickstart for installation
instructions. The current digs deployment, using docker 1.13.1, requires:

```
sudo apt-get install -y nvidia-docker2=2.0.3+docker1.13.1-1 nvidia-container-runtime=2.0.0+docker1.13.1-1
```

Build steps requiring CUDA *must* utilize a custom docker image and the
`nvidia` docker runtime. See [buildkite-cuda](https://github.com/uw-ipd/buildkite-cuda)
for a minimal functional example. Build steps *should* respect the
`CUDA_VISIBLE_DEVICES` and `CUDA_DEVICE_ORDER` environment variables specified
in the agent environment.
