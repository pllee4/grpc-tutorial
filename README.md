# grpc-tutorial

![Build](https://github.com/pllee4/gRPC-tutorial/actions/workflows/Build.yml/badge.svg)

## Dependencies

- make sure gRPC is installed
- the installation part can be referred from [here](https://grpc.io/docs/languages/cpp/quickstart/)
- for the docker building of ci for this repo, kindly refer [here](https://github.com/pllee4/docker/blob/master/ubuntu-ci/Dockerfile.20.04-gRPC)

```
$ mkdir build
$ cd build
$ cmake -DCMAKE_PREFIX_PATH=<YOUR_GRPC_INSTALLATION_PATH> ..
$ make -j
```
