# syntax=docker/dockerfile:1

# This is a bit hacky / messy but it works for now ...

# --------------------------------------------------------------------
# Base

FROM debian:12 AS builder-base
RUN apt-get update
RUN apt-get -y install ca-certificates ninja-build gettext cmake unzip curl build-essential git file libevent-dev ncurses-dev bison pkg-config vim
RUN mkdir /pdebuilder

# --------------------------------------------------------------------
# Neovim

FROM builder-base AS builder-neovim
WORKDIR /pdebuilder
RUN git clone https://github.com/neovim/neovim \
  && cd neovim \
  && make CMAKE_BUILD_TYPE=Release \
  && cd build \
  && cpack -G DEB \
  && echo NEOVIM DONE

# --------------------------------------------------------------------
# Main

FROM mcr.microsoft.com/vscode/devcontainers/javascript-node:22

ARG ARCH=x86_64
ARG ARCH_SHORT=x64

COPY --from=builder-neovim /pdebuilder/neovim/build/nvim-linux-$ARCH.deb /tmp

# Install some generally useful tools
RUN apt-get update \
    && apt-get -y install --no-install-recommends \
    curl git sqlite3 entr source-highlight

# Install Neovim
RUN dpkg -i /tmp/nvim-linux-$ARCH.deb

# Install CAP SDK, plus CDS and typescript language server packages
USER node
ARG HOME="/home/node"
ARG NPMGLOBAL="$HOME/.npm-packages" && mkdir -p "$NPMGLOBAL"
RUN npm config set prefix "$NPMGLOBAL"
RUN npm install -g \
  @sap/cds-dk \
  @sap/cds-lsp \
  typescript-language-server

# Some helpful setup
RUN cat <<EOBASHRC >> "$HOME/.bashrc"
export PATH=$PATH:$HOME/.npm-packages/bin/
export EDITOR=vi
set -o vi
bind -x '"\C-l": clear'
EOBASHRC

# Use the latest version of the "clean" config cluster from
# https://qmacro.org/blog/posts/2025/06/10/a-modern-and-clean-neovim-setup-for-cap-node.js-configuration-and-diagnostics/
RUN mkdir -p "$HOME/.config" \
  && git clone https://github.com/qmacro/dotfiles /tmp/dotfiles \
  && cp -a /tmp/dotfiles/config/nvim-configs/clean "$HOME/.config/nvim"

RUN echo 'export PS1=${PS1/\$ /\\\\n$ }' >> /home/node/.bashrc

# Ready!
WORKDIR /home/node
CMD ["bash"]
