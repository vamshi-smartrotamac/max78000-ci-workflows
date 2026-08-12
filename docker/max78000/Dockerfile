FROM ubuntu:24.04

ARG DEBIAN_FRONTEND=noninteractive

# Base tooling + ccache (ccache is required by the ccache-artifact-location action)
RUN apt-get update && apt-get install -y --no-install-recommends \
        git make wget xz-utils ca-certificates clang-format python3 ccache \
    && ln -sf /usr/bin/python3 /usr/local/bin/python \
    && rm -rf /var/lib/apt/lists/*

# --- ARM bare-metal toolchain (arm-none-eabi) ---
ARG TOOLCHAIN_VERSION=13.2.Rel1
ARG TOOLCHAIN=arm-gnu-toolchain-${TOOLCHAIN_VERSION}-x86_64-arm-none-eabi
RUN wget -q https://developer.arm.com/-/media/Files/downloads/gnu/${TOOLCHAIN_VERSION}/binrel/${TOOLCHAIN}.tar.xz \
    && mkdir -p /opt && tar xf ${TOOLCHAIN}.tar.xz -C /opt \
    && rm ${TOOLCHAIN}.tar.xz
ENV GCC_ARM_PATH=/opt/${TOOLCHAIN}/bin

# --- MaximSDK (MSDK): Libraries/, CMSIS, BSPs, build .mk files ---
ARG MSDK_VERSION=v2024_10
RUN git clone --depth 1 --branch ${MSDK_VERSION} \
        https://github.com/analogdevicesinc/msdk.git /opt/MaximSDK
ENV MAXIM_PATH=/opt/MaximSDK

# --- ccache masquerade (so ccache-artifact-location accelerates builds) ---
ARG CACHE_WS=/cache_ws
ENV CACHE_OUTPUT=$CACHE_WS/.ccache
ARG CCACHE_LINKS=$CACHE_WS/ccache-bin
WORKDIR $CCACHE_LINKS
RUN for tool in $(basename -a $GCC_ARM_PATH/*); do ln -s $(which ccache) $tool; done
ENV PATH="$CCACHE_LINKS:$PATH:$GCC_ARM_PATH"
ENV CCACHE_CONFIGPATH=$CACHE_WS/ccache.conf
RUN ccache -o cache_dir=$CACHE_OUTPUT

WORKDIR /work
