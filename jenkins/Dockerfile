FROM jenkins/jenkins:lts
USER root
RUN apt update && curl -fsSl https://get.docker.com | sh
RUN usermod -aG docker jenkins
# The chmod below may be a security risk
RUN chmod 666 /var/run/docker.sock
USER jenkins