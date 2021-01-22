
# These  instructions  for  getting  and  building  MicroPython  assume  you  are  using  Raspberry  Pi  OS  running  on  aRaspberry Pi 4, or an equivalent Debian-based Linux distribution running on another platform.
FROM ubuntu:20.04

ARG BRANCH=pico
ARG REPO=https://github.com/raspberrypi/micropython.git

ARG DEBIAN_FRONTEND=noninteractive

# To build the RP2040 MicroPython port, you’ll need to install some extra tools. To
# build projects you’ll need CMake, a cross-platform tool used to build the software,
# and the GNU Embedded Toolchain for Arm, which turns MicroPython’s C sourcecode  into
# a  binary  program  RP2040’s  processors  can  understand. `build-essential` is a 
# bundle  of  tools  you  need  to  buildcode  native  to  your  own  machine—this  is  
# needed  for  some  internal  tools  in  MicroPython  and  the  Pico  SDK.  You  can 
# install all of these via apt from the command line. Anything you already have 
# installed will be ignored by apt.
RUN apt-get update && \
    apt-get install --no-install-recommends -y \
        cmake \
        build-essential \
        git \
        openssh-client \
        ca-certificates \
        wget \
        python3 \
        libusb-1.0-0-dev \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# NOTE: install this with recommended, see github.com/raspberrypi/micropython/issues/2
RUN apt-get update && \
    apt-get install -y \
        gcc-arm-none-eabi \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

RUN mkdir /usr/local/share/ca-certificates/cacert.org \
    && wget -P /usr/local/share/ca-certificates/cacert.org http://www.cacert.org/certs/root.crt http://www.cacert.org/certs/class3.crt \
    && update-ca-certificates \
    && git config --global http.sslCAinfo /etc/ssl/certs/ca-certificates.crt

# It’s a good idea to create a pico directory to keep all pico-related checkouts in.
# These instructions create a pico directoryat ~/pico
WORKDIR /root/pico

# Then clone the micropython git repository. These instructions will fetch the latest
# version of the source code. Once  the  download  has  finished, the  source  code  for
# MicroPython  should  be  in  a  new directory  called  micropython.  TheMicroPython
# repository also contains pointers (submodules) to specific versions of libraries it
# needs to run on a particularboard, like the Pico SDK in the case of RP2040. We need to
# explicitly fetch these too
RUN git clone -b ${BRANCH} ${REPO} \
    && cd micropython \
    && git submodule update --init -- lib/pico-sdk \
    && cd lib/pico-sdk \
    && git submodule update --init

# To  build  the  port,  you  first  need  to  change  directory  into  the  micropython
# repository  containing  the  source.  If  you’ve  beenfollowing along with the
# instructions, you’ll need to go up two directories
WORKDIR /root/pico/micropython

# First we need to bootstrap a special tool for MicroPython builds, that ships with the
# source code:
RUN make -C mpy-cross

# We can now build the port we need for RP2040, that is, the version of MicroPython that
# has specific support for our chip.
RUN cd ports/rp2 \
    && make

# picotool
WORKDIR /root/pico/picotool
RUN git clone -b master https://github.com/raspberrypi/picotool.git . \
    && mkdir build \
    && cd build \
    && export PICO_SDK_PATH=~/pico/pico-sdk \
    & cmake ../ \
    & make

# Sanity check to ensure correct build
RUN picotool \
    info -a /root/pico/micropython/ports/rp2/build/firmware.uf2
WORKDIR /root/pico/micropython
ENTRYPOINT ["/usr/bin/env", "bash"]
