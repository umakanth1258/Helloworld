
#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

FROM mcr.microsoft.com/vscode/devcontainers/base:debian-10

# Docker Compose version
ARG COMPOSE_VERSION=1.24.0

# The image above includes a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Update user if needed, Install Docker CE CLI and Docker Compose
RUN if [ "$USER_GID" != "1000" ] || [ "$USER_UID" != "1000" ]; then \
        groupmod --gid $USER_GID $USERNAME \
        && usermod --uid $USER_UID --gid $USER_GID $USERNAME \
        && chmod -R $USER_UID:$USER_GID /home/$USERNAME; \
    fi \
    #
    # Install Docker CE CLI
    && apt-get update \
    && apt-get -y install --no-install-recommends apt-transport-https ca-certificates curl gnupg gnupg-agent software-properties-common lsb-release \
    && curl -fsSL https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/gpg | (OUT=$(apt-key add - 2>&1) || echo $OUT) \
    && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]') $(lsb_release -cs) stable" \
    && apt-get update \
    && apt-get install -y docker-ce-cli \
    #
    # Add script to enable non-root user to interact with Docker socket. On Linux, the socket will 
    # be associated with a Docker group that will be unknown to the container. On Windows/macOS, it 
    # will be in the root group instead. Modify as appropriate to handle each case.
    && echo "#!/bin/bash\n\
        SOCKET_GROUP=\$(stat -c '%G' /var/run/docker.sock)\n\
        if [ \"\$SOCKET_GROUP\" = \"UNKNOWN\" ]; then\n\
            sudo groupadd -g \$(stat -c '%g' /var/run/docker.sock) docker\n\
            sudo usermod -aG docker ${USERNAME}\n\
        elif [ \"\$SOCKET_GROUP\" = \"root\" ]; then\n\
            sudo groupadd docker\n\
            sudo chgrp docker /var/run/docker.sock\n\
            sudo usermod -aG docker ${USERNAME};\n\
        fi" > /usr/local/bin/check-docker-group \
    && chmod +x /usr/local/bin/check-docker-group \
    #
    # Install Docker Compose
    && curl -sSL "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose \
    && chmod +x /usr/local/bin/docker-compose \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Set "overrideCommand": false in devcontainer.json to fire this command on
# start and ensure the non-root user can access the Docker socket
CMD [ "/bin/bash", "-c", "/usr/local/bin/check-docker-group && sleep infinity" ]

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog