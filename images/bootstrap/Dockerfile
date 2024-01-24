# Copyright 2017 The Kubernetes Authors.
# Copyright 2021 The KubeVirt Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Includes basic workspace setup, with gcloud and a bootstrap runner
FROM debian:bullseye

WORKDIR /workspace
RUN mkdir -p /workspace
ENV WORKSPACE=/workspace \
    TERM=xterm

# add env we can debug with the image name:tag
ARG IMAGE_ARG
ENV IMAGE=${IMAGE_ARG}

# Install packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    curl \
    vim  \
    cpio \
    findutils \
    gettext \
    git \
    iproute2 \
    openjdk-11-jdk \
    jq \
    qemu-system \ 
    libvirt-clients \
    libvirt-daemon-system \
    make \
    mercurial \
    patch \
    procps \
    protobuf-compiler \
    python3 \
    rsync \
    skopeo \
    sudo \
    buildah \
    qemu-user-static \
    bind9-utils \
    wget \
    zip \
    unzip \
    python3-jinja2 &&\
  rm -rf /var/lib/apt/lists/*

#
# BEGIN: PODMAN IN CONTAINER SETUP
#

# no need to reinstall the shadow-utils as the debian do not have any issue with podman
# no need to exclude selinux as debain do not use selinux

RUN apt-get -y update && \
    apt-get --no-install-recommends -y install crun podman && \
    rm -rf /var/lib/apt/lists/* && \
    ln -s /usr/bin/podman /usr/bin/docker

VOLUME /var/lib/containers

COPY storage.conf /etc/containers/storage.conf

RUN mkdir -p /var/lib/shared-images/overlay-images /var/lib/shared-images/overlay-layers; touch /var/lib/shared-images/overlay-images/images.lock; touch /var/lib/shared-images/overlay-layers/layers.lock

COPY containers.conf /etc/containers/containers.conf
COPY podman.json /etc/containers/networks/podman.json

RUN chmod 644 /etc/containers/containers.conf

ENV _CONTAINERS_USERNS_CONFIGURED=""

#
# END: PODMAN IN CONTAINER SETUP
#

ENV VERSION=5.4.1
RUN mkdir bazel-${VERSION}
WORKDIR /workspace/bazel-${VERSION}
RUN wget https://github.com/bazelbuild/bazel/releases/download/${VERSION}/bazel-${VERSION}-dist.zip
RUN unzip bazel-${VERSION}-dist.zip
RUN ./compile.sh
RUN mv ./output/bazel /usr/local/bin/
WORKDIR /workspace
RUN rm -rf ./bazel-${VERSION}

# create mixin directories
RUN mkdir -p /etc/setup.mixin.d/ && mkdir -p /etc/teardown.mixin.d/

# Trust git repositories used for e2e jobs
RUN git config --global --add safe.directory '*'

# note the runner is also responsible for making podman in container function if
# env PODMAN_IN_CONTAINER_ENABLED is set and similarly responsible for generating

# .bazelrc files if bazel remote caching is enabled
COPY ["entrypoint.sh", "runner.sh", "create_bazel_cache_rcs.sh", \
        "/usr/local/bin/"]

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
