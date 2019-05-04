FROM ubuntu:18.04

RUN set -e; \
  export DEBIAN_FRONTEND=noninteractive; \
  apt-get update; \
  apt-get install -y --no-install-recommends curl wget ca-certificates gpg dirmngr gpg-agent; \
  wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb; \
  dpkg -i packages-microsoft-prod.deb; \
  curl -sL https://deb.nodesource.com/setup_8.x | bash -; \
  curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg; \
  mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg; \
  sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-bionic-prod bionic main" > /etc/apt/sources.list.d/dotnetdev.list'; \
  apt-get update; \
  apt-get install -y nodejs azure-functions-core-tools dotnet-sdk-2.2; \
  rm -rf /var/lib/apt/lists/* /var/log/apt

RUN groupadd -r func && useradd -d /home/func --no-log-init -r -g func func
RUN mkdir /home/func
RUN chown -R func:func /home/func
RUN mkdir /.azurefunctions && chown func.func /.azurefunctions
USER func:func
