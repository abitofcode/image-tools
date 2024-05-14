# https://wiki.harvard.edu/confluence/display/DigitalImaging/Installing+OpenJPEG+on+Windows+10%2C+Linux%2C+and+MacOS
FROM ubuntu:22.04

# Create a user (on ubuntu)
# RUN useradd -rm -d /app -s /bin/bash -g root -G sudo -u 1001 app
# USER app
# WORKDIR /app

# Prevent interactive response
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get upgrade -y
RUN apt-get install pkg-config -y
RUN apt-get install valgrind emacs-nox libltdl-dev libtiff-tools exiftool git git-lfs cmake liblcms2-dev libtiff-dev \
    libpng-dev libz-dev unzip libzstd-dev libwebp-dev build-essential hwinfo -y
RUN apt-get install wget

# Install python
RUN apt-get install python3 python3-distutils -y

WORKDIR /install/python

RUN wget https://bootstrap.pypa.io/get-pip.py
RUN python3 get-pip.py
RUN apt-get install python3-jpylyzer -y

# openjpeg
WORKDIR /install/openjpeg
RUN wget https://github.com/uclouvain/openjpeg/archive/master.zip && unzip master.zip
WORKDIR /install/openjpeg/openjpeg-master/build
RUN cmake -DCMAKE_BUILD_TYPE=Release ..
RUN make && make install && make clean

# Install ImageMagick from source
WORKDIR /install/magick
RUN wget https://github.com/ImageMagick/ImageMagick/archive/refs/heads/main.zip
RUN unzip main.zip
WORKDIR /install/magick/ImageMagick-main
#Make sure system sees the libopenjp2.pc file just created by config. To look for the libopenjp2.pc files: find ~ -type f | grep libopenjp2.pc
ENV PKG_CONFIG_PATH=/install/openjpeg/openjpeg-master/build
# Check pkg-config path with
# pkg-config --variable pc_path pkg-config
RUN ./configure --enable-shared
RUN make && make install && make clean

# Needed to avoid error
RUN export LD_LIBRARY_PATH=/usr/local/lib

# GrokImage compression
# Why install GrokImageCompression?
# The original JPEG 2000 standard only accommodated three color encoding declarations: sGray, sYCC, and sRGB. As color management gained popularity and a wider variety of ICC display profiles were being embedded in images, the standard was amended in 2004 (15444-1annexi.pdf) to accommodate the use of restricted ICC profiles.

# As of the openjp2 library v2.3.1., OpenJPEG does not carry over the ICC display profiles embedded within the source image, but instead converts any color encoding to sRGB on output.

# Example showing how to perform an ICC profile-to-profile conversion using ImageMagick.
# GrokImageCompression is an open-source JP2 encoder based on the OpenJPEG code that produces JP2 images encoded with the same colorspace -- includes the same ICC display profile -- resident in the source image.

# Want to run Grok Image Compression on a Mac? It can be installed via Homebrew with the terminal command: brew install grokj2k
#ldconfig

#RUN apt-get install -y gcc-10 g++-10 cpp-10
RUN apt-get install -y gcc-11 g++-11 cpp-11

#RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 100 --slave /usr/bin/g++ g++ /usr/bin/g++-10 --slave /usr/bin/gcov gcov /usr/bin/gcov-10
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 100 --slave /usr/bin/g++ g++ /usr/bin/g++-11 --slave /usr/bin/gcov gcov /usr/bin/gcov-11


WORKDIR /install/compression
RUN wget https://github.com/GrokImageCompression/grok/archive/master.zip
RUN unzip master.zip
WORKDIR /install/compression/grok-master/build

# Testing in the CMakeList.txt file causes the build to fail. Line 216 - add_subdirectory(tests)
# so skipping the test for the moment.
# RUN apt-get install -y vim
RUN cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=OFF ..
#To maximize processing speed, you may specify the number of processing cores Grok should use with the command: make -j[number of cores]. To find the machine's cores count, run hwinfo --short.
RUN make && make install && make clean && ldconfig

# Copy the icc profiles into /icc (https://www.color.org/srgbprofiles.xalter)
WORKDIR /icc
COPY icc/* .

# Install Python Deep Zoom image generator
WORKDIR /install/deepzoom
RUN git clone https://github.com/openzoom/deepzoom.py.git
WORKDIR /install/deepzoom/deepzoom.py
# ANTIALIAS was removed in Pillow 10.0.0 which deepzoom relies on so we need to install Pillow 9.5.0
RUN pip install Pillow==9.5.0
RUN pip install .

WORKDIR /app

CMD ["/bin/bash"]
