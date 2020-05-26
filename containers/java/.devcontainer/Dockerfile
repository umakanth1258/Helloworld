#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

ARG VARIANT=11
FROM openjdk:${VARIANT}-jdk-buster

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Options for common package install script
ARG INSTALL_ZSH="true"
ARG UPGRADE_PACKAGES="true"
ARG COMMON_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/common-debian.sh"
ARG COMMON_SCRIPT_SHA="dev-mode"

# Optional Maven settings - leave INSTALL_MAVEN blank to skip
ARG INSTALL_MAVEN="true"
ARG MAVEN_VERSION=3.6.3
ARG MAVEN_SHA=c35a1803a6e70a126e80b2b3ae33eed961f83ed74d18fcd16909b2d44d7dada3203f1ffe726c17ef8dcca2dcaa9fca676987befeadc9b9f759967a8cb77181c0
ENV MAVEN_HOME=${INSTALL_MAVEN:+"/usr/share/maven"}
COPY maven-settings.xml /tmp/

# Optional Gradle settings - leave INSTALL_GRADLE blank to skip
ARG INSTALL_GRADLE=
ARG GRADLE_DOWNLOAD_SHA256=7bdbad1e4f54f13c8a78abc00c26d44dd8709d4aedb704d913fb1bb78ac025dc
ENV GRADLE_HOME=${INSTALL_GRADLE:+"/opt/gradle"}
ENV GRADLE_VERSION=${INSTALL_GRADLE:+"5.4.1"}

# Configure apt
RUN apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    #
    # Verify git, common tools / libs installed, add/modify non-root user, optionally install zsh
    && apt-get -y install --no-install-recommends curl ca-certificates 2>&1 \
    && curl -sSL  ${COMMON_SCRIPT_SOURCE} -o /tmp/common-setup.sh \
    && ([ "${COMMON_SCRIPT_SHA}" = "dev-mode" ] || (echo "${COMMON_SCRIPT_SHA} */tmp/common-setup.sh" | sha256sum -c -)) \
    && /bin/bash /tmp/common-setup.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" \
    && rm /tmp/common-setup.sh \
    #
    # Allow for a consistant java home location for settings since each Java image could be different
    && if [ ! -d "/docker-java-home" ]; then \
        ln -s "${JAVA_HOME}" /docker-java-home; \
    fi \
    #
    # Optionally install Maven
    && if [ "${INSTALL_MAVEN}" = "true" ]; then \
        mkdir -p /usr/share/maven ${MAVEN_HOME}/ref \
        && mv /tmp/maven-settings.xml ${MAVEN_HOME}/ref \
        && curl -fsSL -o /tmp/apache-maven.tar.gz https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
        && echo "${MAVEN_SHA} /tmp/apache-maven.tar.gz" | sha512sum -c - \
        && tar -xzf /tmp/apache-maven.tar.gz -C ${MAVEN_HOME} --strip-components=1 \
        && rm -f /tmp/apache-maven.tar.gz \
        && ln -s "${MAVEN_HOME}/bin/mvn" /usr/bin/mvn; \
    else \
        rm /tmp/maven-settings.xml; \
    fi \
    #
    # Optionally install Gradle
    && if [ "${INSTALL_GRADLE}" = "true" ]; then \
        curl -sSL --output gradle.zip "https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip" \
        && echo "${GRADLE_DOWNLOAD_SHA256} *gradle.zip" | sha256sum --check - \
        && unzip gradle.zip \
        && rm -f gradle.zip \
        && mv "gradle-${GRADLE_VERSION}" "${GRADLE_HOME}/" \
        && ln -s "${GRADLE_HOME}/bin/gradle" /usr/bin/gradle; \
    fi \
    #
    # Cleanup 
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*