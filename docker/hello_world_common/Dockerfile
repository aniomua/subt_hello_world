# ABOUT
# This Dockerfile is the "base image" for both the development and submission containers

# most of the following comes from the melodic-ros-core docker image and ROS install page:
# https://hub.docker.com/_/ros/#!
# https://github.com/osrf/docker_images/blob/b075c7dbe56055d862f331f19e1e74ba653e181a/ros/melodic/ubuntu/bionic/ros-core/Dockerfile
# https://wiki.ros.org/melodic/Installation/Ubuntu

# Ubuntu 18.04 with CUDA and OpenGL support
# https://hub.docker.com/r/nvidia/cudagl/
# https://github.com/NVIDIA/nvidia-docker/wiki/CUDA
FROM nvidia/cudagl:11.3.1-devel-ubuntu18.04

          
# setup timezone
RUN echo 'Etc/UTC' > /etc/timezone && \
  ln -s /usr/share/zoneinfo/Etc/UTC /etc/localtime \
  && apt-get -qq update && apt-get -q -y install tzdata \
  && rm -rf /var/lib/apt/lists/* \
  && apt-get -qq clean

# setup sources.list and keys
RUN echo "deb [trusted=yes] http://packages.ros.org/ros/ubuntu bionic main" > /etc/apt/sources.list.d/ros1-latest.list \
  && apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

  RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak
  RUN echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse" >> /etc/apt/sources.list
  RUN echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse" >>/etc/apt/sources.list
  RUN echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse" >>/etc/apt/sources.list
  RUN echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse" >>/etc/apt/sources.list
  RUN apt-get update && \
  apt-get install -y python3-pip
  RUN pip3 install rosdepc -i https://pypi.tuna.tsinghua.edu.cn/simple

# 配置 UTF-8 环境
RUN apt-get update && \
    apt-get install -y locales && \
    sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && \
    locale-gen en_US.UTF-8 && \
    update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8

ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV PYTHONIOENCODING=utf-8



# install ROS (including dependencies for building packages) and other packages
RUN apt-get -qq update && apt-get -q -y install \
  libpcl-dev \
  libpcl-conversions-dev \
  build-essential \
  git \
  python-rosdep \
  python-rosinstall \
  python-rosinstall-generator \
  python-wstool \
  python3-vcstool \
  python3-colcon-common-extensions \
  ros-melodic-desktop \
  ros-melodic-teleop-twist-keyboard \
  ros-melodic-pcl-ros \
  tmux \
  vim \
  wget \
  && rosdepc init \
  && rm -rf /var/lib/apt/lists/* \
  && apt-get -qq clean


# sdformat8-sdf conflicts with sdformat-sdf installed from gazebo
# so we need to workaround this using a force overwrite
# Do this before installing ign-gazebo
# (then install ign-dome and ros to ign bridge)
RUN echo "deb [trusted=yes] http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list \
  && wget http://packages.osrfoundation.org/gazebo.key -O - | apt-key add - \
  && apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654 \
  && apt-get -qq update && apt-get -q -y install \
  ignition-dome \
  ros-melodic-ros-ign \
  && rm -rf /var/lib/apt/lists/* \
  && apt-get -qq clean

# Add a user with the same user_id as the user outside the container
# Requires a docker build argument `user_id`
ARG user_id
ENV USERNAME developer
RUN apt-get -qq update && apt-get -q -y install sudo \
  && rm -rf /var/lib/apt/lists/* \
  && useradd -U --uid 1000 -ms /bin/bash ${USERNAME} \
  && echo "$USERNAME:$USERNAME" | chpasswd \
  && adduser $USERNAME sudo \
  && echo "$USERNAME ALL=NOPASSWD: ALL" >> /etc/sudoers.d/${USERNAME}

# Commands below run as the user $USERNAME
USER ${USERNAME}

RUN rosdepc update

# save filenames and paths as either args (for repreated use in this Dockerfile)
# or environment variables (for use in other dockerfiles and/or container shell instances)
ARG ws_path=/home/${USERNAME}/workspaces
ARG repos_file=hello_world_deps.rosinstall
ENV DEPS_WS_PATH=${ws_path}/dependencies_ws
ENV SOLUTION_WS_PATH=${ws_path}/solution_ws
ENV PATH_TO_DEPS_WS_SETUP_BASH=${DEPS_WS_PATH}/install/setup.bash
ENV PATH_TO_SOLUTION_WS_SETUP_BASH=${SOLUTION_WS_PATH}/install/setup.bash

# set up source code dependencies
WORKDIR /home/${USERNAME}
COPY ${repos_file} .
RUN mkdir -p ${DEPS_WS_PATH}/src \
  && vcs import \
  --recursive \
  --input ${repos_file} \
  ${DEPS_WS_PATH}/src
# build the dependencies
WORKDIR ${DEPS_WS_PATH}


RUN [ "/bin/bash" , "-c" , \
  "source /opt/ros/melodic/setup.bash \
  && sudo apt-get -qq update \
  && rosdepc install --from-paths src --ignore-src -r -y \
    --skip-keys=ignition-transport9 \
    --skip-keys=ignition-gazebo4 \
    --skip-keys=ignition-math6 \
    --skip-keys=ignition-msgs6 \
    --skip-keys=ignition-plugin1 \
    --skip-keys=ignition-common3 \
    --skip-keys=ignition-launch3" ]
  RUN  sudo apt-get -q -y install ros-melodic-rotors-control
  RUN  sudo apt-get -q -y install ros-melodic-image-geometry
  RUN  sudo apt-get -q -y install ros-melodic-tf2-geometry-msgs
  RUN sudo apt-get -q -y install ros-melodic-control-toolbox 
  RUN sudo apt-get -q -y install ros-melodic-costmap-2d 
  RUN sudo apt-get -q -y install ros-melodic-lms1xx 
  RUN sudo apt-get -q -y install ros-melodic-robot-localization 
  RUN sudo apt-get -q -y install ros-melodic-twist-mux 
  RUN sudo apt-get -q -y install ros-melodic-move-base-msgs 
  ENV CUDA_ARCH_BIN=8.9          
  ENV CUDA_ARCH_PTX=8.9
  RUN sed -i \
  -e 's/-gencode arch=compute_30,code=sm_30/# &/' \
  -e 's/-gencode arch=compute_35,code=sm_35/# &/' \
  ${DEPS_WS_PATH}/src/darknet_ros/darknet_ros/CMakeLists.txt
  RUN [ "/bin/bash" , "-c" , \
  "source /opt/ros/melodic/setup.bash \
  && catkin_make  -DCMAKE_BUILD_TYPE=Release -DCUDA_ARCH_BIN=${CUDA_ARCH_BIN} \
      -DCUDA_ARCH_PTX=${CUDA_ARCH_PTX}  install \
  && sudo rm -rf /var/lib/apt/lists/* \
  && sudo apt-get -qq clean" ]
# set up a workspace for source code (this will be loaded as a volume)
RUN mkdir -p ${SOLUTION_WS_PATH}/src

# When running a container start in the developer's home folder
WORKDIR /home/$USERNAME
