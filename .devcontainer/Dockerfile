# =====================================================
# Stage 1: System Dependencies
# =====================================================
FROM ubuntu:24.04

ARG DEV_USER=user \
    DEV_USER_ID=1000 \
    DEV_USER_GID=1000 \ 
    MICROMAMBA_VERSION=1.5.8 \
    QUARTO_VERSION=1.8.11
ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \ 
    MAMBA_ROOT_PREFIX=/opt/conda \
    ENV_NAME=dev

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    tar \
    bzip2 \
    pandoc \
    libfontconfig1 \
    perl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN curl -Ls "https://micro.mamba.pm/api/micromamba/linux-64/${MICROMAMBA_VERSION}" \
    | tar -xjv bin/micromamba \
    && mv bin/micromamba /usr/local/bin/ \
    && chmod +x /usr/local/bin/micromamba

RUN curl -sSL "https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.deb" -o quarto.deb \
    && dpkg -i quarto.deb \
    && rm quarto.deb

# =====================================================
# Stage 2: User Configuration
# =====================================================

# Create user and group with flexible ID handling for host filesystem compatibility
# This complex logic handles scenarios where UIDs/GIDs might already exist in the container:
# 
# Group Management:
# - Check if target GID already exists using getent group lookup
# - If GID exists but with different name: rename existing group to target name
# - If GID doesn't exist: create new group with specified GID
# - Ensures consistent group ownership regardless of base image state
#
# User Management:
# - Check if target UID already exists using getent passwd lookup  
# - If UID exists but with different name: rename existing user to target name
# - Update home directory path and move existing files if needed
# - Assign user to the correct group (created/renamed above)
# - If UID doesn't exist: create new user with specified UID/GID
# - Ensures proper user/group mapping for volume mounts and file permissions
RUN if getent group ${DEV_USER_GID} >/dev/null; then \
        existing_group=$(getent group ${DEV_USER_GID} | cut -d: -f1); \
        [ "$existing_group" != "${DEV_USER}" ] && groupmod -n ${DEV_USER} $existing_group; \
    else \
        groupadd --gid ${DEV_USER_GID} ${DEV_USER}; \
    fi && \
    if getent passwd ${DEV_USER_ID} >/dev/null; then \
        existing_user=$(getent passwd ${DEV_USER_ID} | cut -d: -f1); \
        [ "$existing_user" != "${DEV_USER}" ] && usermod -l ${DEV_USER} $existing_user; \
        usermod -d /home/${DEV_USER} -m ${DEV_USER} 2>/dev/null || true; \
        usermod -g ${DEV_USER} ${DEV_USER}; \
    else \
        useradd --uid ${DEV_USER_ID} --gid ${DEV_USER_GID} -m -s /bin/bash ${DEV_USER}; \
    fi

RUN mkdir -p ${MAMBA_ROOT_PREFIX} \
    && chown -R ${DEV_USER}:${DEV_USER} ${MAMBA_ROOT_PREFIX} \
    && chown -R ${DEV_USER}:${DEV_USER} /home/${DEV_USER}

USER ${DEV_USER}
WORKDIR /home/${DEV_USER}

# =====================================================
# Stage 3: R/Python Environment Setup
# =====================================================
RUN micromamba shell init --shell bash --root-prefix="${MAMBA_ROOT_PREFIX}" && \
    echo 'eval "$(micromamba shell hook --shell bash)"' >> ~/.bashrc && \
    echo "micromamba activate ${ENV_NAME}" >> ~/.bashrc

RUN micromamba create -y -n ${ENV_NAME} -c conda-forge \
    python \
    jupyter \
    r-base \
    r-knitr \
    r-rmarkdown \
    r-languageserver \
    r-reticulate \
    r-tinytex \
    r-ggplot2 \
    && micromamba clean --all -y

ENV PATH="${MAMBA_ROOT_PREFIX}/envs/${ENV_NAME}/bin:${PATH}" \ 
    R_HOME="${MAMBA_ROOT_PREFIX}/envs/${ENV_NAME}/lib/R" \ 
    R_LIBS_USER="${MAMBA_ROOT_PREFIX}/envs/${ENV_NAME}/lib/R/library"

RUN echo 'options(repos = c(CRAN = "https://cloud.r-project.org"))' >> ~/.Rprofile && \
    echo '.libPaths(c(Sys.getenv("R_LIBS_USER"), .libPaths()))' >> ~/.Rprofile

RUN bash -c "source ~/.bashrc micromamba activate ${ENV_NAME} && \ 
    quarto install tool tinytex --no-prompt && \
    echo 'export PATH=\"\$HOME/.TinyTeX/bin/x86_64-linux:\$PATH\"' >> ~/.bashrc"

ENV PATH="/home/${DEV_USER}/.TinyTeX/bin/x86_64-linux:${PATH}"

RUN bash -c "source ~/.bashrc micromamba activate ${ENV_NAME} && \
    tlmgr update --self && \
    tlmgr install \
    babel-russian \
    fontspec"

# =====================================================
# Stage 4: Julia Environment Setup
# =====================================================
RUN curl -fsSL https://install.julialang.org | sh -s -- -y --path ~/.juliaup

ENV PATH="/home/${DEV_USER}/.juliaup/bin:${PATH}"

RUN juliaup add 1.11.5 && \
    juliaup default 1.11.5

RUN echo 'export JULIA_PROJECT="@."' >> ~/.bashrc
RUN julia -e 'using Pkg; Pkg.add(["Example"])' && \
    julia -e 'using Pkg; Pkg.precompile()'

RUN micromamba run -n ${ENV_NAME} R -e "\
    Sys.setenv(JULIA_BINDIR = '/home/${DEV_USER}/.juliaup/bin'); \
    install.packages('JuliaCall')"

# =====================================================
# Stage 5: Verification and Runtime Configuration
# =====================================================
CMD ["bash", "-i", "-c", "\
    micromamba --version && \
    python --version && \
    R --version && \
    julia --version && \
    quarto --version && \
    tlmgr --version && \
    quarto check && \
    bash"]