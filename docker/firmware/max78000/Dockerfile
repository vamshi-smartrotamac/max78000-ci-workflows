FROM ubuntu:24.04

ARG DEBIAN_FRONTEND=noninteractive

# Base tooling + ccache (required by ccache-artifact-location) + openssh-client
# (SSH private-submodule checkout in consuming repos).
# ca-certificates: TLS root certs for the HTTPS fetches below (git clone / wget).
RUN apt-get update && apt-get install -y --no-install-recommends \
        git make wget xz-utils ca-certificates clang-format python3 ccache openssh-client \
    && ln -sf /usr/bin/python3 /usr/local/bin/python \
    && rm -rf /var/lib/apt/lists/*

# --- ARM bare-metal toolchain (arm-none-eabi) ---
# The MSDK *git repo* does NOT ship the compiler (only the MSDK installer bundles
# it under Tools/GNUTools). Since we clone the repo, we install the toolchain here.
ARG TOOLCHAIN_VERSION=13.2.Rel1
ARG TOOLCHAIN=arm-gnu-toolchain-${TOOLCHAIN_VERSION}-x86_64-arm-none-eabi
RUN wget -q https://developer.arm.com/-/media/Files/downloads/gnu/${TOOLCHAIN_VERSION}/binrel/${TOOLCHAIN}.tar.xz \
    && mkdir -p /opt && tar xf ${TOOLCHAIN}.tar.xz -C /opt \
    && rm ${TOOLCHAIN}.tar.xz
ENV GCC_ARM_PATH=/opt/${TOOLCHAIN}/bin

# --- MaximSDK (MSDK): sparse-clone only the build tree ---
# The full repo is ~850 MB; the bulk is Examples/ (all demo apps). A partial
# (blob:none) + sparse checkout of Libraries/ + Tools/ pulls only what the
# firmware build needs. Tools/ is required: the target .mk includes
# Tools/Flash/flash.mk, so a Libraries-only checkout fails at build time.
ARG MSDK_VERSION=v2024_10
RUN git clone --depth 1 --filter=blob:none --sparse --branch ${MSDK_VERSION} \
        https://github.com/analogdevicesinc/msdk.git /opt/MaximSDK \
    && git -C /opt/MaximSDK sparse-checkout set Libraries Tools
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
