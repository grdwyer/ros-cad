FROM arm64v8/ros:foxy

#COPY qemu-arm-static /usr/bin
##############
# apt update #
##############
RUN apt-get update

RUN apt-get install -y wget git build-essential libgl1-mesa-dev libfreetype6-dev libglu1-mesa-dev libzmq3-dev libsqlite3-dev libicu-dev python3-dev libgl2ps-dev libfreeimage-dev libtbb-dev ninja-build bison autotools-dev automake libpcre3 libpcre3-dev tcl tcl-dev tk tk-dev libxmu-dev libxi-dev libopenblas-dev libboost-all-dev swig libxml2-dev cmake python3-pip curl

RUN dpkg-reconfigure --frontend noninteractive tzdata


############################################################
# OCCT 7.4.0                                               #
# Download the official source package from OCE repository #
############################################################
WORKDIR /opt/build
RUN wget https://github.com/tpaviot/oce/releases/download/official-upstream-packages/opencascade-7.4.0.tgz
RUN tar -zxvf opencascade-7.4.0.tgz >> installed_occt740_files.txt
RUN mkdir opencascade-7.4.0/build
WORKDIR /opt/build/opencascade-7.4.0/build

RUN ls /usr/include
RUN cmake -G Ninja \
 -DINSTALL_DIR=/opt/build/occt740 \
 -DBUILD_RELEASE_DISABLE_EXCEPTIONS=OFF \
 ..

RUN ninja install

RUN echo "/opt/build/occt740/lib" >> /etc/ld.so.conf.d/occt.conf
RUN ldconfig

RUN ls /opt/build/occt740
RUN ls /opt/build/occt740/lib

#############
# pythonocc #
#############
WORKDIR /opt/build
RUN git clone https://github.com/tpaviot/pythonocc-core
WORKDIR /opt/build/pythonocc-core
RUN git checkout 7.4.0
WORKDIR /opt/build/pythonocc-core/build

RUN cmake -G Ninja \
 -DOCE_INCLUDE_PATH=/opt/build/occt740/include/opencascade \
 -DOCE_LIB_PATH=/opt/build/occt740/lib \
 -DPYTHONOCC_BUILD_TYPE=Release \
 ..
 
RUN ninja install

############
# svgwrite #
############
RUN pip3 install svgwrite

###########
# Jupyter 
###########
RUN apt install libffi-dev
RUN pip3 install jupyterlab
RUN pip3 install notebook

#######
# NPM #
#######
WORKDIR /tmp
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN apt-get install -y nodejs
#######################
# Run pythonocc tests #
#######################
WORKDIR /opt/build/pythonocc-core/test
RUN python3 core_wrapper_features_unittest.py

##############################
# Install pythonocc examples #
##############################
WORKDIR /opt/build/
RUN git clone https://github.com/tpaviot/pythonocc-demos
WORKDIR /opt/build/pythonocc-demos

#############
# pythreejs #
#############
WORKDIR /opt/build
#RUN git clone https://github.com/jovyan/pythreejs
#WORKDIR /opt/build/pythreejs
#RUN git checkout 2.1.1
#RUN pip3 install .
#WORKDIR /opt/build/pythreejs/js
#RUN npm run autogen
#RUN npm run build:all
RUN pip3 install pythreejs
RUN jupyter nbextension install --py --symlink --sys-prefix pythreejs
RUN jupyter nbextension enable pythreejs --py --sys-prefix

########
# gmsh #
########
ENV CASROOT=/opt/build/occt740
WORKDIR /opt/build
RUN git clone https://gitlab.onelab.info/gmsh/gmsh
WORKDIR /opt/build/gmsh
RUN git checkout gmsh_4_5_3
WORKDIR /opt/build/gmsh/build

RUN cmake \
 -DCMAKE_BUilD_TYPE=Release \
 -DENABLE_OCC=ON \
 -DENABLE_OCC_CAF=ON \
 -DCMAKE_INSTALL_PREFIX=/usr/local \
 ..

RUN make -j3 && make install

###########
# Cleanup #
###########
RUN rm -rf /var/lib/apt/lists/*

#######################
# Python-OCC examples #
#######################

RUN mkdir -p /dev_ws/src
WORKDIR /dev_ws/src
RUN git clone https://github.com/tpaviot/pythonocc-demos.git

