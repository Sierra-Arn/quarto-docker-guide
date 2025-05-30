# =====================================================
# Stage 1: System Dependencies
# =====================================================

# Use Ubuntu 24.04 as foundation for explicit dependency control
# - Starts clean to avoid hidden pre-installed packages
# - Forces transparent declaration of all components
FROM ubuntu:24.04

# NOTE: Package Management Strategy
# Micromamba serves as a lightweight conda alternative for cross-language package management:
# - Provides pre-compiled packages across multiple ecosystems (Python, R, C++, etc.)
# - Eliminates compilation time compared to pip/CRAN source installations
# - Unified dependency resolution across different programming languages
# - Smaller footprint than full Anaconda/Miniconda distributions

# Set UTF-8 locale to prevent common container issues
# - Eliminates "unsupported locale" errors in logs
# - Ensures proper stdin/stdout encoding
# - Guarantees correct interpretation of special characters
# - Uses C locale with UTF-8 rather than region-specific settings
ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

# Install base system utilities needed for Micromamba
# - ca-certificates: SSL certificate verification
# - curl: Download tool for micromamba
# - tar & bzip2: Archive handling for micromamba binary
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    tar \
    bzip2 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install micromamba (conda alternative) system-wide
# - Binary is placed in /usr/local/bin for global access
# - Version pinned for reproducibility
ARG MICROMAMBA_VERSION=1.5.8

RUN curl -Ls "https://micro.mamba.pm/api/micromamba/linux-64/${MICROMAMBA_VERSION}" \
    | tar -xjv bin/micromamba \
    && mv bin/micromamba /usr/local/bin/ \
    && chmod +x /usr/local/bin/micromamba

# =====================================================
# Stage 2: User Configuration
# =====================================================

# Configure runtime parameters
# - Non-root user for security best practices
# - Dedicated conda prefix for environment isolation
ARG DEV_USER=user
ENV MAMBA_ROOT_PREFIX=/opt/conda

# Create application user with ownership of key directories
# - Home directory required for shell initialization files
# - Explicit permissions prevent privilege escalation risks
RUN useradd --create-home --shell /bin/bash ${DEV_USER} \
    && mkdir -p ${MAMBA_ROOT_PREFIX} \
    && chown -R ${DEV_USER}:${DEV_USER} ${MAMBA_ROOT_PREFIX} \
    && chown -R ${DEV_USER}:${DEV_USER} /home/${DEV_USER}

# Switch to unprivileged user context
USER ${DEV_USER}
WORKDIR /home/${DEV_USER}

# =====================================================
# Stage 3: Python Environment Setup
# =====================================================

# Initialize micromamba shell integration
# - Shell hooks enable 'micromamba activate' in bash sessions
# - Automatic environment activation via .bashrc
ENV ENV_NAME=dev

RUN micromamba shell init --shell bash --root-prefix="${MAMBA_ROOT_PREFIX}" && \
    echo 'eval "$(micromamba shell hook --shell bash)"' >> ~/.bashrc && \
    echo "micromamba activate ${ENV_NAME}" >> ~/.bashrc

# Create base Python environment
# - conda-forge channel for community-maintained packages
# - python: Core Python interpreter
# - numpy: Fundamental package for scientific computing (serves as functionality test)
# - numpy installation validates package management and dependency resolution
RUN micromamba create -y -n ${ENV_NAME} -c conda-forge \
    python \
    numpy \
    && micromamba clean --all -y

# Update container PATH to include environment binaries
ENV PATH="${MAMBA_ROOT_PREFIX}/envs/${ENV_NAME}/bin:${PATH}"

# =====================================================
# Stage 4: Verification and Runtime Configuration
# =====================================================

# Final CMD combines version checks with persistent shell
# - First bash: Initializes new shell process
# - Flag -i: Forces interactive mode to load ~/.bashrc
# - Flag -c: Executes string-based commands then continues
# - && chain: Runs commands sequentially (version checks)
# - Final bash: Maintains active shell after validations
CMD ["bash", "-i", "-c", "micromamba --version && python --version && bash"]