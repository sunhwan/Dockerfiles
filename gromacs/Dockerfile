FROM mambaorg/micromamba:2.0.7 AS micromamba

FROM nvidia/cuda:12.6.3-cudnn-devel-ubuntu24.04 AS build

USER root
ARG VERSION=2025.1

# install required packages
RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    git \
    cmake \
    gnupg \
    libhwloc-dev \
    libblas-dev \
    liblapack-dev \
    libx11-dev \
    ninja-build \
    wget \
  && rm -rf /var/lib/apt/lists/*


# if your image defaults to a non-root user, then you may want to make the
# next 3 ARG commands match the values in your image. You can get the values
# by running: docker run --rm -it my/image id -a
ARG MAMBA_USER=mambauser
ARG MAMBA_USER_ID=57439
ARG MAMBA_USER_GID=57439
ENV MAMBA_USER=$MAMBA_USER
ENV MAMBA_ROOT_PREFIX="/opt/conda"
ENV MAMBA_EXE="/bin/micromamba"

COPY --from=micromamba "$MAMBA_EXE" "$MAMBA_EXE"
COPY --from=micromamba /usr/local/bin/_activate_current_env.sh /usr/local/bin/_activate_current_env.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_shell.sh /usr/local/bin/_dockerfile_shell.sh
COPY --from=micromamba /usr/local/bin/_entrypoint.sh /usr/local/bin/_entrypoint.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_initialize_user_accounts.sh /usr/local/bin/_dockerfile_initialize_user_accounts.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_setup_root_prefix.sh /usr/local/bin/_dockerfile_setup_root_prefix.sh

RUN /usr/local/bin/_dockerfile_initialize_user_accounts.sh && \
    /usr/local/bin/_dockerfile_setup_root_prefix.sh

USER $MAMBA_USER

SHELL ["/usr/local/bin/_dockerfile_shell.sh"]

ENTRYPOINT ["/usr/local/bin/_entrypoint.sh"]
# Optional: if you want to customize the ENTRYPOINT and have a conda
# environment activated, then do this:
# ENTRYPOINT ["/usr/local/bin/_entrypoint.sh", "my_entrypoint_program"]

# You can modify the CMD statement as needed....
CMD ["/bin/bash"]

WORKDIR /tmp

RUN wget ftp://ftp.gromacs.org/gromacs/gromacs-${VERSION}.tar.gz && \
    tar -xvzf gromacs-${VERSION}.tar.gz

RUN git clone https://github.com/Colvars/colvars.git && \
    cd colvars && \
    ./update-colvars-code.sh /tmp/gromacs-${VERSION}
  
WORKDIR /tmp/gromacs-${VERSION}

RUN mkdir build && \
    cd build && \
    cmake .. \
      -DCMAKE_INSTALL_PREFIX=/gromacs \
      -DGMX_BUILD_OWN_FFTW=ON \
      -DGMX_GPU=CUDA \
      -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda \
      -DGMX_MPI=OFF \
      -DGMX_OPENMP=ON \
      -DREGRESSIONTEST_DOWNLOAD=ON && \
    make -j8 && \
    make check

USER root

RUN cd build && make install


#
# production
#

FROM nvidia/cuda:12.6.3-cudnn-devel-ubuntu24.04

USER root

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    libblas-dev \
    liblapack-dev \
  && rm -rf /var/lib/apt/lists/*

COPY --from=build /gromacs /gromacs

# Environment variables
ENV PATH=$PATH:/gromacs/bin
ENV LD_LIBRARY_PATH=/gromacs/lib

