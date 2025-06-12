# syntax=docker/dockerfile:1

FROM mcr.microsoft.com/vscode/devcontainers/javascript-node:22

ARG ARCH=x86_64
ARG ARCH_SHORT=x64

# Install some generally useful tools
RUN apt-get update \
    && apt-get -y install --no-install-recommends \
    curl git sqlite3 entr source-highlight

# Install Neovim 0.11+
RUN curl -L https://github.com/neovim/neovim/releases/download/v0.11.2/nvim-linux-x86_64.tar.gz \
  | tar -C /usr --strip-components 1 -xzf -

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

# Initial install of packages in Neovim
RUN cd "$HOME" \
  && nvim --headless +'Lazy! sync' +qa

# Ready!
WORKDIR /home/node
CMD ["bash"]
