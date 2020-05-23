FROM debian:latest

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# This Dockerfile adds a non-root 'vscode' user with sudo access. However, for Linux,
# this user's GID/UID must match your local user UID/GID to avoid permission issues
# with bind mounts. Update USER_UID / USER_GID if yours is not 1000. See
# https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ARG DOWNLOAD_PATH=/tmp/docker-downloads
ARG MSP432P4_LAUNCHPAD=MSP_EXP432P401R

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Install bz2, ca-certificates, unzip, wget 
    && apt-get install -y \
        bzip2 \
        ca-certificates \
        unzip \
        wget \
    #
    # Create temp location
    && mkdir -p ${DOWNLOAD_PATH} \
    #
    # Install SimpleLink MSP432P4 SDK
    && echo "Downloading SimpleLink MSP432P4 SDK..." \
    && wget -q http://software-dl.ti.com/simplelink/esd/simplelink_msp432_sdk/3.40.01.02/simplelink_msp432p4_sdk_3_40_01_02.run \
    && chmod +x ./simplelink_msp432p4_sdk_3_40_01_02.run \
    && echo "Installing SimpleLink MSP432P4 SDK..." \
    && ./simplelink_msp432p4_sdk_3_40_01_02.run --mode unattended \
    && echo "SimpleLink MSP432P4 SDK install complete!" \
    #
    # Update imports.mak file
    && echo "Downloading imports.mak..." \
    && wget -q -O ${DOWNLOAD_PATH}/imports.mak https://raw.githubusercontent.com/leocamello/imports.mak/master/imports.mak \
    && echo "Updating imports.mak..." \
    && cp -rf ${DOWNLOAD_PATH}/imports.mak /opt/ti/simplelink_msp432p4_sdk_3_40_01_02/imports.mak \
    && echo "imports.mak updated!" \
    #
    # Install SysConfig tool
    && echo "Downloading SysConfig tool..." \
    && wget -q http://software-dl.ti.com/ccs/esd/sysconfig/sysconfig-1.4.0_1234-setup.run \
    && chmod +x ./sysconfig-1.4.0_1234-setup.run \
    && echo "Installing SysConfig tool..." \
    && ./sysconfig-1.4.0_1234-setup.run --mode unattended \
    && echo "SysConfig tool install complete!" \
    #
    # Install GNU Arm Embedded Toolchain
    && echo "Downloading GNU Arm Embedded Toolchain..." \
    && wget -q -O ${DOWNLOAD_PATH}/gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2 https://developer.arm.com/-/media/Files/downloads/gnu-rm/9-2019q4/gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2 \
    && echo "Installing GNU Arm Embedded Toolchain..." \
    && tar -xjf ${DOWNLOAD_PATH}/gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2 -C /opt \
    && echo "GNU Arm Embedded Toolchain install complete!" \
    #
    # Install FreeRTOS
    && echo "Downloading FreeRTOS..." \
    && wget -q -O ${DOWNLOAD_PATH}/FreeRTOSv10.3.1.zip https://github.com/FreeRTOS/FreeRTOS/releases/download/V10.3.1/FreeRTOSv10.3.1.zip \
    && echo "Installing FreeRTOS..." \
    && unzip ${DOWNLOAD_PATH}/FreeRTOSv10.3.1.zip -d /opt \
    && echo "FreeRTOS install complete!" \
    #
    # Build RTOS config
    && cd /opt/ti/simplelink_msp432p4_sdk_3_40_01_02/kernel/freertos/builds/${MSP432P4_LAUNCHPAD}/release/gcc/ \ 
    && echo "Building RTOS config..." \
    && /opt/ti/xdctools_3_60_02_34_core/gmake \ 
    && echo "RTOS config build complete!" \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Remove temp location
    && rm -rf ${DOWNLOAD_PATH} \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog