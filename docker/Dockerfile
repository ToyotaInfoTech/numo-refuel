# =============================================================================
# NUMo-Refuel — minimal headless SUMO 1.19.0 runtime
# =============================================================================
# Copyright (c) 2026 Toyota Motor Corporation
# SPDX-License-Identifier: MIT
#
# Licensing: see LICENSE.md in the NUMo-Refuel repository. The MIT License covers
# this build file only, not the Eclipse SUMO runtime that the image installs (EPL
# 2.0) or the other dependencies pulled in with it.
#
# This image contains ONLY the SUMO 1.19.0 runtime. The scenario bundle
# is NOT baked in; mount it at /scenario (see README.md).
FROM python:3.11-slim@sha256:9c900dea9e8fb7e16277c179b555cc72d29a352dbc33cff48ad5a0412fd5bfc7

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libx11-6 libxext6 libxrender1 libgl1 \
    && rm -rf /var/lib/apt/lists/* \
    && pip install --no-cache-dir eclipse-sumo==1.19.0

WORKDIR /scenario

# Default: run the 24-hour scenario headless (end=86400 from the sumocfg).
# Append SUMO options after the image name to override, e.g.
#   docker run ... numo-refuel --end 3600 --tripinfo-output /out/tripinfo.xml
# Write outputs to a writable mount such as /out; WORKDIR /scenario is normally
# mounted read-only (see the run instructions in README.md).
ENTRYPOINT ["sumo", "-c", "nagoya_with_refuel.sumocfg"]
