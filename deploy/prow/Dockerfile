# Copyright 2021 The Kubernetes Authors All rights reserved.
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

# Includes tools used for kubernetes/minikube CI
# NOTE: we attempt to avoid unnecessary tools and image layers while
# supporting kubernetes builds, minikube installation, etc.
FROM debian:bookworm

# arg that specifies the go version to install
ARG GO_VERSION

# add envs:
# - hinting that we are in a docker container
ENV GOPATH=/home/prow/go \
    PATH=/usr/local/go/bin:${PATH} \
    CONTAINER=docker


# Install tools needed to:
# - install docker
#
# TODO: the `sed` is a bit of a hack, look into alternatives.
# Why this exists: `docker service start` on debian runs a `cgroupfs_mount` method,
# We're already inside docker though so we can be sure these are already mounted.
# Trying to remount these makes for a very noisy error block in the beginning of
# the pod logs, so we just comment out the call to it... :shrug:
RUN export ARCH=$(dpkg --print-architecture) \
    && echo "Installing Packages ..." \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        apt-transport-https \
        build-essential \
        ca-certificates \
        curl \
        file \
        git \
        gnupg \
        kmod \
        lsb-release \
        mercurial \
        pkg-config \
        procps \
        python3 \
        python3-dev \
        python3-pip \
        rsync \
        software-properties-common \
        unzip \
    && rm -rf /var/lib/apt/lists/* \
    && echo "Installing Go ..." \
        && export GO_TARBALL="go${GO_VERSION}.linux-${ARCH}.tar.gz"\
        && curl -fsSL "https://storage.googleapis.com/golang/${GO_TARBALL}" --output "${GO_TARBALL}" \
        && tar xzf "${GO_TARBALL}" -C /usr/local \
        && rm "${GO_TARBALL}"\
        && mkdir -p "${GOPATH}/bin" \
    && echo "Installing Docker ..." \
        && install -m 0755 -d /etc/apt/keyrings \
        && curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg \
        && chmod a+r /etc/apt/keyrings/docker.gpg \
        && echo \
            "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
            bookworm stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null \
        && apt-get update \
        && apt-get install -y --no-install-recommends \
            docker-ce \
            docker-ce-cli \
            containerd.io \
            docker-buildx-plugin \
            docker-compose-plugin \
        && rm -rf /var/lib/apt/lists/* \
        && sed -i 's/cgroupfs_mount$/#cgroupfs_mount\n/' /etc/init.d/docker \
    && echo "Ensuring Legacy Iptables ..." \
        && update-alternatives --set iptables /usr/sbin/iptables-legacy \
        && update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy \
    && echo "Installing Kubectl ..." \
        && curl -LO "https://dl.k8s.io/$(curl -sSL https://dl.k8s.io/release/stable.txt)/bin/linux/${ARCH}/kubectl" \
        && chmod +x ./kubectl \
        && cp kubectl /usr/local/bin
# copy in image utility scripts
COPY wrapper.sh /usr/local/bin/
# entrypoint is our wrapper script, in Prow you will need to explicitly re-specify this
ENTRYPOINT ["wrapper.sh", "/bin/bash"]
# volume for docker in docker, use an emptyDir in Prow
VOLUME ["/var/lib/docker"]
