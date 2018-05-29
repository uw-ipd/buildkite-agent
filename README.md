# Host

A standard docker installation is required to run the buildkite agent. A daemonized agent is launched via [`up -d --build`](./up) and stopped via [`down`](./down). 

## Build Steps

The agent runs within a minimal, `alpine`-based docker image and is likely not a suitable environment for any meaningful build steps. The agent has access to the host docker engine, and may use this engine to execute build steps within custom containers via the buildkite [docker plugin](https://github.com/uw-ipd/docker-buildkite-plugin). 

## Private Repositories

The buildkite agent loads an ssh private key under [`buildkite-secrets`](./buildkite-secrets). To enable support for private repos add the matching [`id_rsa.pub`](./buildkite-secrets/id_rsa.pub) as a repository read-only [deploy key](https://developer.github.com/v3/guides/managing-deploy-keys/) to enable access to the repo.

## CUDA

Executing cuda within a container requires installing [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) matching the current docker installation version. See the nvidia-docker quickstart for installation instructions. The current digs deployment, using docker 1.13.1, requires:

```
sudo apt-get install -y nvidia-docker2=2.0.3+docker1.13.1-1 nvidia-container-runtime=2.0.0+docker1.13.1-1
```

Build steps requiring CUDA *must* utilize a custom docker image and the `nvidia` docker runtime. See [buildkite-cuda](https://github.com/uw-ipd/buildkite-cuda) for a minimal functional example.
