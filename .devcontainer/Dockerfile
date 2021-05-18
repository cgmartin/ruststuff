# Note: You can use any Debian/Ubuntu based image you want.
FROM rust:1.51-buster

# [Option] Install zsh
ARG INSTALL_ZSH="false"
# [Option] Upgrade OS packages to their latest versions
ARG UPGRADE_PACKAGES="false"
# [Option] Enable non-root Docker access in container
ARG ENABLE_NONROOT_DOCKER="true"

# Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
ARG SOURCE_SOCKET=/var/run/docker-host.sock
ARG TARGET_SOCKET=/var/run/docker.sock
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID
COPY library-scripts/*.sh /tmp/library-scripts/
RUN apt-get update \
    && /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" \
    # Use Docker script from script library to set things up
    && /bin/bash /tmp/library-scripts/docker-debian.sh "${ENABLE_NONROOT_DOCKER}" "${SOURCE_SOCKET}" "${TARGET_SOCKET}" "${USERNAME}" \
    # Set up rust for non-root user
    && /bin/bash /tmp/library-scripts/rust-debian.sh "${CARGO_HOME}" "${RUSTUP_HOME}" "${USERNAME}" "true" "true" \
    # Clean up
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts/

# Setting the ENTRYPOINT to docker-init.sh will configure non-root access to
# the Docker socket if "overrideCommand": false is set in devcontainer.json.
# The script will also execute CMD if you need to alter startup behaviors.
ENTRYPOINT [ "/usr/local/share/docker-init.sh" ]
CMD [ "sleep", "infinity" ]

# Set up persistent bash history
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
    && echo $SNIPPET >> "/root/.bashrc" \
    # [Optional] If you have a non-root user
    && mkdir /commandhistory \
    && touch /commandhistory/.bash_history \
    && chown -R $USERNAME /commandhistory \
    && echo $SNIPPET >> "/home/$USERNAME/.bashrc"

# Install rust extras
RUN su ${USERNAME} -c "cargo install wasm-pack cargo-generate"

# Install node/npm extras
RUN curl -fsSL https://deb.nodesource.com/setup_16.x | bash \
    && apt-get update \
    && apt-get install -y nodejs \
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# # Install python extras
# RUN apt-get update \
#     && apt-get -y install --no-install-recommends pipenv 2>&1 \
#     && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# # Install AWS CLI
# RUN curl -sL https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip -o awscliv2.zip \
#     && unzip awscliv2.zip \
#     && aws/install \
#     && rm -rf awscliv2.zip aws

# # Install AWS SAM CLI
# # RUN curl -sL https://github.com/aws/aws-sam-cli/releases/download/sam-cli-nightly/aws-sam-cli-linux-x86_64.zip -o aws-sam-cli-linux-x86_64.zip \
# #   && ln -s /usr/local/bin/sam-nightly /usr/local/bin/sam
# RUN curl -sL https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip -o aws-sam-cli-linux-x86_64.zip \
#     && unzip aws-sam-cli-linux-x86_64.zip -d sam-installation \
#     && sam-installation/install \
#     && rm -rf aws-sam-cli-linux-x86_64.zip sam-installation
