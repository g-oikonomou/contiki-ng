FROM 32bit/ubuntu:16.04

# Tools
RUN apt-get update && \
  apt-get install -y --no-install-recommends \
  build-essential doxygen git wget unzip python-serial python-pip \
  default-jdk ant srecord iputils-tracepath rlwrap \
  mosquitto mosquitto-clients gdb apt-transport-https > /dev/null \
  && apt-get clean

# Create user, enable X forwarding, add to group dialout
# -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix
RUN export uid=1000 gid=1000 && \
    mkdir -p /home/user && \
    echo "user:x:${uid}:${gid}:user,,,:/home/user:/bin/bash" >> /etc/passwd && \
    echo "user:x:${uid}:" >> /etc/group && \
    echo "user ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers && \
    chmod 0440 /etc/sudoers && \
    chown ${uid}:${gid} -R /home/user && \
    usermod -aG dialout user

# Set user for what comes next
USER user

# Environment variables
ENV JAVA_HOME           /usr/lib/jvm/default-java
ENV HOME                /home/user
ENV CONTIKI_NG          ${HOME}/contiki-ng
ENV COOJA               ${CONTIKI_NG}/tools/cooja
ENV                     PATH="${HOME}:${PATH}"
WORKDIR                 ${HOME}

# Create Cooja shortcut
RUN echo "#!/bin/bash\nant -Dbasedir=${COOJA} -f ${COOJA}/build.xml run" > ${HOME}/cooja && \
  chmod +x ${HOME}/cooja

# Install Mono and libcanberra-gtk:i386 (for Renode)
RUN sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF \
  && echo "deb https://download.mono-project.com/repo/ubuntu stable-xenial/snapshots/5.12.0.226 main" | sudo tee /etc/apt/sources.list.d/mono-official-stable.list \
  && sudo apt-get update \
  && sudo apt-get install -y mono-complete gksu libgtk2.0-0 screen uml-utilities gtk-sharp2 libcanberra-gtk-module:i386 \
  && sudo apt-get clean

# Download, build and install Renode
RUN git clone https://github.com/renode/renode.git \
  && cd ${HOME}/renode \
  && git checkout v1.3 \
  && ./build.sh
ENV PATH="${HOME}/renode:${PATH}"

# By default, we use a Docker bind mount to share the repo with the host,
# with Docker run option:
# -v <HOST_CONTIKI_NG_ABS_PATH>:/home/user/contiki-ng
# Alternatively, uncomment the next two lines to download Contiki-NG and pre-compile Cooja.
#RUN git clone --recursive https://github.com/contiki-ng/contiki-ng.git ${CONTIKI_NG}
#RUN ant -q -f ${CONTIKI_NG}/tools/cooja/build.xml jar

# Working directory
WORKDIR ${CONTIKI_NG}

# Enable IPv6 -- must be done at runtime, hence added to .profile
RUN echo "sudo sysctl -w net.ipv6.conf.all.disable_ipv6=0 > /dev/null" >> ${HOME}/.profile

# Start a bash
CMD bash --login
