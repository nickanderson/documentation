FROM ubuntu:22.04

# We need sudo before all
RUN apt-get -qy update && \
    apt-get -qy install sudo

# Add jenkins user
RUN useradd -s /bin/bash jenkins
RUN echo "jenkins    ALL=(ALL)       NOPASSWD: ALL">> /etc/sudoers
RUN mkdir -p /home/jenkins
RUN chown -R jenkins:jenkins /home/jenkins

# Run rest of setup as jenkins user
COPY install.sh /
COPY install_hugo.sh /
USER jenkins
WORKDIR /home/jenkins
RUN bash -x /install.sh
RUN bash /install_hugo.sh

# This is where our repos will be
WORKDIR /nt
