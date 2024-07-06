FROM ubuntu:18.04

ARG HOST_USER_DIR=$HOME

# setup environment
ARG USER_DOCKER=blog
ENV USER_DOCKER=${USER_DOCKER}
ENV HOME_DOCKER=/home/${USER_DOCKER}

# install sudo
RUN apt update
RUN apt install -y sudo
RUN apt install -y git
# install openssh
RUN apt install -y openssh-*
RUN apt install -y build-essential \
			libssl-dev \
			libreadline-dev \
			zlib1g-dev \
			libyaml-dev \
			libsqlite3-dev \
			sqlite3 \
			libxml2-dev \
			libxslt1-dev \
			libcurl4-openssl-dev \
			software-properties-common \
			libffi-dev \
			rbenv

RUN rbenv install 2.4.1
RUN rbenv global 2.4.1
RUN apt install -y ruby2.5-dev
RUN apt install -y net-tools

#RUN echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
#RUN echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
#RUN echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
#RUN source ~/.bashrc
#RUN gem install rouge -v 3.30.0
#RUN gem install ffi -v 1.17.0
##RUN gem install bundler
#gem install bundler -v 2.3.27
# setup sudo user
RUN adduser --disabled-password --gecos '' $USER_DOCKER
RUN adduser $USER_DOCKER sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers


RUN mkdir -p ${HOME_DOCKER}/docker_share
# Define ENTRYPOINTS
#ENTRYPOINT ["-v ${HOST_USER_DIR}/docker_share:${HOME_DOCKER}/docker_share", "-v /tmp/.X11-unix:/tmp/.X11-unix", "-e DISPLAY=unix${DISPLAY}", "vivado:latest"]
#ENTRYPOINT ["blog:latest"]

# setup container environment
USER $USER_DOCKER
WORKDIR $HOME_DOCKER
CMD ["/bin/bash"]
