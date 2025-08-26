FROM nvidia/cuda:12.9.1-cudnn-devel-ubuntu24.04

ARG USERNAME
ARG USER_UID=1001
ARG USER_GID=${USER_UID}

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV SHELL=/bin/bash


# Initialize apt 
RUN apt-get update --yes 
RUN apt-get dist-upgrade --yes

# Setup all the locale and tz stuff
RUN DEBIAN_FRONTEND=noninteractive apt-get install --yes locales language-pack-en tzdata
RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen \
    && locale-gen
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8
ARG TZ="America/Los_Angeles"
ENV TZ=${TZ}
RUN echo ${TZ} > /etc/timezone \
    && ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
    && dpkg-reconfigure --frontend noninteractive tzdata


# Setup user
RUN groupadd --gid ${USER_GID} ${USERNAME} 
RUN useradd --uid ${USER_UID} \
    --gid ${USER_GID} \
    --shell /bin/bash \
    --create-home ${USERNAME} 
    # to add sudo uncomment the following lines and slash end the prececding line
    # && apt-get update --yes \
    # && apt-get install --yes sudo \
    # && echo ${USERNAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USERNAME} \
    # && chmod 0440 /etc/sudoers.d/${USERNAME}


# Permanent installs 
RUN apt-get install --yes \
    build-essential \
    llvm-dev \
    lldb \
    lld \
    clang \
    python3-dev \
    graphviz \
    libgraphviz-dev \
    openssh-client \
    git \
    ca-certificates 
    # should ca-certificates be in the temp installs section?
    # other stuff



# Temp installs
RUN apt-get install --yes \
    curl 

# Setup nushell because, why not?
RUN curl -fsSL https://apt.fury.io/nushell/gpg.key | gpg --dearmor -o /etc/apt/trusted.gpg.d/fury-nushell.gpg
RUN echo "deb https://apt.fury.io/nushell/ /" | tee /etc/apt/sources.list.d/fury.list
RUN apt-get --yes update
RUN apt-get --yes install nushell

RUN curl -sS https://starship.rs/install.sh | sh -s -- -y

USER ${USERNAME}
WORKDIR /home/${USERNAME}


# **********************************************
# User stuff that does not require root
# **********************************************

# install rust
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y

# load rust and use it
ENV PATH="/home/${USERNAME}/.cargo/bin:${PATH}"
RUN rustup component add rustfmt rust-analyzer rust-src clippy

# install uv
RUN curl -LsSf https://astral.sh/uv/install.sh | sh

# load starship for user because beauty
RUN echo 'eval "$(starship init bash)"' >> ${HOME}/.bashrc
RUN starship preset pastel-powerline -o ${HOME}/.config/starship.toml


# obviously necessary dependency
# RUN cargo install lolcat

# **********************************************
# Cleanup
# **********************************************



USER root
# Remove temporary packages
RUN apt-get remove --yes \
    curl \
    && apt-get autoremove --yes \
    && apt-get clean 

USER ${USERNAME}
WORKDIR /home/${USERNAME}


CMD ["sleep","infinity"]