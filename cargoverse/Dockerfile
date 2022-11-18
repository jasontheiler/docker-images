# syntax=docker/dockerfile:1
# See: https://docs.docker.com/engine/reference/builder/

FROM --platform=$BUILDPLATFORM tonistiigi/xx:master AS xx-build

FROM --platform=$BUILDPLATFORM rust:1-slim AS build
COPY --from=xx-build / /
ENV TARGETPLATFORM=${BUILDPLATFORM}
RUN --mount=type=cache,target=/var/cache/apt/,sharing=locked \
    --mount=type=cache,target=/var/lib/apt/,sharing=locked \
    <<EOF
    apt update
    apt install -y --no-install-recommends clang lld curl
    curl -fsSL "https://github.com/cargo-bins/cargo-binstall/releases/latest/download/cargo-binstall-$(xx-cargo --print-target).tgz" | tar -xzC /usr/local/cargo/bin/
EOF
ARG TARGETPLATFORM
ENV CARGO_INSTALL_ROOT=/xx-cargo/
RUN --mount=type=cache,target=/var/cache/apt/,sharing=locked \
    --mount=type=cache,target=/var/lib/apt/,sharing=locked \
    --mount=type=cache,target=/usr/local/cargo/git/db/ \
    --mount=type=cache,target=/usr/local/cargo/registry/cache/ \
    --mount=type=cache,target=/usr/local/cargo/registry/index/ \
    <<EOF
    xx-apt install -y --no-install-recommends xx-c-essentials
    xx-cargo binstall --no-confirm cargo-binstall cargo-chef cargo-watch
EOF

FROM tonistiigi/xx:master AS xx-final

FROM rust:1-slim AS final
COPY --from=xx-final / /
RUN --mount=type=cache,target=/var/cache/apt/,sharing=locked \
    --mount=type=cache,target=/var/lib/apt/,sharing=locked \
    <<EOF
    apt update
    apt install -y --no-install-recommends clang lld curl
EOF
COPY --from=build /xx-cargo/bin/* /usr/local/cargo/bin/
