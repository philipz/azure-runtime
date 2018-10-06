FROM ubuntu:18.04@sha256:de774a3145f7ca4f0bd144c7d4ffb2931e06634f11529653b23eba85aef8e378

RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y gpg
RUN curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
RUN mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
RUN sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-bionic-prod bionic main" > /etc/apt/sources.list.d/dotnetdev.list'
RUN apt-get update
RUN apt-get install -y azure-functions-core-tools

RUN groupadd -r func && useradd --no-log-init -r -g func func
RUN mkdir /.azurefunctions && chown func.func /.azurefunctions
USER func:func
