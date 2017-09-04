# Install pykaldi in a Docker container
# With code from 
# https://github.com/RaRe-Technologies/gensim/blob/develop/docker/Dockerfile
FROM ubuntu:16.04

WORKDIR /root

ENV PROTOBUF_GIT "https://github.com/google/protobuf.git"
ENV NINJA_GIT "https://github.com/ninja-build/ninja.git"
ENV CLIF_GIT "https://github.com/google/clif.git"

# update ubuntu and install essentials
RUN apt-get update \
    && apt-get install -y \
    git=1:2.7.4-0ubuntu1 \
    cmake=3.5.1-1ubuntu3 \
    autoconf=2.69-9 \
    automake=1:1.15-4ubuntu1 \
    libtool=2.4.6-0.1 \
    curl=7.47.0-1ubuntu2.2 \
    make=4.1-6 \
    g++=4:5.3.1-1ubuntu1 \
    unzip=6.0-20ubuntu1 \
    python-dev=2.7.11-1 \
    build-essential=12.1ubuntu2 \
    python3=3.5.1-3 \
    python3-pip=8.1.1-2ubuntu0.4 \
    python=2.7.11-1 \
    python-pip=8.1.1-2ubuntu0.4 \
    virtualenv=15.0.1+ds-3ubuntu1 \
    libatlas3-base=3.10.2-9 \
    wget=1.17.1-1ubuntu1.2 \
    zlib1g-dev=1:1.2.8.dfsg-2ubuntu4.1 \
    subversion=1.9.3-2ubuntu1.1 \
    pkg-config=0.29.1-0ubuntu1


# Upgrade pip
RUN pip2 install --upgrade pip

# install necessary python packages
RUN pip2 install \
    numpy==1.13.1 \
    setuptools==27.2.0 \
    jupyter==1.0.0

# Install protobuf with python package
RUN git clone $PROTOBUF_GIT \
    && cd protobuf \
    && ./autogen.sh \
    && ./configure \
    && make -j \
    && make install \
    && ldconfig \
    && cd python \
    && python2 setup.py build \
    && python2 setup.py install \
    && python3 setup.py build \
    && python3 setup.py install

# Install ninja
RUN git clone $NINJA_GIT \
    && cd ninja \
    && ./configure.py --bootstrap \
    && cp ninja /usr/local/bin

# Install clif
RUN git clone $CLIF_GIT \
    && cd clif \
    && ./INSTALL.sh $(which python2)

# # NOTE (VM): Args invalidate cache, thus from this point forward
# #              everything happens every build
# # Update this with your own credentials
# # at time of build
ARG githubuser
ARG githubpasswd

ENV KALDI_GIT https://$githubuser:$githubpasswd@github.com/usc-sail/kaldi-pykaldi.git
ENV PYKALDI https://$githubuser:$githubpasswd@github.com/usc-sail/pykaldi.git

# Install (our) kaldi
RUN git clone $KALDI_GIT kaldi \
    && cd kaldi/tools \
    && ./extras/check_dependencies.sh \
    && make -j4 \
    && cd ../src \
    && ./configure --shared \
    && make clean -j \
    && make depend -j \
    && make -j4

# # set env variables
ENV PATH "/root/opt/clif/bin:$PATH"
ENV KALDI_DIR "/root/kaldi"
ENV LD_LIBRARY_PATH "/root/protobuf/lib:${LD_LIBRARY_PATH}"
ENV CLIF_CXX_FLAGS "-I$CLIF_DIR/clang/lib/clang/5.0.0/include"
ENV DEBUG 1

# Download pykaldi from git and install
RUN git clone $PYKALDI pykaldi \
    && cd pykaldi \
    && python2 setup.py install

# Overwrite variables so that binary doesn't keep password
ENV KALDI_GIT ""
ENV PYKALDI ""

# # Add run permission to startup script
RUN chmod +x pykaldi/docker/start_jupyter_notebook.sh 

# # Define the starting command for this container and expose its running port
CMD sh -c '/root/pykaldi/docker/start_jupyter_notebook.sh 9000'
EXPOSE 9000