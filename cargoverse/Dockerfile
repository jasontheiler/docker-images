# syntax=docker/dockerfile:1
# See: https://docs.docker.com/engine/reference/builder/

FROM --platform=$BUILDPLATFORM tonistiigi/xx:master AS xx-build

FROM --platform=$BUILDPLATFORM rust:1-slim AS build
COPY --from=xx-build / /
ARG TARGETPLATFORM
ENV PKG_CONFIG_SYSROOT_DIR=/ CARGO_INSTALL_ROOT=/xx-cargo/
RUN --mount=type=cache,target=/var/cache/apt/,sharing=locked \
    --mount=type=cache,target=/var/lib/apt/,sharing=locked \
    --mount=type=cache,target=/usr/local/cargo/git/db/ \
    --mount=type=cache,target=/usr/local/cargo/registry/cache/ \
    --mount=type=cache,target=/usr/local/cargo/registry/index/ \
    <<EOF
    apt update
    apt install -y --no-install-recommends clang lld pkg-config libssl-dev
    xx-apt install -y --no-install-recommends xx-c-essentials libssl-dev
    xx-cargo install cargo-binstall cargo-chef cargo-watch sccache
EOF

FROM tonistiigi/xx:master AS xx-final

FROM rust:1-slim AS final
COPY --from=xx-final / /
RUN --mount=type=cache,target=/var/cache/apt/,sharing=locked \
    --mount=type=cache,target=/var/lib/apt/,sharing=locked \
    <<EOF
    apt update
    apt install -y --no-install-recommends clang lld pkg-config patch libssl-dev ca-certificates curl
    rustup component add rustfmt clippy
    rustup install nightly-2023-10-30
    rustup component add --toolchain nightly-2023-10-30 rustfmt clippy
EOF
COPY --from=build /xx-cargo/bin/* /usr/local/cargo/bin/
COPY ./ /
