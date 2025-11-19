FROM ghcr.io/trueforge-org/ubuntu:24.4@sha256:0628c88c702e66783b5565dadf138d0f878ee3c5e8ef8fc7c52b28224ae4d174

USER root

ARG TARGETARCH
ARG VERSION

ENV NVIDIA_DRIVER_CAPABILITIES="compute,video,utility"
# https://github.com/dlemstra/Magick.NET/issues/707#issuecomment-785351620

ENV MALLOC_TRIM_THRESHOLD_=131072

RUN apt-get update && apt-get install -y --no-install-recommends \
      apt-transport-https \
      at \
      libjemalloc2 \
      mesa-va-drivers \
      xmlstarlet \
    && echo "**** add Jellyfin repo *****" \
    && curl -fsSL https://repo.jellyfin.org/jellyfin_team.gpg.key | gpg --dearmor -o /usr/share/keyrings/jellyfin-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/jellyfin-archive-keyring.gpg] https://repo.jellyfin.org/ubuntu noble main" \
         > /etc/apt/sources.list.d/jellyfin.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
      jellyfin=${VERSION}* \
    && chown -R apps:apps /app && chmod -R 755 /app \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Intel VAAPI Tone mapping dependencies:
# Prefer NEO to Beignet since the latter one doesn't support Comet Lake or newer for now.
# Do not use the intel-opencl-icd package from repo since they will not build with RELEASE_WITH_REGKEYS enabled.
# https://github.com/intel/compute-runtime/releases
RUN if test "${PACKAGE_ARCH}" = "amd64"; then \
    mkdir intel-compute-runtime \
 && cd intel-compute-runtime \
 && curl -LO https://github.com/intel/intel-graphics-compiler/releases/download/igc-${IGC_VERSION}/intel-igc-core_${IGC_VERSION}_amd64.deb \
         -LO https://github.com/intel/intel-graphics-compiler/releases/download/igc-${IGC_VERSION}/intel-igc-opencl_${IGC_VERSION}_amd64.deb \
         -LO https://github.com/intel/compute-runtime/releases/download/${NEO_VERSION}/intel-opencl-icd_${NEO_VERSION}_amd64.deb \
         -LO https://github.com/intel/compute-runtime/releases/download/${NEO_VERSION}/intel-level-zero-gpu_${LEVEL_ZERO_VERSION}_amd64.deb \
         -LO https://github.com/intel/compute-runtime/releases/download/${NEO_VERSION}/libigdgmm12_${GMMLIB_VERSION}_amd64.deb \
 && apt-get install --no-install-recommends --no-install-suggests -f -y ./*.deb \
 && cd .. \
 && rm -rf intel-compute-runtime \
 ; fi \
 && apt-get clean autoclean --yes \
 && apt-get autoremove --yes \
 && rm -rf /var/cache/apt/archives* /var/lib/apt/lists/*

# Rockchip RK3588 libmali OpenCL dependencies:
RUN if test "${PACKAGE_ARCH}" = "arm64"; then \
    mkdir libmali-rockchip \
 && cd libmali-rockchip \
 && curl -LO https://github.com/tsukumijima/libmali-rockchip/releases/download/${MALI_PKG_TAG}/libmali-${MALI_PKG_CFG}_${MALI_PKG_VER}.deb \
 && apt-get install --no-install-recommends --no-install-suggests -f -y ./*.deb \
 && cd .. \
 && rm -rf libmali-rockchip \
 ; fi \
 && apt-get clean autoclean --yes \
 && apt-get autoremove --yes \
 && rm -rf /var/cache/apt/archives* /var/lib/apt/lists/*

USER apps
COPY ./root/ /
COPY ./ /

EXPOSE 8096 8920
VOLUME /config
