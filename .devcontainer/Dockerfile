FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu-20.04

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Node.js major-version to be installed.
ARG NODE_VERSION=12

# Go version
ENV GOLANG_VERSION 1.14.6
ENV GOLANG_DOWNLOAD_URL https://golang.org/dl/go$GOLANG_VERSION.linux-amd64.tar.gz
ENV GOLANG_DOWNLOAD_SHA256 5c566ddc2e0bcfc25c26a5dc44a440fcc0177f7350c1f01952b34d5989a0d287

ARG USERNAME=vscode
ARG DOCKER_GID=994

# Docker Compose version
ARG COMPOSE_VERSION=1.26.2

ENV ETCD_VER=v3.4.10

ENV GITHUB_URL=https://github.com/etcd-io/etcd/releases/download

ENV GOPATH /go

ENV MAGE_VERSION=1.10.0
ENV GOTESTSUM_VERSION=0.5.2

RUN apt-get update \
  && apt-get -y install curl telnet jq git build-essential \
  # Install Node and YARN
  && curl -sL https://deb.nodesource.com/setup_${NODE_VERSION}.x | bash - \
  && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
  && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
  && apt update \
  && apt install -y --no-install-recommends nodejs yarn \
  # Install Go
  && curl -fsSL "$GOLANG_DOWNLOAD_URL" -o golang.tar.gz \
  && echo "$GOLANG_DOWNLOAD_SHA256  golang.tar.gz" | sha256sum -c - \
  && tar -C /usr/local -xzf golang.tar.gz  \
  && ln -s /usr/local/go/bin/go /usr/local/bin \
  && rm golang.tar.gz \
  # Build Go tools w/module support
  && mkdir -p /tmp/gotools \
  && cd /tmp/gotools \
  && GOPATH=/tmp/gotools GO111MODULE=on go get -v golang.org/x/tools/gopls@latest 2>&1 \
  && GOPATH=/tmp/gotools GO111MODULE=on go get -v \
  honnef.co/go/tools/...@latest \
  golang.org/x/tools/cmd/gorename@latest \
  golang.org/x/tools/cmd/goimports@latest \
  golang.org/x/tools/cmd/guru@latest \
  golang.org/x/lint/golint@latest \
  github.com/mdempsky/gocode@latest \
  github.com/cweill/gotests/...@latest \
  github.com/haya14busa/goplay/cmd/goplay@latest \
  github.com/sqs/goreturns@latest \
  github.com/josharian/impl@latest \
  github.com/davidrjenni/reftools/cmd/fillstruct@latest \
  github.com/uudashr/gopkgs/v2/cmd/gopkgs@latest  \
  github.com/ramya-rao-a/go-outline@latest  \
  github.com/acroca/go-symbols@latest  \
  github.com/godoctor/godoctor@latest  \
  github.com/rogpeppe/godef@latest  \
  github.com/zmb3/gogetdoc@latest \
  github.com/fatih/gomodifytags@latest  \
  github.com/mgechev/revive@latest  \
  github.com/go-delve/delve/cmd/dlv@latest 2>&1 \
  # Build Go tools w/o module support
  && GOPATH=/tmp/gotools go get -v github.com/alecthomas/gometalinter 2>&1 \
  # Build gocode-gomod
  && GOPATH=/tmp/gotools go get -x -d github.com/stamblerre/gocode 2>&1 \
  && GOPATH=/tmp/gotools go build -o gocode-gomod github.com/stamblerre/gocode \
  # Install Go tools
  && mv /tmp/gotools/bin/* /usr/local/bin/ \
  && mv /tmp/gotools/gocode-gomod /usr/local/bin/ \
  && cd ~ \
  && rm -rf /tmp/gotools \
  # Install mage
  && curl -sSL "https://github.com/magefile/mage/releases/download/v${MAGE_VERSION}/mage_${MAGE_VERSION}_Linux-64bit.tar.gz" | sudo tar -xz -C /usr/local/bin mage \
  # Install gotestsum
  && curl -sSL "https://github.com/gotestyourself/gotestsum/releases/download/v${GOTESTSUM_VERSION}/gotestsum_${GOTESTSUM_VERSION}_linux_amd64.tar.gz" | sudo tar -xz -C /usr/local/bin gotestsum \
  # Install golangci-lint
  && curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b /usr/local/bin 2>&1 \
  # Setup Go directories
  && mkdir -p "$GOPATH/src" "$GOPATH/bin" "$GOPATH/pkg/mod" \
  && chmod -R 777 "$GOPATH" \
  # Install docker
  && apt-get install -y apt-transport-https gnupg-agent software-properties-common lsb-release \
  && curl -fsSL https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/gpg | (OUT=$(apt-key add - 2>&1) || echo $OUT) \
  && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]') $(lsb_release -cs) stable" \
  && apt-get update \
  && apt-get install -y docker-ce-cli \
  # Install docker compose
  && curl -sSL "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-Linux-x86_64" -o /usr/local/bin/docker-compose \
  && chmod +x /usr/local/bin/docker-compose \
  # Create a docker group and add user to docker group
  && groupadd --gid $DOCKER_GID docker \
  && usermod -aG docker $USERNAME \
  # Install eslint globally
  && npm install -g eslint \
  # Install quasar cli
  && yarn global add @quasar/cli \
  # Install Tilt
  && curl -fsSL https://raw.githubusercontent.com/windmilleng/tilt/master/scripts/install.sh | bash \
  # Install etcdctl
  && mkdir -p /tmp/etcd \
  && curl --fail --silent --show-error -L ${GITHUB_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz \
  && tar xzf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd --strip-components=1 \
  && cp /tmp/etcd/etcdctl /usr/local/bin/ \
  && rm -rf /tmp/etcd* \
  # Clean up
  && apt-get autoremove -y \
  && apt-get clean -y \
  && rm -rf /var/lib/apt/lists/*

ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

# Update this to "on" or "off" as appropriate
ENV GO111MODULE=auto

# switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog
