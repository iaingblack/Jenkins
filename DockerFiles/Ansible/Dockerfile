# For running the Ansible Playbook

FROM python:3.6-stretch

ENV ANSIBLE_VERSION 2.9.23
ENV AZURE_COLLECTION 1.7.0
ENV ANSIBLE_PYTHON_INTERPRETER /usr/local/bin/python

ENV BUILD_PACKAGES \
  bash \
  curl \
  tar \
  openssh-client \
  sshpass \
  git \
  python3-dateutil \
  python3-httplib2 \
  python3-jinja2 \
  python3-paramiko \
  python3-pip \
  python3-yaml \
  ca-certificates \
  python3-dev \
  libpq-dev \
  musl-dev \
  gcc \
  build-essential \
  make \
  musl-dev \
  libffi-dev \
  libssl-dev \
  apt-transport-https \
  lsb-release \
  gnupg

RUN set -x && \
    echo "==> Upgrading system..."  && \
    apt-get update && apt-get upgrade -y && \
    echo "==> Adding system packages..."  && \
    apt-get install -y ${BUILD_PACKAGES}

RUN curl -sL https://packages.microsoft.com/keys/microsoft.asc | \
    gpg --dearmor | \
    tee /etc/apt/trusted.gpg.d/microsoft.asc.gpg > /dev/null

RUN AZ_REPO=$(lsb_release -cs) && \
    echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | \
    tee /etc/apt/sources.list.d/azure-cli.list

# https://packages.microsoft.com/repos/azure-cli/pool/main/a/azure-cli/
# Frontdoor not installed by default in the CLI. Remove the 2nd line when it is available
RUN apt-get update && apt-get install -y azure-cli=2.10.0-1~stretch && \
    az extension add --name front-door

RUN set -x && \
    echo "==> Adding python libs"  && \
    pip install --upgrade pip && \
    pip install xmltodict pywinrm

RUN set -x && \
    echo "==> Installing Ansible, Azure and Chocolatey Collections..."  && \
    pip install ansible==${ANSIBLE_VERSION} && \
    ansible-galaxy collection install azure.azcollection chocolatey.chocolatey --ignore-cert

RUN set -x && \
    echo "==> Adding hosts for convenience..."  && \
    mkdir -p /etc/ansible /ansible && \
    echo "[local]" >> /etc/ansible/hosts && \
    echo "localhost" >> /etc/ansible/hosts

RUN set -x && \
    echo "==> Adding Azure libraries - azure.mgmt.privatedns seems missing from ansible[azure] right now"  && \
    pip install cryptography>=2.3 openshift 'ansible[azure]' jq azure.mgmt.privatedns

RUN set -x && \
    echo "==> Adding Powershell Core to Use Invoke-RestMethod Later"  && \
    apt-get install -y curl gnupg apt-transport-https && \
    curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - && \
    sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-debian-stretch-prod stretch main" > /etc/apt/sources.list.d/microsoft.list' && \
    apt-get update && \
    apt-get install -y powershell

# RUN set -x && \
#     echo "==> Install Az Cmdlets"  && \
#     pwsh -command Install-Module -Name Az -Repository PSGallery -Force

RUN set -x && \
    echo "==> Cleaning up..."  && \
    apt-get clean && apt-get autoclean && rm -rf /tmp/* /var/tmp/* && rm -rf /var/lib/apt/lists/* && \
    rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin

# add core repository
RUN mkdir /app
WORKDIR /app
ADD . /app/

# clean dev data if present
RUN rm -rf /app/ansible/kube-configs/* && rm -rf /app/.git
