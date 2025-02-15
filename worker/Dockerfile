FROM ubuntu:focal

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && apt install -y software-properties-common git

# UHD/lime drivers
RUN add-apt-repository -y ppa:pothosware/framework && \
    add-apt-repository -y ppa:pothosware/support && \
    add-apt-repository -y ppa:myriadrf/drivers && \
    add-apt-repository -y ppa:ettusresearch/uhd && \
    apt update

RUN apt install -y --no-install-recommends \
    libuhd-dev libuhd4.8.0 uhd-host \
    soapysdr-tools libsoapysdr-dev \
    soapysdr-module-lms7 \
    soapysdr-module-bladerf

ENV UHD_IMAGES_DIR=/usr/share/uhd/images/

RUN uhd_images_downloader

# build deps
RUN apt install -y --no-install-recommends \
    cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev \
    libzmq3-dev libboost-system-dev libboost-test-dev libboost-thread-dev \
    g++ make pkg-config libpython2-dev python-numpy swig libi2c-dev \
    libboost-program-options-dev libconfig++-dev libusb-1.0-0-dev swig

# Get srsLTE, compile and install
RUN git clone https://github.com/srsran/srsRAN_4G

#COPY ./srsRAN_4G /srsRAN_4G

COPY sib_logger.patch /srsRAN_4G

WORKDIR /srsRAN_4G/

# origin/master at 2024.12.25
RUN git checkout ec29b0c1ff79cebcbe66caa6d6b90778261c42b8 && git apply sib_logger.patch && \
    mkdir build && cd build && \
    cmake -DUSE_LTE_RATES=ON ../ && make -j`nproc` cell_search srsue && make install && \
    ldconfig

RUN cp /srsRAN_4G/build/lib/examples/cell_search /usr/bin/

RUN mkdir /vol/

WORKDIR /vol

ENTRYPOINT ["bash", "-c", "cp /vol/helpers/uhd_images/*.bin /usr/share/uhd/images/ 2>/dev/null || true && bash"]