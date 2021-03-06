FROM docker.io/nvidia/cuda:11.2.1-cudnn8-devel-ubuntu20.04
LABEL maintainer "BMW AG [Munich]"

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

ARG GPU=1
ARG CUDNN=1
ARG CUDNN_HALF=0
ARG OPENCV=1
ARG OPENMP=0

# To save you a headache
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8
ENV LIBRARY_PATH /usr/local/cuda/lib64/stubs

# Install needed libraries
RUN apt-get update && \
	apt-get upgrade -y && \
	apt-get install -y clang-format wget apt-utils build-essential\
	checkinstall cmake pkg-config yasm git vim curl\
	autoconf automake libtool libopencv-dev build-essential


# Install python-dev and pip
RUN apt-get update && apt-get install -y python3-pip python3.8-dev

# Install all needed python librarires
COPY ./docker/requirements.txt .
RUN pip3 install -r requirements.txt

# Create working directory
WORKDIR /training

# Fetch Repo
RUN git clone https://github.com/AlexeyAB/darknet.git
WORKDIR /training/darknet

# Modify Makefile according to needed env variables
RUN \
	sed -i ':a;N;$!ba;s/getchar();/ /6' ./src/detector.c && \
	sed -i 's/fprintf(stderr, "%d/\/\/fprintf(stderr, "%d/' ./src/detector.c && \
	sed -i 's/fmax(calc_map_for_each, 100)/300/' ./src/detector.c && \
	sed -i 's/next_map_calc = fmax(next_map_calc, ne/\/\/next_map_calc = fmax(next_map_calc, ne/' ./src/detector.c && \
	sed -i "s/GPU=.*/GPU=${GPU}/" Makefile && \
	sed -i "s/CUDNN=0/CUDNN=${CUDNN}/" Makefile && \
	sed -i "s/CUDNN_HALF=0/CUDNN_HALF=${CUDNN_HALF}/" Makefile && \
	sed -i "s/OPENCV=0/OPENCV=${OPENCV}/" Makefile && \
	sed -i "s/OPENMP=0/OPENMP=${OPENMP}/" Makefile && \
	make

RUN sed -i "s/names = data\/coco.names/names = \/training\/darknet\/data\/coco.names/" cfg/coco.data

WORKDIR /training

# Default command
CMD ["python3", "main.py"]
