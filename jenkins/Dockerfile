FROM jenkins/jenkins:lts-jdk17

USER root

# Docker CLI + Compose plugin (o daemon é o do host, via /var/run/docker.sock)
COPY --from=docker:cli /usr/local/bin/docker /usr/local/bin/docker
COPY --from=docker:cli /usr/local/libexec/docker/cli-plugins/docker-compose /usr/local/libexec/docker/cli-plugins/docker-compose
